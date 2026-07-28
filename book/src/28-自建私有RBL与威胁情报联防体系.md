# 第二十八章：自建私有 RBL/URIBL 与威胁情报联防体系

> 从公网 RBL 的被动消费者，到自建私有黑灰产情报网的主动生产者——本章教你用 rbldnsd 构建企业级高并发 Blackhole 后端，实现跨集群秒级威胁情报同步与全网联防。

## 28.1 为什么企业需要自建 RBL 后端

在企业级邮件安全架构中，仅依赖公网 DNSBL（Spamhaus、SpamCop、Barracuda、Invaluement）是**不够的**。

### 28.1.1 公网 RBL 的三大天花板

| 限制 | 具体表现 | 影响 |
|------|---------|------|
| **QPS 限制** | Spamhaus 免费查询有严格的 QPS 上限，超出后返回 `127.255.255.254`（拒绝服务） | MTA 高峰期大量邮件因 RBL 超时被 `defer`，造成队列堆积 |
| **商业许可** | 企业级大规模查询需购买 Datafeed Service（Spamhaus DQS），费用不菲 | 中小企业与自建邮局无法负担 |
| **无法注入私有威胁情报** | 你发现了一个正在暴破 IMAP 的 IP，无法实时写入 Spamhaus 让全网 MTA 封禁 | 威胁只能在本地 Fail2ban 层面封禁，无法跨集群共享 |

### 28.1.2 自建 RBL 的架构价值

1. **突破公网限制**：通过 Rsync/BGP 下载完整 Feed 到本地，用轻量级 `rbldnsd` 提供内部高并发 DNS 查询——QPS 从"公网 3K/s"提升到"单核 50K/s+"
2. **构建企业威胁情报联防联控**：HoneyPot 捕获的暴破 IP、Rspamd 标记的钓鱼域名、SIEM/SOAR 判定的恶意 IP → REST API 写入私有 Blackhole → **1 秒内全网 Postfix/Exchange 同步阻断**
3. **域名/URL 级阻断（URIBL/RHSBL）**：不仅能封 IP（IP-based DNSBL），还能将钓鱼邮件中的恶意域名（Domain-based）实时注入本地 URIBL，供 Rspamd 的 `uribl` 模块实时查询

## 28.2 DNSBL 协议规范与工作原理 (RFC 5782)

### 28.2.1 逆向 DNS 查询机制

DNSBL 不是查正向 A 记录，而是通过**反转 IP 地址**构造特殊的 DNS 查询名：

```
发件人 IP: 203.0.113.45
↓ 反转
查询名: 45.113.0.203.zen.spamhaus.org
↓ dig A
返回值: 127.0.0.4  (表示命中 CBL 黑名单)
```

### 28.2.2 Return Code 语义 (127.0.0.x)

| 返回值 | 含义 | 示例 |
|--------|------|------|
| `127.0.0.2` | 确认的垃圾邮件源（SBL） | Spamhaus SBL |
| `127.0.0.3` | 确认的雪鞋/邮件炸弹（CSS） | Spamhaus CSS |
| `127.0.0.4` | 确认的僵尸网络/CBL | Spamhaus CBL/XBL |
| `127.0.0.10` | 拨号/动态 IP（PBL） | Spamhaus PBL |
| `127.255.255.254` | **查询被拒绝（超 QPS 限制）** | ⚠️ 不要封禁这个 IP！ |
| `127.255.255.255` | **查询被拒绝（其他原因）** | 同样不封禁 |

### 28.2.3 TXT 记录的拦截原因

除了 A 记录判断是否命中，RBL 查询还会附带 TXT 记录说明命中的理由：

```
$ dig +short TXT 2.0.0.127.zen.spamhaus.org
"https://www.spamhaus.org/query/ip/127.0.0.2"
```

Postfix 可以在 `reject_rbl_client` 中配置 `=message` 参数，直接显示 TXT 记录内容到 SMTP 退信。

## 28.3 自建 RBL 服务端选型：rbldnsd

### 28.3.1 为什么选 rbldnsd

