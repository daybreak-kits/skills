# Coding — 编码工作流的自律框架

> 让 AI agent 写出的每一行改动都**精准、可验证、不制造结构混乱**。它不是风格指南，而是一套「边界 + 心法 + 工作循环」：边界之内给足技术自主权，触碰边界就说明理由。
>
> 核心四句话：**先理解后实施 · 单一事实源 · 复用有门槛 · 外科手术式修改**。

[![skill: coding](https://img.shields.io/badge/skill-coding-purple)](./SKILL.md) [![license: Apache-2.0](https://img.shields.io/badge/license-Apache--2.0-green)](../LICENSE) [![返回主目录](https://img.shields.io/badge/repo-daybreak-kits%2Fskills-lightgrey)](https://github.com/daybreak-kits/skills)

> **规则以 [`./SKILL.md`](./SKILL.md) 为唯一事实源**，本文只讲这是什么、怎么装、怎么调、交付什么。

## 这是什么 / 给谁用

**不是**代码风格指南、语言规范、框架教程、lint 规则集——它不规定你用 tabs 还是空格。**而是**给编码 agent 的行为约束层：**动手前的侦察义务**（找现有实现 / 找唯一事实源 / 找周围惯例）、**动手时的克制边界**（只改该改的、不建平行实现、不顺手重构）、**交付时的汇报契约**（改了什么 / 怎么验证 / 没验证什么 / 剩余风险）。

| 适合谁 | 典型诉求 |
| --- | --- |
| 用 AI agent 写生产代码的工程师 | 受够了 agent 造 `xxx_v2`、乱建 `utils2.py`、一次 PR 蔓延 20 个文件 |
| 需要 review AI 产出 diff 的人 | 希望 diff 小而聚焦，每行改动都有理由 |
| 功能开发 / bug 修复 / 重构 | 长链条任务中防止目标漂移、防止改错地方 |
| 团队协作仓库 | 让 agent 遵守项目既有惯例，而不是它自己的审美 |

**不适合**：纯文档编写、配置微调、非代码类问答（frontmatter 已明确排除）。

## 快速开始

### 安装

本 skill 遵循通用的 **Agent Skills 规范**（一个目录 + 一份带 frontmatter 的 `SKILL.md`），不绑定任何特定 agent。把目录放进你所用工具的 skills 目录即可：

```bash
git clone https://github.com/daybreak-kits/skills.git
cp -r skills/coding/ <你的 agent 的 skills 目录>/
```

常见 agent 的 skills 目录：

| Agent | 用户级（全局可用） | 项目级（团队共享同一版本，推荐） |
| --- | --- | --- |
| Claude Code | `~/.claude/skills/` | `<project>/.claude/skills/` |
| OpenCode | `~/.config/opencode/skills/` | `<project>/.opencode/skills/` |
| 其他支持 Agent Skills 的工具 | 按其文档指定的 skills 目录 | 同上（团队共享同一版本） |

> 不安装也能用：直接在 prompt 里给出 `SKILL.md` 的绝对路径，让 agent 加载即可。

自检（以 Claude Code 用户级为例）：`ls ~/.claude/skills/coding/SKILL.md`（应输出路径不报错）。触发方式：agent 依据 `SKILL.md` frontmatter 的 `description` 自动触发；也可在 prompt 中显式给出绝对路径加载。

### 最小调用（给 agent 的 prompt 模板）

```
你是一名遵守编码自律框架的工程师。

任务：[要开发的功能 / 要修的 bug / 要做的重构]
代码库：[绝对路径]

加载 SKILL（必须）：<你的 skills 目录>/coding/SKILL.md

严格按 SKILL.md 执行，尤其是「动手前三次搜索」「先复现再定位后修改」「外科手术式修改」。
完成后按 SKILL.md 的 4 项完成汇报契约输出。
```

### 预期输出

1. **代码改动** — 小而聚焦的 diff，只含任务范围内的修改。
2. **验证证据** — 跑过的测试 / lint / 复现步骤结果；没验证的要明说为什么。
3. **完成汇报**（4 项必填）— 改了什么 · 怎么验证的 · 没验证什么 · 剩余风险与建议。

成功判据：代码能跑 + 侦察时写下的成功标准全部满足 + diff 逐行自审无计划外文件、无调试残留。

## 覆盖什么

- **五条红线**：禁止平行实现 / 秘密不落盘 / 不可逆操作先确认 / 不静默吞错 / 改签名先列调用方。
- **授权三段**：自由决定 · 先说明理由再动手 · 停下来问。能查清的事自己查，不推回用户。
- **工程准则**：恰到好处 · 结构纪律 · 复用有门槛 · 外科手术。启发式，有更好理由可偏离，说明即可。

细则见 [`./SKILL.md`](./SKILL.md)；新建文件判定与公共代码准入见 [`./references/code-organization.md`](./references/code-organization.md)；原则打架时听谁的（一致性 > 可读性 > YAGNI > KISS > 外科手术 > 显式失败 > 组合 > 测量）见 [`./references/principle-tradeoffs.md`](./references/principle-tradeoffs.md)。

## 输入 / 输出契约

| 方向 | 项 | 必填 | 说明 |
| --- | --- | --- | --- |
| 输入 | 任务描述 | ✓ | 新功能 / bug 修复 / 重构 |
| 输入 | 代码库路径 | ✓ | agent 需读到真实代码才能做动手前侦察；成功标准未给出时自行确定 |
| 输出 | 代码改动 | ✓ | 源码 diff（路径由任务决定，本 skill 不规定产物路径） |
| 输出 | 完成汇报 | ✓ | 4 项固定契约：改了什么 / 怎么验证的 / 没验证什么 / 剩余风险与建议 |

## 使用场景 / Anti-Patterns

| 反模式 | 为什么坏 |
| --- | --- |
| 跳过侦察直接写代码 | 制造重复实现，后续所有修改都可能落在死代码上 |
| 改完现象没变就继续叠加改动 | 大概率改错了地方；应回到执行路径验证 |
| 建 `utils2.py` / `helpers.py` 这类垃圾抽屉 | 按技术动作碎片化，职责无处安放 |
| 把被多次 import 的业务函数挪进 common | 流行度 ≠ 通用性，业务规则散落到无人认领的地方 |
| 内部纯逻辑包 try/except、加 fallback | 掩盖本应修复的问题，把一次性成本变成长期排查成本 |
| 顺手重构任务范围外的代码 / 重试 3 次仍盲改 | 违反外科手术纪律、diff 膨胀；卡住时应停止并汇报已排除的假设 |

## 目录结构（纯文本规范，无脚本、无依赖）

```
coding/
├── SKILL.md                    # 目标与授权 / 先复现后修改 / 红线 / 工作循环 / 工程准则 / 汇报
└── references/
    ├── code-organization.md    # 新建文件判定 · 公共代码准入 · 重复实现 · 抽象时机
    └── principle-tradeoffs.md  # 原则冲突仲裁表（按优先级排序）
```

## 许可证

本项目基于 [Apache License 2.0](../LICENSE) 发布。欢迎到 [daybreak-kits/skills](https://github.com/daybreak-kits/skills) 提 Issue 或 PR。

*Last updated: 2026-10-29*
