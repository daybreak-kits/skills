# skills — Agent Skills 合集

> 一组遵循通用 **Agent Skills 规范**的 AI agent 技能集：每个 skill 是一个目录 + 一份带 frontmatter 的 `SKILL.md`，覆盖编码、代码审计、CLI 接口设计、深度调研、正式文档写作、工作 PPT 创作、README 写作；并附带一套 agent 提示词（`agents/`）。不绑定任何特定 agent——Claude Code、OpenCode、Cursor、Codex 或任何支持该规范的工具都能加载。

[![license: Apache-2.0](https://img.shields.io/badge/license-Apache--2.0-green)](./LICENSE) [![spec: Agent Skills](https://img.shields.io/badge/spec-Agent%20Skills-blue)](https://docs.claude.com/en/docs/agents-and-tools/agent-skills) [![repo: daybreak-kits/skills](https://img.shields.io/badge/repo-daybreak-kits%2Fskills-blue)](https://github.com/daybreak-kits/skills)

---

## 设计原则（2026-10 版）

- **意图优先于步骤**：先讲“要什么效果、为什么”，步骤是实现意图的手段，不是意图本身。
- **红线与自主区两分**：`以下必须遵守`（少数硬红线）与`以下由你自主判断`（含判断依据的授权区）；未标注的模糊地带会让代理过度收紧。
- **验证手段，不是质量保证**：不写“确保正确”这类不可执行的承诺；写“跑测试、贴输出”这类可自验的动作。
- **规则可溯源**：每条规则都能对应到一个已观察到的失败；未观察到失败的规则不立。
- **单一事实源**：`SKILL.md` 是唯一事实源；`README.md` 只讲“是什么 / 怎么装 / 怎么调用 / 交付什么”，不复述规则。

---

## 技能一览

| Skill | 干什么 | 典型场景 |
| --- | --- | --- |
| [coding](./coding/) | 编码自律框架：先理解后实施、外科手术式修改、硬红线约束 | 开发新功能、修 bug、重构防 diff 蔓延 |
| [code-audit](./code-audit/) | 只读代码审计：7 维度 + 三项对位 + 阻塞/主要/次要分级，问题定位到行号 | PR Review、合并前风险把关、技术债盘点 |
| [cli-design](./cli-design/) | 把 CLI 当公共契约治理：接口盘点、命名一致、弃用有周期 | 新增命令/标志、重构存量 CLI、弃用治理 |
| [deep-research](./deep-research/) | 决策级深度调研：任务卡 → 探针搜索 → 决策，产出可拍板报告 | 选型决策、竞品/投资调研、战略分析 |
| [office-docx-design](./office-docx-design/) | 正式工作文档写作：报告 / 方案 / SOW / 制度 / 汇报 / 复盘 | 年度汇报、SOW 编制、项目复盘 |
| [office-pptx-design](./office-pptx-design/) | 工作类 PPT 创作：受众澄清、调研核查、结论式大纲、逐页内容与讲稿 | 年终汇报、培训课件、方案演示 |
| [readme-writer](./readme-writer/) | README 写作方法论：按项目类型定制 + 可验证的验收清单 | 新建仓库缺 README、改写现有文档 |

每个 skill 目录内均有独立的 `README.md`（是什么 / 怎么装 / 怎么调用）与 `SKILL.md`（完整规则，唯一事实源），部分 skill 附带 `references/` 深度参考与 `scripts/` 工具脚本。

---

## Agent 提示词（agents/）

| Agent | 角色 | 返回状态词表 |
| --- | --- | --- |
| [leader](./agents/leader.md) | 主控：理解需求、编排、验收、汇报；只产出“清晰”，不产交付物正文 | `确认完成 \| 局部完成 \| 任务阻塞` |
| [worker](./agents/worker.md) | 通用执行：路径清楚、有成熟做法的实现与修复 | 同上 |
| [solver](./agents/solver.md) | 攻坚裁决：反复失败的根因、独立验收高风险交付 | 同上 |
| [seeker](./agents/seeker.md) | 信息探索：联网调研 / 本地侦察 / 事实核对，只回证据不裁决 | 同上 |

四个 agent 共享同一套状态语言（`确认完成 \| 局部完成 \| 任务阻塞`）、同一套缺陷分级（`阻塞 \| 主要 \| 次要`）与“其他事项”信息通道——执行体范围内看到的、不做的事写这里，由 leader 逐条接收，不得静默丢弃。

---

## 快速开始

### 安装（通用）

SKILL 遵循通用的 Agent Skills 规范（一个目录 + 一份带 frontmatter 的 `SKILL.md`）。把想要的 skill 目录复制进你所用工具的 skills 目录即可：

```bash
cp -r skills/<skill-name>/ <你的 agent 的 skills 目录>/
```

常见 agent 的 skills 目录：

| Agent | 用户级（全局可用） | 项目级（团队共享同一版本，推荐） |
| --- | --- | --- |
| Claude Code | `~/.claude/skills/` | `<project>/.claude/skills/` |
| OpenCode | `~/.config/opencode/skills/` | `<project>/.opencode/skills/` |
| 其他支持 Agent Skills 的工具 | 按其文档指定的 skills 目录 | 同上（团队共享同一版本） |

> **不安装也能用**：直接在 prompt 里给出 `SKILL.md` 的绝对路径，让 agent 加载即可。

自检（以 Claude Code 用户级为例）：`ls ~/.claude/skills/<skill-name>/SKILL.md`

### 调用方式

- **自动触发**：agent 读取 `SKILL.md` frontmatter 的 `description`，命中任务类型时自动加载（各 skill 的触发词见其 README）。
- **显式加载**：在 prompt 中给出 `SKILL.md` 绝对路径并要求严格执行，适合强制启用或调试。
- **prompt 纪律**：只给需求 + 路径 + 约束，**不要复述 SKILL 正文规则**——复述会稀释 skill 自身的约束力。

---

## 目录结构

```
skills/
├── agents/                 # agent 提示词（leader / worker / solver / seeker）
├── coding/                 # 编码自律框架
│   ├── SKILL.md
│   └── references/         # 代码组织 / 原则取舍
├── code-audit/             # 只读代码审计
│   ├── SKILL.md
│   └── references/         # 审计维度清单
├── cli-design/             # CLI 接口设计与治理
│   └── SKILL.md
├── deep-research/          # 决策级深度调研
│   ├── SKILL.md
│   ├── README.md / README.en.md
│   ├── references/         # 探针搜索 / 假设红队 / 报告渲染
│   └── scripts/            # research_tools.py（ledger / audit 工具链）
├── office-docx-design/     # 正式工作文档写作
│   ├── SKILL.md
│   └── references/         # 文体指南 / 风格指南 / 范例 / 表格规范
├── office-pptx-design/     # 工作类 PPT 创作
│   └── SKILL.md
└── readme-writer/          # README 写作方法论
    └── SKILL.md
```

---

## 贡献

欢迎通过 Issue / PR 补充新 skill 或改进现有规则。新增 skill 请保持结构：`SKILL.md`（必填 frontmatter：`name` + `description`，description 写清触发场景）+ 可选 `references/`、`scripts/`，并附一份 README。

## 许可证

[Apache-2.0](./LICENSE)

---

*Last updated: 2026-10-30*