`rbldnsd` 是 Michael Tokarev 开发的**专用 DNSBL 轻量级守护进程**（C 语言实现）。它**不是**通用 DNS 服务器（无需 BIND/Unbound），而是完全为 RBL 场景优化的内存引擎。

| 特性 | rbldnsd | BIND/Unbound | CoreDNS + Redis |
|------|---------|-------------|----------------|
| 内存占用 | 加载 500 万 IP ~40MB | 同规模 Zone 需 300MB+ | 取决于 Redis 内存 |
| QPS（单核） | 50,000+ | 10,000-15,000 | 20,000-40,000 |
| 数据格式 | CIDR/ip4set/combined/A+TXT | 标准 BIND Zone 文件 | 自定义 |
| 更新方式 | SIGHUP 重新加载 Zone 文件 | `rndc reload` | REST API 实时写入 |
| 启动速度 | < 1 秒 | 30-60 秒 | 即时 |

### 28.3.2 rbldnsd 安装与 Zone 文件格式

```bash
# Debian/Ubuntu 安装
apt-get install rbldnsd

# Zone 文件格式 (rbldnsd.dat)
# :<type>:<name>:<描述>
:ip4set:blacklist:企业内部黑名单
1.2.3.4        # 单个 IP
5.6.7.0/24     # 整个 C 段
192.168.1.100  # 内部蜜罐捕获的暴破 IP

:ip4trie:spamhaus_mirror:Spamhaus 本地镜像
10.0.0.0/8    Not listed
203.0.113.45  SBL/CBL - 已知垃圾邮件源

:dnset:uribl:钓鱼域名黑名单
evil-phishing.com    钓鱼域名
malware-c2.net       C2 通信域名
```

### 28.3.3 部署 rbldnsd 并配置 Postfix 接入

```bash
# 启动 rbldnsd（监听 127.0.0.1:5300）
rbldnsd -b 127.0.0.1/5300 \
  -r /var/lib/rbldnsd \
  -w /var/lib/rbldnsd/private \
  blacklist:ip4set:/var/lib/rbldnsd/blacklist.dat

# 本地测试
dig @127.0.0.1 -p 5300 100.1.2.3.blacklist.example.com A

# Postfix main.cf
smtpd_recipient_restrictions =
    ...
    reject_rbl_client blacklist.example.com=127.0.0.2
    reject_rhsbl_sender  uribl.example.com
    ...
```

### 28.3.4 配置 Rspamd 加载私有 RBL

```lua
-- /etc/rspamd/local.d/rbl.conf
rbls {
  PRIVATE_BLACKLIST {
    rbl = "blacklist.example.com";
    returncodes = {
      PRIVATE_BLACKLIST = "127.0.0.2";
    };
    symbol = "RBL_PRIVATE";
  }
  PRIVATE_URIBL {
    rbl = "uribl.example.com";
    returncodes = {
      PRIVATE_URIBL = "127.0.0.2:8";
    };
    symbol = "URIBL_PRIVATE";
  }
}
```

## 28.4 公网商业 Feed 本地镜像化

### 28.4.1 Spamhaus Datafeed (DQS)

- **Rsync 同步**：购买 DQS 订阅后通过 `rsync` 定时拉取增量数据
- 同步脚本（crontab 每 15 分钟）：

```bash
#!/bin/bash
rsync -avz --delete rsync://rsync.spamhaus.net/dqs/ /var/lib/rbldnsd/dqs/
# 通知 rbldnsd 重新加载
kill -HUP $(cat /var/run/rbldnsd.pid)
```

### 28.4.2 其他可镜像的 Feed

| Feed | 获取方式 | 说明 |
|------|---------|------|
| Spamhaus DQS | Rsync（付费） | SBL/CBL/XBL/PBL 完整数据 |
| Invaluement | Rsync（付费） | 高质量小众反垃圾 Feed |
| Abuse.ch | Rsync/Cron HTTP | 开源威胁情报，免费 |
| FireHOL IP Lists | HTTP Download | 综合黑名单聚合，免费 |
| Emerging Threats | HTTP Download | Proofpoint 维护，含 C2/Botnet IOC |

