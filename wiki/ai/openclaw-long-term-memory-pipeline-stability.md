---
tags: [摘要, ai, agent, openclaw, memory, architecture]
sources: [raw/ai/2026-05-09-openclaw-long-term-memory-pipeline-stability-webclip.md]
updated: 2026-05-09
---

# OpenClaw长期记忆：优秀管线与玄学效果

**来源：** [城决, 2026-04-15](https://mp.weixin.qq.com/s/pLqKTe1J2FkiiquoT4dOJA)

## 核心结论

这篇文章的判断很清楚：OpenClaw 的记忆体系 `设计上很优雅`，但 `实际效果并不天然稳定`。它最值得保留的，不是“记忆能写进 Markdown”，而是把整条记忆管线拆开后，你能看见哪些环节是确定性的，哪些环节仍然在赌 LLM 的临场发挥。

## OpenClaw 的优势在于“文件化分层”，不是神奇记忆本身

文章把记忆层拆成：

- `AGENTS.md`、`SOUL.md`、`USER.md` 这类身份和规则文件
- `memory/YYYY-MM-DD.md` 这类日记忆
- `MEMORY.md` 这类长期记忆
- `DREAMS.md` 这类后台梦境产物

这个分层和 [[wiki/ai/openclaw-architecture-implementation-part2|OpenClaw 技术架构（下）]]、[[wiki/ai/openclaw-practice-six-agents-mac|OpenClaw 实战：六个 AI Agent]] 能很好互证：OpenClaw 真正押注的是 `file-first persistence`，而不是黑箱 embedding magic。

## 两条写入路径都离不开 LLM 判断，这就是不稳定来源

文章指出日记忆写入主要有两条路：

- 对话中 agent 主动写入
- compaction 前的 memory flush 补写

这两条路径都依赖 LLM 决定：

- 记不记
- 记什么
- 如何表述

所以它的上限很高，下限也不稳。你会得到一个“理论上可成长”的记忆系统，但不一定得到一个“每次都能稳稳记住关键事实”的系统。

这和 [[wiki/ai/ai-agent-memory-systems-architecture-practice|AI Agent 记忆系统]] 提到的结论一致：记忆系统难点从来不只是存储，而是抽取、晋升、召回和治理。

## 默认 promote 机制太自由，工程上会出现“记忆玄学”

文章最关键的批评是，OpenClaw 默认从日记忆晋升到 `MEMORY.md` 的过程太依赖 agent 自主维护：

- 直接改长期记忆
- heartbeat 时定期整理
- 由 Dreaming 后台巩固

这意味着 promote 规则并不严格。对个人实验系统来说，这种自由度很好；对想要稳定复现的系统来说，这会带来几个问题：

- 同类信息今天可能晋升，明天可能不晋升
- 重要事实可能被写在 daily note 却没进入长期层
- 召回效果会受历史写法和表述漂移影响

它与 [[wiki/ai/memory-architecture-ledger-views-policy|Memory 架构与思考]] 形成了一个很好的对照：理论上应该把 raw ledger、derived views、policy 分开，而 OpenClaw 的默认实践在 policy 层仍然偏弱。

## Dreaming 系统很有启发，但并不等于“自动变稳”

文章对 Dreaming 的拆解很有价值：

- Light Sleep 做摄取和机械去重
- REM 做主题反射和候选真理
- Deep Sleep 做更深层 consolidate

这条链条说明 OpenClaw 已经不满足于“把记忆存起来”，而是在尝试做异步巩固和抽象。这个方向是对的，但作者也指出了核心问题：

- 去重未必理解语义，只能近似做字面处理
- 候选真理筛选仍然依赖启发式和 LLM
- 最终写入长期记忆并非强约束流程

所以 Dreaming 更像 `记忆研究原型`，不是已经完全工业化的稳定机制。

## RDSClaw 这类补强，本质是在给记忆管线加确定性

文章后半段引入插件式补强，我认为值得记住的不是具体插件名，而是补强思路：

- 用更明确的写入结构
- 用更稳定的召回策略
- 用额外索引或数据库降低纯文本漂移
- 用更强规则替代“想到就写”

也就是说，OpenClaw 原生记忆的价值更像 `优秀基底`，而不是最终答案。

## 我的判断与保留

我认为这篇最值得留下的是三个判断：

- file-first memory 是很好的可审计底座，但不自动等于高质量记忆
- 记忆系统稳定性取决于 promote / recall / policy，而不只是能不能写文件
- Dreaming 很有前瞻性，但在默认形态下更像探索性机制而非稳态生产方案

它的局限是：

- 对插件补强的收益引用较多，但通用性仍需谨慎看
- 对 “玄学效果” 的判断有经验色彩，缺少更系统的失效样本统计

## 阅读评级

🔴 建议深读 — 如果你在做 agent memory，这篇很适合拿来校正预期：好看的记忆架构图，不等于稳定可复现的记忆系统

## 与其他页面的关联

- [[wiki/ai/openclaw-architecture-implementation-part2|OpenClaw 技术架构（下）]] — 那篇讲 memory 模块长什么样，这篇讲它为什么会不稳定
- [[wiki/ai/openclaw-practice-six-agents-mac|OpenClaw 实战：六个 AI Agent]] — 实战页展示记忆 promote 的运营样本，这篇补背后的机制与风险
- [[wiki/ai/ai-agent-memory-systems-architecture-practice|AI Agent 记忆系统]] — 一篇更宏观，一篇更贴 OpenClaw 实装
- [[wiki/ai/memory-architecture-ledger-views-policy|Memory 架构与思考]] — 这篇可以作为 OpenClaw 记忆系统 policy 不足的具体案例
