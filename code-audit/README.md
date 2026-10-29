# code-audit —— 证据驱动的只读代码审计技能

> 把"帮我 review 一下"变成**可定位、可追溯、可判定是否阻塞发布**的审计报告。每条结论必须落到 `文件:行号`，没有证据的评价不进报告。
> 给需要判断"这次改动能不能合"的人用：开发者、Reviewer、Tech Lead。

[![skill: code-audit](https://img.shields.io/badge/skill-code--audit-purple)](./SKILL.md) [![license: Apache-2.0](https://img.shields.io/badge/license-Apache--2.0-green)](../LICENSE) [![repo: daybreak-kits/skills](https://img.shields.io/badge/repo-daybreak-kits%2Fskills-blue)](https://github.com/daybreak-kits/skills) [![back to skills](https://img.shields.io/badge/back%20to-skills-lightgrey)](../..)

> **规则以 [`./SKILL.md`](./SKILL.md) 为唯一事实源**，本文只讲这是什么、怎么装、怎么调、交付什么。

## 这是什么 / 给谁用

**不是**：自动修 bug 的机器人、风格挑刺机、linter 的替代品、凑条数的"15 条真假混杂"评论。

**而是**：一套**只读**审计行为准则。起点假设是"当前代码可能存在问题"，但证据不支持任何发现时，如实输出"未发现问题"就是合格产出——为了让报告显得充实而制造边缘问题，比漏报更糟。终点不是"代码漂亮"，而是**可验证的正确性 + 可接受的风险敞口**。

| 适合谁 | 典型场景 |
| --- | --- |
| 开发者 | 提交前自检：这次改动有没有埋雷、调用方是否都同步了 |
| Reviewer / PR Review | 拿到大 diff，快速区分"阻塞发布"与"可选优化" |
| Tech Lead / 架构师 | 评估技术债务、无效代码、重复代码的处置优先级 |
| 项目负责人 | 验证改动是否真的达成了业务目标（需求双向校验） |

**不适合**：单文件语法检查、格式化建议（不触发本 skill）；需要 AI 直接动手改代码的任务——本 skill **只读**，只描述与建议，不代为修复。

## 快速开始

### 安装

本 skill 遵循通用的 **Agent Skills 规范**（一个目录 + 一份带 frontmatter 的 `SKILL.md`），不绑定任何特定 agent。把目录放进你所用工具的 skills 目录即可：

```bash
git clone https://github.com/daybreak-kits/skills.git
cp -r skills/code-audit/ <你的 agent 的 skills 目录>/
```

常见 agent 的 skills 目录：

| Agent | 用户级（全局可用） | 项目级（团队共享同一版本，推荐） |
| --- | --- | --- |
| Claude Code | `~/.claude/skills/` | `<project>/.claude/skills/` |
| OpenCode | `~/.config/opencode/skills/` | `<project>/.opencode/skills/` |
| 其他支持 Agent Skills 的工具 | 按其文档指定的 skills 目录 | 同上（团队共享同一版本） |

> 不安装也能用：直接在 prompt 里给出 `SKILL.md` 的绝对路径，让 agent 加载即可。

自检（以 Claude Code 用户级为例）：`ls ~/.claude/skills/code-audit/SKILL.md`

### 最小调用（给 Agent 的 prompt 模板）

```
# 不要在 prompt 里复述 SKILL 的规则——会污染加载效果，只给范围 + 目标 + SKILL 路径。
你是一名代码审计专家，对下述变更做一次系统性审计。

审计范围：[文件路径 / 目录 / PR 链接 / commit 区间]
业务目标：[本次改动要达成什么，或指向 ROADMAP / spec / .planning 的路径]

加载 SKILL（绝对路径）：<你的 skills 目录>/code-audit/SKILL.md
严格按 SKILL.md 执行；只读，不要修改任何被审计文件。
```

报告格式与严重度词表**由调用方给**（你的任务卡怎么写，它就怎么交）；没给时用 SKILL 的默认四节（结论先行 → 发现 → 未覆盖 → 待确认）与阻塞/主要/次要分级。

### 预期输出

一份 Markdown 报告 + 每条阻塞/主要缺陷的可重跑验证方式。

**成功判据**：每条问题能落到 `文件:行号`；每条阻塞/主要缺陷带修复建议 + 可验证的测试断言或可重跑命令；阻塞/主要/次要三级分开归类；未覆盖部分显式声明覆盖率与原因。

> ❌ 不合格："这段代码可能有问题，建议检查一下折扣计算逻辑。"
> ✅ 合格：`order/service.go:128` `applyDiscount` 对 `coupon.Amount` 未做负数校验，上游 `handler.go:44` 直接透传用户输入，可构造负数优惠券使订单金额为负。修复：L128 前增加负值校验并返回 `ErrInvalidCoupon`。验证：`pytest tests/test_order.py::test_negative_coupon`。

## 覆盖什么

七个审计方向：调用链与数据流 / 代码逻辑与安全 / 无效与冗余 / 重复代码 / 设计与架构 / 需求覆盖 / 测试可验证性。精力按风险分布分配，不均摊。

判定分两条轴：**后果 × 可逆性**（阻塞/主要/次要）与**误导性**（是否会让人读错，构成"地基债"）。完整判据见 [`./SKILL.md`](./SKILL.md)「定级」。

## 输入 / 输出契约

| 输入项 | 必填 | 说明 |
| --- | --- | --- |
| 审计范围 | × | 文件 / 目录 / PR / commit 区间；未指定时优先覆盖变更文件及其直接依赖 |
| 业务目标 | × | ROADMAP / spec / `.planning/` 路径；缺失则反推并标注置信度 |
| 深度与预算 | × | 调用链默认上下各一层；高风险路径（支付/认证/并发/公共接口/迁移脚本）追溯至边界 |
| 输出格式 | × | Markdown 报告；格式与词表由调用方约定 |

## 触发与不触发

| ✅ 触发 | ❌ 不触发 |
| --- | --- |
| "帮我 review 这个 PR" / "做次代码审计" | 单文件语法检查 |
| "这次改动有风险吗？能不能合？" | 格式化 / 代码风格建议 |
| "检查代码质量 / 技术债务"、"验证改动是否完整、有没有漏掉调用方" | 需要 AI 直接修改代码的修复任务 |

## 反模式

| 反模式 | 为什么坏 | 怎么做 |
| --- | --- | --- |
| 凭模型记忆下"调用链/死代码"结论 | 未验证的猜测就是误报来源 | 用文件读取与检索实际验证后再定性 |
| 为让报告好看制造边缘问题 | 每条误报都在教调用方忽略你所有结论 | 证据不足就弃权，列入待确认项 |
| 把风格偏好标成阻塞项 | 稀释阻塞项注意力 | 降级为次要，单独归类 |
| 审计中顺手改代码 | 违反只读身份定义 | 只描述与建议，修复交给调用方 |
| 规范与代码冲突时替调用方拍板 | 越权裁决可能压掉真实约束 | 呈现冲突事实，列入待确认项 |
| 大 diff 上均摊精力求"覆盖全面" | 高风险区反而审计不足 | 先分级，高风险区深潜 + 声明覆盖率 |
| 测试通过就当"符合需求" | 测试可能与被测代码出自同一误解 | 检查断言本身是否锁定了错误预期 |

## 目录结构

```
code-audit/
├── SKILL.md                 # 唯一入口：授权、红线、工作循环、三项对位、维度、定级、报告
└── references/
    └── audit-dimensions.md  # 7 维度展开：检查要点、无效代码分类、重复判据、OWASP 安全附录
```

- [`./SKILL.md`](./SKILL.md) — 触发条件写在 frontmatter 的 `description` 里，由 agent 自动匹配。
- [`./references/audit-dimensions.md`](./references/audit-dimensions.md) — 审到第三个函数、或要下"安全/重复"类定性时必须打开（SKILL 里有硬触发）。

## 许可证

Apache-2.0，详见 [`../LICENSE`](../LICENSE)。

*Last updated: 2026-10-29*