## 28.5 自动化威胁情报注入：从蜜罐到全网秒级封禁

> 这是将本书从"RBL 消费者"升维到"威胁情报生产者"的关键一环。

### 28.5.1 SOC 发现 → RBL 注入的自动化流水线

```text
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│ HoneyPot │ →  │  SIEM    │ →  │ SOAR     │ →  │ rbldnsd  │
│ CrowdSec │    │ Wazuh/ELK│    │ Playbook │    │ 私有 RBL  │
└──────────┘    └──────────┘    └──────────┘    └─────┬────┘
     │                                                │
     捕获暴破 IP                                   毫秒级全网生效
                                                    │
                              ┌──────────┐           │
                              │ Postfix  │ ←─────────┘
                              │ Exchange │   smtpd_recipient_
                              │ 全网网关  │   restrictions
                              └──────────┘
```

### 28.5.2 基于 Redis + Custom DNS Server 的现代方案

如果企业需要**毫秒级实时注入**而非依赖 SIGHUP 重载：

- **CoreDNS** + **`redis` plugin**：后端直接读取 Redis 中的 SET 数据结构
- REST API 注入：

```bash
# SOAR Playbook 触发后：
curl -X POST https://rbl-api.internal/api/v1/block \
  -H "Authorization: Bearer $SOAR_TOKEN" \
  -d '{"ip": "203.0.113.99", "reason": "SSH brute-force detected by CrowdSec", "ttl": 86400}'

# → Redis SET rbl:v4:203.0.113.99 "Blocked: SSH brute-force"
# → CoreDNS redis plugin 即时响应 DNS 查询
# → 全网 Postfix 在下一个 SMTP 连接中即可拦截
```

### 28.5.3 防御效果指标

| 指标 | 公网 RBL 独用 | 加入自建 RBL 后 |
|------|:----------:|:-----------:|
| 新型暴破 IP 阻断延迟 | 6-24 小时（依赖公网 Feed 更新周期） | **< 3 秒**（HoneyPot 捕获 → API 注入 → DNS 解析） |
| 针对性钓鱼域名覆盖率 | 约 70%（只覆盖大范围公开威胁） | **> 95%**（企业专属域名 + 手动提交 + Feed 聚合） |
| 跨集群阻断一致性 | 各节点独立查询公网 RBL，结果不一致 | **完全一致**（所有节点查询同一个私有 RBL） |

## 28.6 对比：历史方案与现代演进

### 28.6.1 djbdns rbldns（DJB 经典方案）

- Daniel J. Bernstein 设计的 `rbldns` 工具，是 DNSBL 自建的"祖师爷"
- 代码安全性极高（DJB 的极端安全哲学），但配置格式古板
- 在现代架构中，`rbldnsd` 作为功能更丰富的后续者已基本取代 `rbldns`

### 28.6.2 方案对比矩阵

| 方案 | 数据存储 | 更新方式 | QPS | 适用场景 |
|------|---------|---------|:---:|---------|
| **rbldnsd**（推荐） | 内存 Zone | SIGHUP 重载 | 50K+ | 本地镜像 Spamhaus Feed |
| **djbdns rbldns** | 内存 Zone | 手动重载 | 20K+ | 极简环境 / 历史研究 |
| **CoreDNS + Redis** | Redis SET | REST API 实时 | 40K+ | SOAR / 动态封禁系统 |
| **PowerDNS + MySQL** | 数据库 | SQL 实时 | 5K+ | 需要 Web 管理面板 |

---

> **章节总结**：从"依赖公网 RBL"到"自建威胁情报生产中心"，是企业邮件安全成熟度从 L2（可重复级）跃迁到 L4（已管理级）的核心标志。`rbldnsd` 是这一跃迁中最务实的技术底座——它足够轻量（C 语言、内存化、无依赖），又足够强大（单核 50K QPS、支持 CIDR/URIBL/TXT）。
