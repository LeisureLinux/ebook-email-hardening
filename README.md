# 邮件系统安全加固：从 SMTP 协议到国密与高可用架构

> 现代企业邮件与协同基础设施安全加固 —— 从 SMTP 协议到国密、云端 PaaS 与高可用架构。by [LeisureLinux](https://github.com/LeisureLinux).

[![build](https://github.com/LeisureLinux/ebook-email-hardening/actions/workflows/build.yml/badge.svg)](https://github.com/LeisureLinux/ebook-email-hardening/actions/workflows/build.yml)
[![license](https://img.shields.io/badge/license-MIT-blue.svg)](./LICENSE)

## 前言

> 在企业数字化基础设施与安全防御体系中，邮件系统始终占据着特殊而关键的技术地位。它既是企业最古老、最基础的异步通信协议（SMTP/POP3/IMAP）的集大成者，也是现代企业身份认证（IAM）、协同办公（CalDAV/CardDAV）、信息权利管理（DRM）以及合规审计的底层核心。

## 在线阅读 / 下载

| 格式 | 入口 |
|---|---|
| HTML 在线版 | <https://leisurelinux.github.io/ebook-email-hardening/> |
| PDF | [Releases](../../releases) 或 [直接下载](https://leisurelinux.github.io/ebook-email-hardening/email-hardening.pdf) |
| ePub | [Releases](../../releases) 或 [直接下载](https://leisurelinux.github.io/ebook-email-hardening/email-hardening.epub) |

## 完整目录

### 第一部分：邮件系统体系架构与安全术语图谱

- **第 1 章** — 邮件与协同系统拓扑与协议演进架构
- **第 2 章** — 邮件安全概念与规范术语全景图（Glossary）

### 第二部分：终端接入、协同服务与数据防泄露 (DLP) 加固

- **第 3 章** — 胖客户端（MUA）、自动配置与 DRM 权限控制
- **第 4 章** — Webmail 架构与前端安全攻防及动态水印
- **第 5 章** — 日历（Calendar）与通讯录（Contacts）协同服务安全
- **第 6 章** — 端到端加密 (E2EE) 与数字签名体系

### 第三部分：收件、过滤与存储层 (POP3/IMAP/Groupware) 加固

- **第 7 章** — POP3/IMAP 服务端组件架构（以 Dovecot 为例）
- **第 8 章** — 服务端邮件过滤引擎（Sieve）与规则安全
- **第 9 章** — 身份鉴权深度集成：PAM、LDAP 与数据库
- **第 10 章** — 企业协同群组平台与历史遗留系统加固
- **第 11 章** — 邮件存储层安全与合规归档

### 第四部分：传输层 (MTA)、邮件列表与反病毒网关集成

- **第 12 章** — 主流 MTA 架构与访问控制（以 Postfix 为核心）
- **第 13 章** — SASL 鉴权与传输链路加密硬化
- **第 14 章** — 域身份校验与反冒用自动化建设
- **第 15 章** — 邮件列表服务（Mailing List）安全与 ARC 信任链
- **第 16 章** — Milter 管道与反病毒网关（Anti-Virus Gateway）集成

### 第五部分：邮件安全网关 (SEG)、高级威胁与动态防护

- **第 17 章** — 开源邮件安全网关架构与集成
- **第 18 章** — 商业邮件安全网关（SEG）能力拆解与高级威胁防护
- **第 19 章** — 日志分析、动态防护与 SIEM/SOAR 联动

### 第六部分：云端 PaaS、事务邮件与 EDM 营销自动化加固

- **第 20 章** — 云端事务邮件推送 (PaaS) 与 EDM 营销自动化加固

### 第七部分：信创生态、高可用架构与合规落地

- **第 21 章** — 信创邮件系统架构解析
- **第 22 章** — 国密算法（SM2/SM3/SM4）在邮件体系中的落地
- **第 23 章** — 等保 2.0 与关基（CII）合规加固实战
- **第 24 章** — 企业级邮件系统高可用（HA）架构与灾难恢复（DR）实战

## 仓库结构

```
.
├── book/
│   ├── src/             # Pandoc 输入：每章一个 Markdown
│   ├── metadata.yml     # Pandoc 元数据 (title/author/date/lang)
│   ├── theme/           # 排版主题（HTML/CSS/LaTeX）
│   ├── cover.svg/.png   # 封面
│   └── (构建产物不入仓：dist/ 见 .gitignore)
├── .github/workflows/   # GitHub Action：build PDF + ePub + HTML
├── scripts/             # 构建辅助脚本
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

# 构建
cd book
pandoc --pdf-engine=xelatex --metadata-file=metadata.yml \
       --toc --toc-depth=3 \
       -V documentclass=book -V papersize=a4 \
       src/*.md -o ../email-hardening.pdf

pandoc --to=epub3 --metadata-file=metadata.yml \
       --toc --toc-depth=3 \
       --epub-cover-image=cover.png \
       src/*.md -o ../email-hardening.epub

pandoc --to=html5 --standalone --metadata-file=metadata.yml \
       --toc --css=theme/html.css --self-contained \
       src/*.md -o ../email-hardening.html
```

## 发布流程

| 触发 | 行为 |
|---|---|
| Push 到 `main` | 构建 PDF / ePub / HTML；更新 GitHub Pages |
| `git tag v*` 并 push | 上述 + 附加到 GitHub Release |
| `workflow_dispatch` | 手动触发 |

## License

MIT — see [LICENSE](./LICENSE).
