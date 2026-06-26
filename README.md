# SRC Hunter — 实战漏洞挖掘工作流 Skill

> 基于 [MyuriKanao/src-hunter-skill](https://github.com/MyuriKanao/src-hunter-skill) 改编，MIT 许可。
> 仅用于授权的安全研究和合法的漏洞赏金 / SRC 测试。

---

## 📌 项目简介

**SRC Hunter** 是一个面向 AI Agent（如 Trae / OpenClaw / Claude）的**结构化漏洞挖掘工作流 Skill**。它将实战 SRC（Security Response Center）、Bug Bounty、众测的全流程拆解为 **5 个强制 checkpoint 阶段**，并内置 **19 类攻击 playbook、305+ 结构化 payload、国产组件指纹库、WAF 绕过变体及 HackerOne 真实案例**。

核心理念：**带强制 checkpoint 的可复现工作流，而非参考手册**。每个阶段有 MUST 输出，未通过不进下一阶段，杜绝 AI 幻觉和"凭记忆出 payload"的问题。

---

## 🏗️ 目录结构

```
src-hunter/
├── SKILL.md                              # 主入口：5 阶段工作流 + 触发条件 + 反幻觉约束
├── references/
│   ├── compliance.md                      # 合规与合法红线（Phase 5 前必读）
│   ├── dictionaries/
│   │   ├── chinese-srcfingerprints.md     # 国产组件指纹库（weaver/seeyon/tongda 等）
│   │   └ default-credentials-cn.md       # 国产系统默认凭据字典
│   ├── industry/
│   │   ├── banking-finance.md             # 银行/金融行业专项攻击路径
│   │   ├── telecom-isp.md                 # 运营商/BOSS/物联网专项
│   ├── methodology/
│   │   ├── 01-attack-priority.md          # 攻击路径最短原则（阻力评分）
│   │   ├── 02-bypass-toolkit.md           # WAF / EDR 绕过工具箱
│   │   ├── 03-evidence-discipline.md      # 证据纪律（防幻觉自检）
│   │   ├── 04-control-gap-hunting.md      # 控制缺口猎手方法论
│   │   ├── 05-srctimebox-priority.md      # SRC 时间盒优先级排序
│   ├── playbooks/                         # 19 类漏洞攻击 playbook
│   │   ├── sqli.md                        # SQL 注入（含 305+ payload）
│   │   ├── xss/                           # XSS（目录式，00-index + 子路由）
│   │   ├── rce/                           # RCE / SSTI / XXE / 反序列化
│   │   ├── ssrf-cache-host/               # SSRF / 缓存投毒 / Host 注入
│   │   ├── path-traversal/                # 路径穿越 / LFI / RFI
│   │   ├── file-upload/                   # 文件上传 + 解析漏洞
│   │   ├── arbitrary-x-authz.md           # 任意账号 / 越权 / IDOR
│   │   ├── logic-flaws/                   # 逻辑漏洞（密码重置/支付/验证码）
│   │   ├── oauth-saml-jwt/               # OAuth / SAML / JWT 认证绕过
│   │   ├── api-rest/                      # REST API / BOLA / Mass Assignment
│   │   ├── http-smuggling.md              # HTTP 请求走私
│   │   ├── race-conditions.md             # 竞态条件 / TOCTOU
│   │   ├── dos.md                         # DoS / ReDoS / 资源耗尽
│   │   ├── graphql.md                     # GraphQL introspection + 注入
│   │   ├── mobile.md                      # APK / IPA 移动端专项
│   │   ├── unauth-access.md               # 未授权访问 / 默认端口 / Swagger
│   │   ├── info-disclosure.md             # 信息泄露（.git/.env/heapdump）
│   │   ├── intranet-postexp/              # 内网后渗透（拿到 shell 后）
│   │   ├── llm-prompt-injection/          # LLM Prompt 注入
│   ├── templates/
│   │   ├── report-submission.md           # 漏洞报告提交模板（三段式）
│   │   └ tools/
│   │       └ mcp-jshook.md               # Burp Suite MCP 集成 + JS Hook
│   └── LICENSE                            # MIT License（建议添加）
│   └ README.md                            # 本文件
```

---

## 🔄 五阶段工作流

| 阶段 | 名称 | 核心任务 | MUST 输出 |
|------|------|----------|-----------|
| Phase 1 | **Intake（接单）** | 确认目标 scope、规则、时间盒 | in-scope / out-of-scope / 规则 / 时间盒四项清单 |
| Phase 2 | **Recon（被动侦察）** | 不发包给目标，纯 OSINT | 资产清单（≥3 种来源） |
| Phase 3 | **Enum（主动探测）** | 端口/服务/指纹/JS endpoint | 活资产矩阵 |
| Phase 4 | **Hunt（漏洞探测）** | 按 playbook 路由逐目标探测 | 可重现 finding + HTTP 包 |
| Phase 5 | **Report（提交）** | 合规自检 → 三段式报告 | 标题 / 重现步骤 / 影响+修复 |

每个阶段有 **强制 checkpoint**，不通过则不进下一阶段。

---

## 🧠 反幻觉硬约束

SRC Hunter 内置了四条反幻觉硬约束，确保 AI Agent 输出的可靠性：

1. **不准凭记忆出 payload** — 必须先 Read 对应 playbook 文件，payload 来自文件而非训练记忆
2. **不准编造案例编号** — 引用 H1 / WooYun 案例前必须读取实际文件
3. **无证据不下结论** — 无 HTTP 包 / 截图只能写"待验证"
4. **出 scope 立即停** — 发现资产不在 in-scope 列表立即停手

---

## 📋 19 类攻击 Playbook

| 编号 | Playbook | 覆盖漏洞类型 |
|------|----------|-------------|
| 1 | SQLi | SQL 注入（305+ payload，含 WAF 绕过变体） |
| 2 | XSS | 反射/存储/DOM XSS |
| 3 | RCE | SSTI / XXE / 反序列化 / 原型链 / 框架 RCE |
| 4 | SSRF | SSRF / 缓存投毒 / Host 注入 |
| 5 | Path Traversal | LFI / RFI / 路径穿越 |
| 6 | File Upload | 上传 + 解析漏洞绕过 |
| 7 | Arbitrary X AuthZ | 任意账号操作 / 越权 / IDOR / CSRF |
| 8 | Logic Flaws | 密码重置 / 支付 / 验证码 / 提现逻辑漏洞 |
| 9 | OAuth / SAML / JWT | 认证协议绕过（redirect_uri / SAML assertion / JWT 伪造） |
| 10 | REST API | BOLA / Mass Assignment / 速率限制 |
| 11 | HTTP Smuggling | CL.TE / TE.CL 请求走私 |
| 12 | Race Conditions | TOCTOU / 竞态条件 |
| 13 | DoS | ReDoS / 资源耗尽 / 算法爆炸 |
| 14 | GraphQL | introspection / injection / alias bypass |
| 15 | Mobile | APK / IPA 反编译 + API 硬编码提取 |
| 16 | Unauth Access | 默认端口 / Swagger / Actuator / 弱密码 |
| 17 | Info Disclosure | .git / .svn / .env / heapdump / 路径列举 |
| 18 | Intranet Post-Exp | 内网后渗透横向移动 |
| 19 | LLM Prompt Injection | AI Agent prompt 注入 / 工具调用劫持 |

---

## 🔧 特色能力

### 国产组件专项
- **指纹库**：覆盖泛微 / 致远 / 通达 / 兰图 / 用友 / 金蝶 / 海康 / 大华等国产 OA/IoT 系统
- **默认凭据字典**：国产系统常见默认账号密码集合
- **行业专项**：银行/金融、运营商/BOSS 独立攻击路径

### WAF 绕过工具箱
- 内置 WAF / EDR 绕过变体方法论（编码混淆 / 分块传输 / HTTP 走私 / 大小写 / 注释注入）

### Burp Suite MCP 集成
- 可通过 MCP 工具直接联动 Burp Suite（代理历史 / Repeater / Intruder / Collaborator / Scanner）

---

## ⚖️ 合规红线

本项目强制执行以下合规原则：

- **样本控制**：SQLi 探测到库名即停，不 dump 数据；IDOR 拉 1-3 条样本
- **自演越权**：用自己注册的两个号互测，不碰陌生人账号
- **只读不写**：RCE 只跑 `id` / `whoami`；Redis 未授权只 `info`
- **不真做副作用**：不真发短信 / 扣款 / 发邮件，200 状态码即停
- **凭据拿到不用**：泄露凭据仅验证，绝不实际操作
- **PII 脱敏**：报告中手机号 / 邮箱保留前 2 + 后 2 位
- **没抓包就没发现**：所有断言必须有 HTTP 包 / 截图 / 视频

---

## 🚀 使用方式

### 作为 Trae / OpenClaw Skill 使用

1. 将 `src-hunter` 目录放置到 Skills 目录（如 `.trae/skills/` 或 `.openclaw/workspace/skills/`）
2. 当对话中命中触发关键词（"SRC 挖洞"、"Bug Bounty"、"WAF 绕过"、"未授权访问" 等），Skill 自动激活
3. AI Agent 按 5 阶段工作流逐步推进，按需读取 references 中的 playbook 和 payload

### 关键触发词

- `src 挖洞 / 漏洞赏金 / bug bounty / 众测 / hackerone`
- `如何挖 / 怎么测 / 怎么打 + 目标`
- `WAF 绕过 / 任意账号 / 密码重置 / 未授权访问 / 默认凭据`
- 提供一个 URL / API endpoint 让测

---

## 📄 报告模板

内置三段式报告结构：

1. **标题**：≤80 字，精确到 endpoint + 漏洞类型
2. **重现步骤**：每步可执行，带 HTTP 包 / curl / 截图
3. **影响 + 修复建议**：CVSS vector + 业务影响段

模板文件：`references/templates/report-submission.md`

---

## ⚠️ 免责声明

本项目仅供**授权的安全研究和合法漏洞赏金 / SRC 测试**使用。任何未经授权的使用（包括但不限于对未授权目标的攻击、数据窃取、破坏行为）均由使用者自行承担全部法律责任。

---




