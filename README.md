# 公司与客户调研

`company-research`（v4.1）是一份 Agent Skill：把公开信息与用户提供的一手材料整理成可追溯、可快速决策的报告，贯通公司总览、业务、技术、财务、商业质量与风险。界面显示名为「公司与客户调研」。正式触发名是 `$company-research`；旧名称 `$customer-research` 由独立兼容入口转发到本 Skill，新提示词与新自动化使用正式名称。

## 功能、模式与深度

`company` 模式用业务报告说明公司怎样经营，用技术报告说明产品承诺怎样实现，再用贯通矩阵核对两者与财务、变现是否一致。信息不足时缩小结论，并在报告中保留缺口。`company` 的读者可见结构是五个 H2：「公司总览」「业务报告」「技术报告」「财务、商业质量与风险」「来源与信息缺口」。业务报告与技术报告可独立阅读，默认放在同一份文件里。

| 模式 | 适用场景 | 主要产物 |
|---|---|---|
| `company` | 了解经营企业、集团、机构或品牌 | 含可独立阅读的业务报告与技术报告的一体化公司报告 |
| `client` | 客户销售、采购、合作、续签或大单推进 | 公司底图，加上机会、决策链、会议问题与议程 |
| `investor` | 拜访投资机构、融资或维护关系 | 机构底图，加上基金、投资节奏、合伙人与匹配 |

默认选择：只有公司名且目标是了解这家公司时用 `company`；有客户拜访、采购或方案切入时用 `client`；对象是投资机构且目标是融资或募资沟通时用 `investor`。

| 深度 | 选择规则 | 研究强度 |
|---|---|---|
| `quick` | 用户要求快速、简单或最低档；或只给公司名且无重大决策场景 | 用少量高价值来源覆盖核心维度，缺项留空 |
| `standard` | 用户要求完整、标准或合作评估；`client` / `investor` 未指定深度时的默认 | 关键结论交叉验证，并写入竞对与最近变化 |
| `deep` | 用户明确要求深度、尽调、投委会、董事会，或以财报为重点 | 展开集团边界、连续财期、分部、竞争与对抗审查 |

深度控制篇幅与证据密度，各深度共用同一事实标准。`quick` 同样不得用融资额、估值、注册资本或媒体估算代替营收。

`agents/openai.yaml` 允许隐式调用（`allow_implicit_invocation: true`）。其默认提示为：使用 `$company-research` 调研主体；公司模式输出五模块整合报告，拜访模式补充客户或投资人决策信息。

## 安装与调用

本仓库以 `SKILL.md` 为技能入口。`SKILL.md` 中记录的 Codex 技能路径形如 `~/.codex/skills/<技能名>/SKILL.md`。按此约定，将本仓库克隆或复制到与 frontmatter `name` 一致的目录：

```bash
mkdir -p ~/.codex/skills
git clone https://github.com/BMBWaveforms/company-research.git ~/.codex/skills/company-research
```

技能名、安装目录与 GitHub 仓库名都是 `company-research`。其他遵循同一 `SKILL.md` 目录约定的运行时，把包含 `SKILL.md` 的仓库根目录放进其 skills 目录即可。

在对话中用 `$company-research` 触发。最小输入是待研究主体；未说明调研目的时，按「快速了解公司」处理。可选输入包括官网、股票代码、地区、法人全称或品牌名、目的与关注问题、`mode`、`depth`、输出路径，以及年报、招股书、白皮书、会议纪要、合同、RFP 或用户指定的竞对。

## 仓库结构

```
SKILL.md                          技能说明、模式边界、Gate 与交付规则（v4.1）
company_report_template.html      公司报告单文件 HTML 模板
agents/openai.yaml                显示名、短描述、默认提示与隐式调用策略
references/                       运行时按模式读取的规范
scripts/build_report_html.py      由定稿 Markdown 生成 HTML
scripts/audit_client_report.py    审计 Markdown / HTML 交付
evals/cases.json                  Loading、Hero Query、hard gate 与 Judge
```

运行时读取：

- 所有模式先完整读取 `references/company-research-framework.md`。
- `references/report-spec.md` 仅用于 `company`；其中五模块结构与 frontmatter 不用于 `client` / `investor`。
- `references/company-html-layout.md`：生成、重写或验收公司报告 HTML。
- `references/client-investor-extension.md`：`client` 或 `investor`。
- `references/discovery-question-quality-standard.md`：生成或修复会议问题。
- 用户本轮提供的年报、会议纪要、RFP、合同、邮件、产品手册、白皮书或截图优先于历史 reference。

