---
name: src-hunter
description: "实战 SRC / 众测 / Bug bounty 漏洞挖掘工作流。包含 5 阶段方法论（intake → recon → enum → hunt → report）、19 类攻击 playbook（SQLi/XSS/RCE/SSRF/IDOR/CSRF/Path Traversal/File Upload/SSTI/XXE/Race/HTTP Smuggling/OAuth/JWT/SAML/GraphQL/Mobile/LLM/DoS）、305 个结构化 payload、WAF/EDR 绕过变体、HackerOne 真实案例、国产组件指纹库。当用户提到 src 挖洞 / bug bounty / 众测 / 漏洞赏金 / SRC / 渗透测试 / 如何测某个目标 / WAF 绕过 / 任意账号 / 密码重置 / 未授权访问 时触发。"
---

# SRC Hunter — 实战漏洞挖掘工作流

这是一个**带强制 checkpoint 的工作流**，不是参考手册。每个阶段有 MUST 输出，未通过不进下一阶段。

详细 payload / playbook / H1 案例**按需 Read**，不准凭记忆生成。

## 触发条件

命中任一即进入：

- "src 挖洞 / 漏洞赏金 / bug bounty / 众测 / hackerone / Security Response Center"
- "如何挖 / 怎么测 / 怎么打 + 某目标 / 某接口 / 某参数"
- "WAF 绕过 / 任意账号 / 任意修改 / 密码重置 / 未授权访问 / 默认凭据"
- 用户给一个 URL / API endpoint 让你测

**不应触发**: 纯白盒源码审计；漏洞修复问答；CTF。

---

## 反幻觉硬约束（全程适用）

1. **不准凭记忆出 payload**。要给 SQLi/RCE/SSRF/XSS 任何 payload 前，先 Read 对应 `references/playbooks/<type>.md`。
2. **不准编造案例编号**。引用 H1/WooYun 案例前必须 Read 实际文件。
3. **无证据不下结论**。无 HTTP 包/截图时只能写"待验证"，不写"已确认"。
4. **出 scope 立即停**。任何时候发现资产不在 in-scope 列表 → 立即停手。

---

## Phase 1 · Intake（接单）

**进入条件**: 用户首次给出目标 / 程序名 / URL。

**MUST 输出 checkpoint**（四项缺一不进 Phase 2）：

- [ ] **In-scope**: 可测域名 / IP 段 / app / endpoint（逐条列）
- [ ] **Out-of-scope**: 禁测项（逐条列）
- [ ] **规则**: payout tier / disclosure window / safe-harbor
- [ ] **时间盒**: 6h / 单日 / HVV / 月度

仅当用户问"哪个最值得先测" → Read `references/methodology/05-srctimebox-priority.md`

---

## Phase 2 · Recon（被动侦察）

**进入条件**: Phase 1 checkpoint 四项全过。

**禁止**: 任何主动发包（端口扫描 / 路径爆破 / payload 测试）。

**MUST 输出**: 不发包给目标得到的资产清单 + 历史信息，来源 ≥3 种：

- CT 日志（crt.sh / Censys）
- Wayback / CommonCrawl 历史快照
- GitHub dorks（`org:target` + `password|api_key|SECRET|.env`）
- FOFA / Shodan favicon hash
- SecurityTrails / DNS 历史
- ASN / IP 段（bgp.he.net）

---

## Phase 3 · Enum（主动探测）

**进入条件**: Phase 2 资产清单非空。

**MUST 输出**: 活资产矩阵——`域 → 端口 → 服务 → 指纹 → JS endpoint`。

**条件触发 Read**：

| 命中信号 | MUST Read |
|---|---|
| 指纹含 weaver/seeyon/tongda/landray/yongyou/kingdee/hikvision/dahua | `references/dictionaries/chinese-srcfingerprints.md` + `references/dictionaries/default-credentials-cn.md` |
| 资产含 银行 / 支付 / 网银 | `references/industry/banking-finance.md` |
| 资产含 运营商 / BOSS / 物联网卡 | `references/industry/telecom-isp.md` |

---

## Phase 4 · Hunt（漏洞探测）

**进入条件**: Phase 3 矩阵 ≥1 个候选目标。

**强制流程（每个候选目标走一遍）**：

1. 看目标信号，从下表选 1 个 playbook
2. **Read 该 playbook 文件**（不准跳过、不准凭记忆替代）
3. 按 playbook 的"参数频率表"挑入口
4. 按 playbook 的"payload 库"探测——payload 来自文件，不来自训练记忆
5. 被 WAF 拦 → Read `references/methodology/02-bypass-toolkit.md`
6. 命中后立即保存 HTTP 包 / 截图 → 进 Phase 5 候选

### Playbook 路由表

