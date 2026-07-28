# 邮件系统安全加固：从 SMTP 协议到国密与高可用架构

> 现代企业邮件与协同基础设施安全加固 —— 从 SMTP 协议到国密、云端 PaaS 与高可用架构。
> 作者：**LeisureLinux** (郭靖大侠)

[![build](https://github.com/LeisureLinux/ebook-email-hardening/actions/workflows/build.yml/badge.svg)](https://github.com/LeisureLinux/ebook-email-hardening/actions/workflows/build.yml)
[![license](https://img.shields.io/badge/license-MIT-blue.svg)](./LICENSE)

---

## 📚 姐妹系列图书

| # | 书名 | 在线阅读 | 仓库 |
|---|------|---------|------|
| 1 | **Nginx 纵深加固：从反向代理到 WAF 实战** | [在线阅读](https://leisurelinux.github.io/ebook-nginx-hardening/) | [GitHub](https://github.com/LeisureLinux/ebook-nginx-hardening) |
| 2 | **SSH 纵深加固：从攻击链到六层防御体系** | [在线阅读](https://leisurelinux.github.io/ebook-ssh-hardening/) | [GitHub](https://github.com/LeisureLinux/ebook-ssh-hardening) |
| 3 | **/proc 攻防演义：从 Linux 进程真相到可观测性实战** | [在线阅读](https://leisurelinux.github.io/ebook-procfs-hardening/) | [GitHub](https://github.com/LeisureLinux/ebook-procfs-hardening) |
| 4 | **邮件系统安全加固（本书）** | [在线阅读](https://leisurelinux.github.io/ebook-email-hardening/) | [GitHub](https://github.com/LeisureLinux/ebook-email-hardening) |

---

## 在线阅读 / 下载

| 格式 | 入口 |
|---|---|
| 🌐 HTML 在线版 | <https://leisurelinux.github.io/ebook-email-hardening/> |
| 📄 PDF | [Releases](../../releases) 或 [直接下载](https://leisurelinux.github.io/ebook-email-hardening/email-hardening.pdf) |
| 📖 ePub | [Releases](../../releases) 或 [直接下载](https://leisurelinux.github.io/ebook-email-hardening/email-hardening.epub) |

---

## 前言

在企业数字化基础设施与安全防御体系中，邮件系统始终占据着特殊而关键的技术地位。它既是企业最古老、最基础的异步通信协议（SMTP/POP3/IMAP）的集大成者，也是现代企业身份认证（IAM）、协同办公（CalDAV/CardDAV）、信息权利管理（DRM）以及合规审计的底层核心。

长期以来，许多 IT 运维与安全人员将邮件系统简化为"搭建一个 MTA/MDA 服务并配置 SPF/DKIM/DMARC 记录"。然而，在复杂的企业网络拓扑与高对抗的安全环境下，这种割裂的认知正面临严重的系统性风险：

1. **威胁模型的维度升级**：高级持续性威胁（APT）组织与商业邮件诈骗（BEC）团伙早已突破传统的垃圾邮件过滤范畴。利用 Autodiscover 机制回退劫持凭据、通过日历邀约（.ics）注入恶意外链绕过网关沙箱、在客户端或 Webmail 层面设置隐蔽的 Sieve/Inbox Rule 进行持久化权限维持、以及利用未隔离的营销邮件 IP 污染主域名声誉，已成为当前最主流的攻击路径。
2. **业务与安全特性的深度缠绕**：现代邮件系统并非孤立的文本收发引擎。如何实现在 Webmail 前端针对敏感数据的动态水印与防复制/防打印（DRM），如何将反病毒网关（Anti-Virus Gateway）以零退信风暴（Backscatter）的方式接驳至 Milter 管道，如何通过 Fail2ban、CrowdSec 与 Postscreen 在 TCP handshake/SASL 阶段防御自动化凭据喷射，如何对历史遗留系统（如 HCL Domino）进行 NRPC 与 ACL 硬化，都是需要从底层协议层进行整体规划的系统工程。
3. **架构解耦与高可用/灾备（HA/DR）的要求**：邮件系统包含无状态服务（MTA/Proxy）与强有状态服务（Mailbox/Storage/DB/Session）。在零数据丢失（RPO=0）与秒级故障转移（RTO→0）的技术约束下，如何实现 Dovecot Cluster 的实时状态同步、共享存储/分布式文件系统的锁机制调优、以及云端 PaaS（AWS SES/阿里云 DirectMail）与营销自动化（EDM）的域名与 IP 声誉物理隔离，决定了架构的健壮性上限。
4. **信创生态与自主可控合规要求**：随着网络安全等级保护 2.0（三级）与商用密码应用安全性评估（密评）的全面推进，邮件系统必须适配鲲鹏、飞腾、海光等国产 CPU 及统信 UOS、银河麒麟操作系统，并实现传输层双证书（TLCP）及内容层国密（SM2/SM3/SM4 S/MIME）算法的深度落地。

本书旨在从**系统架构师、安全分析师与高级 IT 咨询顾问**的视角，剥离不必要的概念包装与比喻，以底层 RFC 规范、报文交互逻辑、组件源码级工作原理以及生产环境部署实战为线索，全面解构现代企业级邮件与协同系统的硬化与架构防护体系。

全书分为七个核心部分与三个附录，逻辑严密，涵盖从协议基础、终端/Webmail 加固、存储/过滤层防护、传输/网关/反病毒集成、云端 PaaS/EDM 隔离，到信创国密、高可用灾备实战的完整知识体系。

---

## 完整目录

### 第一部分：邮件系统体系架构与安全术语图谱

#### 第一章 邮件与协同系统拓扑与协议演进架构

- **1.1** 企业级全栈邮件工作流与角色拆解
  - 1.1.1 发送链条：MUA → MSA (580/465) → MTA (25) 协议手握与报文封装
  - 1.1.2 投递与检索链条：MTA → MDA (LMTP) → 存储引擎 → MRA (POP3/IMAP)
  - 1.1.3 协同与 Web 链条：Webmail / DAV 服务 (CalDAV/CardDAV) 与后端解耦拓扑
- **1.2** 零信任视角下的边界防御与逻辑隔离模型
  - 1.2.1 信道安全、身份鉴权、内容检测与存储解耦架构
  - 1.2.2 DMZ 网关与 Core 核心区邮件系统的微隔离与安全域划定
- **1.3** 现代邮件系统综合威胁图谱
  - 1.3.1 凭据窃取与喷射攻击（Password Spraying / Brute-Force）
  - 1.3.2 身份伪造与商业邮件诈骗（BEC / Lookalike Domain）
  - 1.3.3 供应链注入、日历钓鱼与 Autodiscover 劫持机制

#### 第二章 邮件安全概念与规范术语全景图（Glossary）

- **2.1** 传输、检索与遗留协同协议：SMTP, ESMTP, POP3, IMAP4, LMTP, NRPC (Lotus/Domino)
- **2.2** 身份鉴权与目录服务：SASL (PLAIN, LOGIN, GSSAPI), PAM, OpenLDAP, Active Directory, Kerberos, OIDC/OAuth2
- **2.3** 域名身份校验与防冒用体系：SPF, DKIM, DMARC, ARC, BIMI
- **2.4** 传输层加密与通道防御：STARTTLS, Implicit TLS, MTA-STS, TLS-RPT, DANE (TLSA)
- **2.5** 服务端规则与协同协议：Sieve, ManageSieve, CalDAV, CardDAV, iCalendar, vCard, Autodiscover/Autoconfig
- **2.6** 数字签名、DRM 与归档：OpenPGP, S/MIME v4.0, WKD, TSA, RMS/MIP, WORM 存储

---

### 第二部分：终端接入、协同服务与数据防泄露 (DLP) 加固

#### 第三章 胖客户端（MUA）、自动配置与 DRM 权限控制

- **3.1** 胖客户端攻击面与防御基线
  - 3.1.1 HTML 渲染引擎漏洞、外部追踪像素（Tracking Pixels）与动态远程资源阻断
  - 3.1.2 从 Basic Auth 到 OAuth 2.0 / OIDC 现代鉴权的平滑迁移
  - 3.1.3 Thunderbird 与 Outlook 企业策略下发（GPO）、S/MIME 强制签名与弱套件禁用
- **3.2** 客户端自动配置（Autodiscover / Autoconfig）防劫持
  - 3.2.1 Microsoft Autodiscover XML/JSON 流程硬化与盲目回退劫持防御
  - 3.2.2 Thunderbird `autoconfig.xml` 与 Apple `.mobileconfig` 策略部署与签名校验
- **3.3** 企业级邮件 DRM / IRM 权限控制实现
  - 3.3.1 微软 MIP / RMS (Rights Management Services) 架构：内存解密与 Windows API Hook 拦截
  - 3.3.2 禁用转发（Do Not Forward）、禁止打印、禁止复制与防截屏技术路径
  - 3.3.3 第三方 DLP 客户端插件与透明加密接驳

#### 第四章 Webmail 架构与前端安全攻防及动态水印

- **4.1** Webmail 前端安全威胁与沙箱隔离
  - 4.1.1 HTML 邮件恶意渲染防护：XSS 过滤、CSS 样式隔离、DOM Clobbering 防范
  - 4.1.2 附件安全：SVG 跨站脚本、同源沙箱隔离（Sandbox Domain）与 `Content-Disposition: attachment`
- **4.2** Webmail 动态水印与泄密溯源
  - 4.2.1 HTML5 Canvas / CSS 屏幕明水印（工号/IP/时间戳）渲染与防篡改
  - 4.2.2 文本零宽字符（Zero-Width Space）盲水印注入与抄袭/截图倒查追溯
- **4.3** 网页端 DRM 限制与附件无下载预览沙箱
  - 4.3.1 CSS `@media print` 打印阻断与 JS 右键/选中/复制事件强行拦截
  - 4.3.2 附件在线转码（LibreOffice/OnlyOffice）H5 向量流预览与物理下载剥离
- **4.4** 主流 Webmail 实例加固
  - 4.4.1 Roundcube / SnappyMail：CSRF Token、CSP 响应头与插件沙箱硬化
  - 4.4.2 Zimbra Web Client / OWA：接口防暴破、WAF 挂接与 API 鉴权加固

#### 第五章 日历（Calendar）与通讯录（Contacts）协同服务安全

- **5.1** CalDAV / CardDAV 协议架构与鉴权（SabreDAV, Baïkal, Radicale）
- **5.2** 日历钓鱼（Calendar Invite Phishing）防御机制
  - 5.2.1 外部 `.ics` 会议邀约的网关级沙箱解析与恶意 URL 重写
  - 5.2.2 客户端"自动接受邀约"（Auto-Accept）规则的安全封禁与策略调整
- **5.3** 通讯录与企业全局地址簿（GAL）防拖库
  - 5.3.1 CardDAV / LDAP 地址簿拉取速率限制（Rate-limiting）与异常行为告警
  - 5.3.2 敏感组织架构（高管/财务）在 GAL 中的隐藏与 ACL 访问控制

#### 第六章 端到端加密 (E2EE) 与数字签名体系

- **6.1** OpenPGP/GPG 体系及自动化密钥分发
  - 6.1.1 GnuPG 命令行与库级集成、信任网（Web of Trust）模型
  - 6.1.2 WKD (Web Key Directory) 自动化公钥分发部署，替代传统 HKP
- **6.2** S/MIME 与企业 PKI 体系集成
  - 6.2.1 企业内部 CA 证书颁发、自动密钥轮转与 CRL/OCSP 实时状态校验
  - 6.2.2 CAdES / PAdES 公文附件签名与 TSA (Time Stamping Authority) 可信时间戳引入
- **6.3** 硬件密码设备集成：Smartcard、HSM 与 USB Token 托管签名私钥

---

### 第三部分：收件、过滤与存储层 (POP3/IMAP/Groupware) 加固

#### 第七章 POP3/IMAP 服务端组件架构（以 Dovecot 为例）

- **7.1** Dovecot 模块化架构与权限隔离
  - 7.1.1 `master`, `auth`, `imap-login`, `config` 进程隔离与 UNIX Socket 权限控制
  - 7.1.2 TLS/SSL 链路硬化：禁用 TLS 1.0/1.1、强 Cipher Suites 限制
- **7.2** 协议命令限制与内置防暴破机制
  - 7.2.1 明文登录（Plaintext Auth）强制阻断策略
  - 7.2.2 Dovecot `anvil` 进程内存计数与 `auth_penalty` 延迟响应惩罚机制

#### 第八章 服务端邮件过滤引擎（Sieve）与规则安全

- **8.1** Dovecot Sieve / Pigeonhole 架构与安全部署
  - 8.1.1 Sieve 脚本沙箱隔离、CPU 执行超时与内存资源配额限制
  - 8.1.2 ManageSieve 协议 (端口 4190) TLS 强关联与 SASL 鉴权加固
- **8.2** 异构平台服务端过滤规则安全审计
  - 8.2.1 Procmail / Maildrop 遗留脚本的隐蔽提权风险与 Sieve 安全迁移
  - 8.2.2 Exchange Transport Rules (ETR) 与 Inbox Rules 恶意隐蔽转发后门审计

#### 第九章 身份鉴权深度集成：PAM、LDAP 与数据库

- **9.1** SASL 鉴权管道与后端接驳
- **9.2** 目录服务与数据库加固
  - 9.2.1 OpenLDAP/Active Directory 安全查询、凭据哈希（SSHA512/Argon2）校验
  - 9.2.2 数据库 (MySQL/PgSQL) 鉴权防 SQL 注入与只读账号隔离
- **9.3** 协议级多因子认证（MFA/2FA）
  - 9.3.1 IMAP/POP3 下 TOTP 机制引入与应用专用密码（App-Specific Password）设计

#### 第十章 企业协同群组平台与历史遗留系统加固

- **10.1** 遗留系统生存状态：HCL Domino (Lotus Notes)、Zimbra Collaboration、Exchange On-Premises
- **10.2** HCL Domino (Lotus Notes v12/v14) 专项加固
  - 10.2.1 NRPC 专有协议端口 (TCP 1352) 访问控制与通道加密
  - 10.2.2 ID File 密钥文件保护、Notes Agent 执行安全等级设置
  - 10.2.3 Domino ACL (Access Control List) 精细化权限与 NSF 数据库 Web 越权防御
- **10.3** 混合云与遗留系统的隔离代理：DMZ 代理网关、网络微隔离与平滑迁移

#### 第十一章 邮件存储层安全与合规归档

- **11.1** 磁盘存储加固与静态加密
  - 11.1.1 Maildir 与 mdbox 格式 Linux POSIX ACL 精细化权限控制
  - 11.1.2 基于 Dovecot Plugin 的存储层静态透明加密 (Data-at-Rest Encryption)
- **11.2** 企业级邮件合规归档与 WORM 存储
  - 11.2.1 活性存储（Mailbox）与归档存储（Archiving）物理隔离
  - 11.2.2 基于 Mailpiler 的全量邮件盲摄取、索引与 Legal Hold（法律留存）
  - 11.2.3 WORM (Write Once, Read Many) 存储接驳与日志不可抵赖性保障

---

### 第四部分：传输层 (MTA)、邮件列表与反病毒网关集成

#### 第十二章 主流 MTA 架构与访问控制（以 Postfix 为核心）

- **12.1** Postfix 模块化 Chroot 隔离设计哲学
- **12.2** SMTP 阶段性限制策略与防 Open Relay
  - 12.2.1 `smtpd_client_restrictions`, `smtpd_sender_restrictions`, `smtpd_recipient_restrictions` 精细配置
  - 12.2.2 彻底阻断开放中继（Open Relay）与严格 ACL 编写
- **12.3** Postscreen 前置防御引擎实战
  - 12.3.1 TCP 握手阶段行为分析：Pregreet 违规阻断、裸 IP 检测
  - 12.3.2 结合 DNSBL/RBL 动态黑名单与 Greylisting（灰名单）延迟响应

#### 第十三章 SASL 鉴权与传输链路加密硬化

- **13.1** Postfix 与 Dovecot-SASL / Cyrus-SASL 管道加固
- **13.2** SMTP 通道 TLS 加密硬化
  - 13.2.1 `smtpd_tls_security_level = encrypt` 强制策略与 PFS (Forward Secrecy)
  - 13.2.2 基于 DANE (TLSA) 与 MTA-STS 的出站 SMTP 防降级攻击

#### 第十四章 域身份校验与反冒用自动化建设

- **14.1** OpenSPF 策略部署与 DNS 10 次解析超限防护
- **14.2** OpenDKIM 部署、多域名私钥自动轮转（Key Rotation）
- **14.3** OpenDMARC 日志聚合、解析与 `p=none` 到 `p=reject` 平滑过渡策略
- **14.4** BIMI (Brand Indicators) 接入与 VMC 证书审核

#### 第十五章 邮件列表服务（Mailing List）安全与 ARC 信任链

- **15.1** 邮件列表引擎（GNU Mailman 3 / Sympa）架构
- **15.2** 身份认证破坏修复：Header/Body 修改导致 SPF/DKIM 失效的技术原理
- **15.3** ARC (Authenticated Received Chain - RFC 8617) 完整落地
  - 15.3.1 ARC-Seal, ARC-Message-Signature, ARC-Authentication-Results 签署原理
  - 15.3.2 在 Mailman/Sympa 前端构建 ARC 信任链
- **15.4** 列表安全防护：订阅接口防暴破、反向地址泄漏与 VERP 安全设计

#### 第十六章 Milter 管道与反病毒网关（Anti-Virus Gateway）集成

- **16.1** 反病毒网关（AV Gateway）的 4 种集成模式对比
  - 16.1.1 模式 1：Milter 实时流式挂接（`clamav-milter` / Rspamd AV）——性能最优，直接拒绝
  - 16.1.2 模式 2：Dual-MTA / Amavisd-new 内容过滤器重注入——成熟度高，支持深度解包
  - 16.1.3 模式 3：Inline 独立 SEG 网关模式（Store-and-Forward）——拓扑解耦
  - 16.1.4 模式 4：Exchange Transport Agent / API 拦截与落盘后异步扫描
- **16.2** ClamAV / 商业杀毒引擎（Sophos, Kaspersky）与 Postfix 实战集成
- **16.3** 附件解包、压缩包炸弹（Zip Bomb）与恶意宏（VBA）拦截规则

---

### 第五部分：邮件安全网关 (SEG)、高级威胁与动态防护

#### 第十七章 开源邮件安全网关架构与集成

- **17.1** Proxmox Mail Gateway (PMG) 集群部署与策略定制
- **17.2** Rspamd 高并发异步过滤引擎深度解析
  - 17.2.1 基于 Lua 的规则编写、Redis 后端缓存与神经网络（Neural）概率过滤
  - 17.2.2 Rspamd Ratelimit 模块与 Fuzzy 散列重复邮件阻断
- **17.3** SpamAssassin 规则集定制与贝叶斯（Bayesian）学习模型

#### 第十八章 商业邮件安全网关（SEG）能力拆解与高级威胁防护

- **18.1** 商业 SEG 体系架构分析
  - 18.1.1 Cisco IronPort (AsyncOS)：SenderBase 评估与 CBR Engine
  - 18.1.2 Proofpoint：TAP 威胁保护、URL 重写 (URL Defense) 与 DLP 联动
  - 18.1.3 Fortinet FortiMail / Barracuda 策略映射
- **18.2** 高级威胁防护 (ATP) 机制
  - 18.2.1 商业邮件诈骗（BEC）与相似域名（Lookalike Domain）算法识别
  - 18.2.2 动态沙箱投递与附件行为分析（Detonation Chamber）

#### 第十九章 日志分析、动态防护与 SIEM/SOAR 联动

- **19.1** 基于日志解析的动态入侵防御
  - 19.1.1 Fail2ban 对 Dovecot / Postfix SASL 失败日志解析与 `nftables` 联动封禁
  - 19.1.2 现代协同式 IPS：CrowdSec 部署、规则解析与全球黑名单情报共享
- **19.2** 企业级 SIEM / SOC 架构接驳
  - 19.2.1 邮件全栈 Syslog 格式标准化与 Vector / Syslog-ng 汇聚管道
  - 19.2.2 Wazuh / ELK 邮件安全告警规则编写与 SOAR 自动化防火墙封禁下发

---

### 第六部分：云端 PaaS、事务邮件与 EDM 营销自动化加固

#### 第二十章 云端事务邮件推送 (PaaS) 与 EDM 营销自动化加固

- **20.1** 业务邮件与批量推送的架构解耦与子域隔离
  - 20.1.1 域名声誉隔离：主域（`company.com`）与事务域（`notify.company.com`）、营销域（`marketing.company.com`）隔离
  - 20.1.2 独享 IP（Dedicated IP）预热算法（IP Warming）与递增投递模型
  - 20.1.3 软硬退信（Soft / Hard Bounce）与垃圾邮件反馈环（FBL）处理
- **20.2** 云端事务性邮件推送 PaaS 平台架构与接入加固
  - 20.2.1 AWS SES / 阿里云 DirectMail / SendGrid / Mailgun 能力对比与选型
  - 20.2.2 RESTful API / SDK 鉴权（HMAC/API Key）与 Webhook 签名防伪造攻击
- **20.3** EDM 营销自动化与邮件活动（Mail Campaign）平台
  - 20.3.1 商业 SaaS（Mailchimp / Brevo）与开源平台（Mautic / Listmonk）拓扑接驳
  - 20.3.2 RFC 8058 (`List-Unsubscribe-Post`) 一键退订规范强行注入与合规
- **20.4** EDM 可达性（Deliverability）与安全防护
  - 20.4.1 双重确认（Double Opt-in）、订阅 Form 防 DDoS 验证码与 Rate-limit
  - 20.4.2 像素追踪（Tracking Pixel）HTTPS 签名与应对 Apple MPP 伪造点击

---

### 第七部分：信创生态、高可用架构与合规落地

#### 第二十一章 信创邮件系统架构解析

- **21.1** 信创邮件产品栈概述：Coremail、RichMail、TurboMail 等
- **21.2** 适配信创基础软硬件环境
  - 21.2.1 基于鲲鹏/飞腾/海光 CPU 与统信 UOS/银河麒麟 OS 的底层适配要点
  - 21.2.2 信创环境下的邮件系统高可用（HA）与容灾架构
- **21.3** 国产邮件客户端与移动协同端安全接入

#### 第二十二章 国密算法（SM2/SM3/SM4）在邮件体系中的落地

- **22.1** 国密算法与标准体系（GM/T 0024, GM/T 0006）
- **22.2** 国密双证书传输层协议 (TLCP) 在 SMTP/IMAP/POP3 中的实现
- **22.3** 国密 S/MIME（SM2 证书 + SM4 加密）终端与服务器接驳
- **22.4** 国密硬件密码机 (HSM) 与邮件鉴权/签名系统的对接

#### 第二十三章 等保 2.0 与关基（CII）合规加固实战

- **23.1** 等保 2.0（三级）对邮件系统的硬性指标拆解
- **23.2** 操作系统与数据库基线配置（Sysctl 优化、PAM 强密码策略、权限最小化）
- **23.3** 全链路审计日志标准化与 SIEM / SOC 系统对接
- **23.4** 应急响应、备份恢复与 RTO/RPO 保障策略

#### 第二十四章 企业级邮件系统高可用（HA）架构与灾难恢复（DR）实战

- **24.1** 邮件系统 HA/DR 理论模型与关键指标评估
  - 24.1.1 业务连续性指标拆解：RTO（恢复时间目标）、RPO（恢复点目标）与 SLA 等级划分
  - 24.1.2 邮件系统架构解耦：无状态组件（MTA/Webmail/Proxy）与有状态组件（Storage/Database/IMAP Session）的冗余设计原则
  - 24.1.3 全链路单点故障（SPOF）梳理与故障树分析（FTA）
- **24.2** 接入层与传输层（MTA）的高可用与负载均衡
  - 24.2.1 基于 DNS MX 记录权重的多入口 MTA 冗余、自动降级与队列回流机制
  - 24.2.2 HAProxy + Keepalived 四/七层负载均衡集群：SMTP (25/587/465)、IMAP/POP3 (993/995) 协议健康检查与 TLS 动态卸载
  - 24.2.3 Webmail 与 API 层高可用：Session 粘滞（Stickiness）、Redis 共享会话缓存与无状态扩退容
- **24.3** 存储层与后端检索引擎高可用集群建设
  - 24.3.1 Dovecot Cluster 架构：Dovecot Director 代理分发、dsync 实时双向同步与分布式 IMAP 集群
  - 24.3.2 共享/分布式文件系统在 Maildir/mdbox 场景下的选型与踩坑：NFSv4 锁机制、DRBD 块级同步与 Ceph FS 性能调优
  - 24.3.3 核心数据库高可用：MySQL/MariaDB Galera Cluster 多主同步、OpenLDAP 多主复制（Multi-Master Replication）与读写分离
- **24.4** 灾难恢复（DR）与异地多活/主备方案
  - 24.4.1 容灾等级与架构选型：冷备（Cold Standby）、温备（Warm Standby）与异地双活（Active-Active）代价平衡
  - 24.4.2 异地数据异步复制路径：块级别（DRBD/RBD）、文件级增量（`lsyncd`/`dsync`）与数据库 Binlog 跨机房传输
  - 24.4.3 GSLB（全局负载均衡）与 DNS 动态切流：基于健康检查的异地机房自动故障转移（Failover）
- **24.5** 裂脑（Split-Brain）防护、集群仲裁与灾备演练
  - 24.5.1 Corosync / Pacemaker 高可用集群仲裁机制（Quorum）与 STONITH/Fencing 节点强行隔离策略
  - 24.5.2 脑裂发生后的数据一致性修复、双写冲突解拆与日志审计追溯
  - 24.5.3 企业级灾备演练（DR Drill）SOP 制定：模拟光纤中断、主库宕机与数据秒级拉起验证

---

## 附录结构

### 附录 A：名词解释与技术词典（Glossary）

- 邮件传输与服务角色（MUA/MSA/MTA/MDA/MRA/LMTP/MLM/Webmail）
- 协同与自动化服务协议（Sieve/ManageSieve/CalDAV/CardDAV/iCalendar/vCard/Autodiscover）
- 身份校验、防冒用与加密（SPF/DKIM/DMARC/ARC/BIMI/OpenPGP/S/MIME/WKD/TSA/RMS/MIP/WORM）
- 动态防御、PaaS 与合规（Transactional Email/EDM/Deliverability/IP Warming/FBL/Fail2ban/CrowdSec/TLCP）

### 附录 B：企业级邮件与安全产品矩阵（Product Matrix）

涵盖 Postfix、Dovecot (Pigeonhole)、Proxmox Mail Gateway、Rspamd、GNU Mailman 3/Sympa、SabreDAV/Baïkal、Mailpiler、Roundcube/SnappyMail、GnuPG、HCL Domino、Zimbra Collaboration、Cisco IronPort、Proofpoint、AWS SES、阿里云 DirectMail、Mailchimp、Mautic、Listmonk、Coremail/RichMail、Microsoft Exchange Server 等 20+ 产品的架构定位、协议栈与核心安全能力。

### 附录 C：参考文献与标准规范（References & RFCs）

- IETF RFC：5321/5322 (SMTP)、3501/9051 (IMAP)、1939 (POP3)、5228 (Sieve)、5804 (ManageSieve)、4791/6352 (CalDAV/CardDAV)、3207 (STARTTLS)、8461/8460 (MTA-STS/TLS-RPT)、6698 (DANE)、7208/6376/7489/8617 (SPF/DKIM/DMARC/ARC)、4880/8551 (OpenPGP/S/MIME)、2369/8058 (List-Unsubscribe) 等
- 国家标准：GB/T 22239-2019（等保 2.0）、GB/T 39786-2021（密评）、GM/T 0024-2014（国密 TLCP）、GM/T 0006-2012（SM2/SM3/SM4 标识）

---

## 仓库结构

```
.
├── book/
│   ├── src/             # Pandoc 输入：每章一个 Markdown
│   ├── metadata.yml     # Pandoc 元数据 (title/author/date/lang)
│   ├── theme/           # 排版主题（HTML/CSS/LaTeX/landing）
│   ├── cover.svg/.png   # 封面
│   └── (构建产物不入仓：dist/ 见 .gitignore)
├── .github/workflows/   # GitHub Action：build PDF + ePub + HTML + Pages 发布
├── scripts/             # 构建辅助脚本（colophon 渲染、ePub 后处理）
├── .gitignore
├── LICENSE              # MIT
└── README.md
```

## 本地构建

需要：

- `pandoc` ≥ 3.1
- `texlive-xetex` + `texlive-lang-chinese` + `fonts-noto-cjk`

```bash
# 安装依赖 (Debian/Ubuntu)
sudo apt-get install -y pandoc texlive-xetex texlive-lang-chinese fonts-noto-cjk

# 构建 PDF
cd book
pandoc --pdf-engine=xelatex --metadata-file=metadata.yml \
       --include-in-header=theme/latex-header.tex \
       --toc --toc-depth=2 \
       --from=markdown+smart+footnotes+raw_attribute \
       --resource-path=src \
       -V documentclass=book -V papersize=a4 \
       src/00-preface.md src/01-*.md src/02-*.md src/03-*.md \
       src/04-*.md src/05-*.md src/06-*.md src/07-*.md \
       src/08-*.md src/09-*.md src/10-*.md src/11-*.md \
       src/12-*.md src/13-*.md src/14-*.md src/15-*.md \
       src/16-*.md src/17-*.md src/18-*.md src/19-*.md \
       src/20-*.md src/21-*.md src/22-*.md src/23-*.md \
       src/24-*.md src/99-*.md \
       -o ../dist/email-hardening.pdf

# 构建 ePub（需要 cover.png）
# rsvg-convert -w 1200 -h 1600 cover.svg -o cover.png
pandoc --to=epub3 --metadata-file=metadata.yml \
       --css=theme/epub.css \
       --toc --toc-depth=3 \
       --from=markdown+smart+footnotes+raw_attribute \
       --resource-path=src \
       --epub-cover-image=cover.png \
       src/00-preface.md src/01-*.md ... src/24-*.md src/99-*.md \
       -o ../dist/email-hardening.epub

# 构建 HTML
pandoc --to=html5 --standalone --metadata-file=metadata.yml \
       --toc --toc-depth=3 \
       --from=markdown+smart+footnotes+raw_attribute \
       --resource-path=src \
       --css=theme/html.css --self-contained \
       src/00-preface.md src/01-*.md ... src/24-*.md src/99-*.md \
       -o ../dist/email-hardening.html
```

## 发布流程

| 触发 | 行为 |
|---|---|
| Push 到 `main` | 自动构建 PDF / ePub / HTML；更新 GitHub Pages 站点 |
| `git tag v*` 并 push | 上述 + 附加到 GitHub Release |
| `workflow_dispatch` | 手动触发构建 |

## License

MIT — see [LICENSE](./LICENSE).

---

> © 2026 LeisureLinux. Built with Pandoc + XeLaTeX + GitHub Actions.