## 报告生成与审计

Markdown 是唯一内容真源。先定稿 MD，再生成 HTML。用户指定路径时遵从；未指定时：

- `company`：`~/Desktop/公司调研/{企业简称}_公司研究/`，文件名为 `{企业简称}_公司调研_vN.md` / `.html`
- `client`：`~/Desktop/客户调研/{企业简称}_{目的}/`
- `investor`：`~/Desktop/投资人调研/{机构简称}_{目的}/`
- `client` / `investor` 的文件名为 `{主体简称}_拜访作战地图_vN.md` / `.html`

`{skill_dir}` 是上面的技能安装目录：

```bash
python3 {skill_dir}/scripts/build_report_html.py <MD绝对路径>
python3 {skill_dir}/scripts/audit_client_report.py <MD绝对路径> <HTML绝对路径> --strict
```

HTML 使用 `company_report_template.html`，完整内嵌同一份 UTF-8 Markdown，并遵守 `references/company-html-layout.md`。两个脚本只依赖 Python 3 标准库。

用户明确只交付 Markdown 时，不生成 HTML，改为：

```bash
python3 {skill_dir}/scripts/audit_client_report.py <MD绝对路径> --strict --markdown-only
```

`quick` 至少通过与交付格式对应的严格静态审计。`standard` / `deep`，或 HTML 布局、交互、资源发生变化时，再做真实浏览器的桌面与移动端抽查。

`--allow-legacy-company` 只用于回读 v4.0 之前的旧版 company 章节布局。新建或更新公司报告不带该参数。它只放行旧布局，主体、财务、来源和证据检查仍然生效。

同一主体重跑时先读现有 MD，保留旧版本并递增版本号。用户要求 Word、PDF 或飞书时，派生载体仍以定稿 MD 为准。飞书仅在用户明确要求时，再读取 `~/.codex/skills/feishu-rich-doc/SKILL.md` 并按其写入规则呈现同一份定稿。

## 边界与注意事项

下列请求交给对应 Skill，或直接回答，不启动本 Skill 的完整报告：

| 场景 | 去向 |
|---|---|
| 岗位、JD、候选人或面试目标 | `interview-research` |
| 已拿到 Offer，决定接、谈或拒 | `offer-auditor` |
| 单一产品的功能、用户、体验或定价 | `product-analyst` |
| 专利检索、IPC、专利族、权利要求、FTO 或专利组合 | `tech-mining` |
| 只查注册地址、成立时间或一个财务数字 | 直接查询 |

公司报告里的少量专利只作为技术证据；专利阶段、IPC、权利要求、FTO 及其专用脚本由 `tech-mining` 承担。

写作时保持这些口径（完整条目见 [SKILL.md](SKILL.md) 的「Gotchas」与「模式与边界」）：

- 注册资本、融资额、估值与营收分开书写；经监管或审计披露的数字保留精确值、期间、币种、单位和口径。
- 私营主体没有公开财报时，可报告融资、招投标、合同、招聘、定价、客户和经营代理信号，并写明财务缺口。
- 招聘信息证明岗位需求与投入方向；官网、发布会和白皮书证明公司这样表述；产品功能证明用户可见能力。架构、模型来源、数据流、部署形态、自研程度、客户采用和规模化收入需要各自的原始证据，不足时降级为可确认能力、技术信号或公开未知。
- 母公司财报、集团融资和品牌用户数只在有归属证据时写入目标法人。历史材料只解释演变。
- 「公开渠道未见」只描述本轮检索范围。
- 集团、上市母公司、境内运营主体和单一品牌的数据保持分口径，不拼成一个确定总数。

## 评测

回归用例在 [`evals/cases.json`](evals/cases.json)（`version` 4.1，`skill` 为 `company-research`）。文件包含 Loading 正例与负例、全部 Hero Query、hard gates，以及 company 模式的 100 分 Judge。通过规则是 `score >= 85` 且没有 hard gate 失败。`client` / `investor` 的 Hero Query 使用各自期望断言，加上全模式及适用的 HTML hard gates。`judge_rubric` 的五模块评分范围是 `company`。

修改 description、模式边界、财务口径、贯通矩阵或报告 Gate 后，用该文件回归 Loading 正负例、全部 Hero Query 和 100 分 Judge。仓库未附带评测运行器。

## 许可

本仓库未声明许可证：根目录没有 `LICENSE` 文件，GitHub 上的 license 信息为空。