| 入口信号 | MUST Read |
|---|---|
| Actuator / Swagger / 默认端口 / 弱密码 | `references/playbooks/unauth-access.md` |
| .git / .svn / .env / heapdump / 路径列举 | `references/playbooks/info-disclosure.md` |
| 用户态 ID 可遍历 / 任意 X 越权 | `references/playbooks/arbitrary-x-authz.md` |
| 密码重置 / 支付 / 验证码 / 订单 / 提现 | `references/playbooks/logic-flaws/00-index.md` |
| OAuth / SAML / JWT / redirect_uri | `references/playbooks/oauth-saml-jwt/00-index.md` |
| REST API / BOLA / Mass Assignment / 速率 | `references/playbooks/api-rest/00-index.md` |
| 任何用户输入进 DB | `references/playbooks/sqli.md` |
| 反序列化 / SSTI / XXE / 原型链 / 框架 RCE | `references/playbooks/rce/00-index.md` |
| URL 入参 / 缓存 / Host 注入 | `references/playbooks/ssrf-cache-host/00-index.md` |
| 文件路径入参 / LFI / RFI | `references/playbooks/path-traversal/00-index.md` |
| 上传点 + 解析漏洞 | `references/playbooks/file-upload/00-index.md` |
| 用户输入回显到 HTML / JS | `references/playbooks/xss/00-index.md` |
| 反代 + Content-Length / TE | `references/playbooks/http-smuggling.md` |
| GraphQL endpoint / introspection | `references/playbooks/graphql.md` |
| 并发 / TOCTOU | `references/playbooks/race-conditions.md` |
| ReDoS / 资源不限速 / 算法爆炸 | `references/playbooks/dos.md` |
| APK / IPA / 移动端 | `references/playbooks/mobile.md` |
| LLM agent / prompt 入口 / 工具调用 | `references/playbooks/llm-prompt-injection/00-index.md` |
| 已拿到 shell / 凭据 / 内网 | `references/playbooks/intranet-postexp/00-index.md` |

**两步 Read 模式**：目录形式的 playbook 第一步只 Read `00-index.md`（含子文件路由表），再根据路由定位到具体子文件。

### 通用方法论（卡壳时才 Read）

| 场景 | Read 文件 |
|---|---|
| 不知道下一步打什么 | `references/methodology/01-attack-priority.md` |
| 被 WAF / EDR 拦 | `references/methodology/02-bypass-toolkit.md` |
| 怀疑幻觉 / 检查证据链 | `references/methodology/03-evidence-discipline.md` |
| 找不到漏洞点 | `references/methodology/04-control-gap-hunting.md` |

---

## Phase 5 · Report（提交）

**进入条件**: Phase 4 至少一个 finding 已具备可重现 HTTP 包 / 截图 / 视频。

**MUST 流程**（顺序执行）：

1. Read `references/compliance.md` 核对合规红线（不准跳）
2. Read `references/templates/report-submission.md` 取模板
3. 三段式输出：
   - **标题**: ≤80 字，精确到 endpoint + 漏洞类型
   - **重现步骤**: 每步可执行，带 HTTP 包 / curl / 截图
   - **影响 + 修复建议**: CVSS vector + 业务影响段

---

## MCP 工具集成

本 Skill 可配合 Burp Suite MCP 使用：
- `mcp_burp_get_proxy_http_history` — 查看代理历史流量
- `mcp_burp_send_http1_request` — 发送 HTTP 请求
- `mcp_burp_create_repeater_tab` — 创建 Repeater 标签页
- `mcp_burp_send_to_intruder` — 发送到 Intruder 批量测试
- `mcp_burp_generate_collaborator_payload` — 生成 OOB 测试 Payload
- `mcp_burp_get_scanner_issues` — 查看扫描器发现的漏洞

在 Hunt 阶段优先使用 Burp MCP 工具进行请求发送和响应分析。

---

## 合规红线（全程强制）

- **样本控制**: SQLi 探测到库名即可证明，不要 dump 数据；IDOR 拉 1–3 条样本就够
- **测试账号自演**: 越权、密码重置全部用自己注册的两个号互测，不要碰陌生人的账号
- **只读，不写**: 拿到 RCE 只跑 `id` / `whoami`；Redis 未授权只 `info`
- **不真做副作用动作**: 不真发短信、不真扣款、不真发邮件。证明能调通 + 200 即停
- **DoS / 并发**: 单次复现 ≤60s，串行做 5 次足够
- **不留物**: webshell、dump 源码——本地保存，报告后立即删除
- **凭据:拿到不用**: 泄露的凭据仅验证，绝不用于实际操作
- **报告里所有 PII 脱敏**: 手机号、邮箱留前 2 + 后 2
- **没抓包就没发现**: 所有断言都要有 HTTP 包 / 截图 / 视频

> 本 Skill 基于 [MyuriKanao/src-hunter-skill](https://github.com/MyuriKanao/src-hunter-skill) 改编，原项目 MIT 许可。
> 仅用于授权的安全研究和合法的漏洞赏金 / SRC 测试。
