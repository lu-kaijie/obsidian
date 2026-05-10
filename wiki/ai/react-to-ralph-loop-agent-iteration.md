---
tags: [摘要, ai, agent, coding, loop, harness]
sources: [raw/ai/2026-05-09-react-to-ralph-loop-agent-iteration-webclip.md]
updated: 2026-05-09
---

# 从 ReAct 到 Ralph Loop

**来源：** [丹坤, 2026-01-27](https://mp.weixin.qq.com/s/K4ZUGBzT0s9RwFlaYcuHiA)

## 核心结论

这篇文章最有价值的地方，是把很多人模糊感受到的一个现象说清楚了：AI 编程助手经常不是“不会做”，而是“过早停”。`Ralph Loop` 的意义不在于让模型更聪明，而在于把“是否继续工作”的决定权，从模型主观判断手里拿走，交给外部可验证条件和循环控制器。

如果说 `ReAct` 是 agent 在单次会话里做“观察 -> 推理 -> 行动”的内部循环，那么 `Ralph Loop` 更像一个外层调度器：同一个任务反复运行，直到测试、结构检查或显式完成承诺真正满足为止。

## 文章提出的问题很具体

作者总结的几类痛点都很真实：

- 模型在自认为“差不多”时提前退出
- 复杂任务难以靠单轮提示完成
- 每次人工重新提示都在重复支付 orchestration 成本
- 会话重启后，历史上下文和进展容易断裂

它的核心诊断是：`LLM` 的自我完成判断不可靠，所以不能把“何时停下”完全交给模型。

## Ralph Loop 的关键机制

文章把 `Ralph Loop` 讲成一个非常简洁的模式：

- 任务提示反复注入
- 每轮都读取文件系统、测试输出和 Git 历史
- 如果没有满足完成条件，就通过 `stop hook` 拦截退出并继续下一轮
- 用 `max-iterations` 作为安全阀，避免无限循环

这里真正重要的不是 shell while 循环本身，而是它背后的控制逻辑：

- 完成标准必须外部可验证
- 状态要落到文件系统，而不是只留在会话 token 里
- 退出条件要受外部程序控制，而不是只靠模型自述“我做完了”

这使它更接近一种 `refine-until-done` 的工程控制层，而不是传统意义上的“更强 agent 架构”。

## 它和 ReAct / Plan-and-Execute 的差异

文章做的最好的一点，是没有把 `Ralph Loop` 说成通用替代品，而是明确它和常见 agent loop 的定位不同。

对比可以简化成：

- `ReAct`：适合在单会话里根据观察动态调整下一步
- `Plan-and-Execute`：适合先拆子任务再顺序执行
- `Ralph Loop`：适合一个可验证目标上的持续修正，重点是“没达标就别停”

所以 `Ralph` 不是在优化内部推理链，而是在外部加了一层强制完成机制。这一点和 [[wiki/ai/spec-driven-development-redefining-ai-coding|Spec-Driven Development 重新定义 AI 编程]] 很能对上：前者约束“什么时候算完成”，后者约束“做成什么样才算对”。

## 状态持久化是这篇的第二个重点

作者强调 `Ralph Loop` 能跨长周期任务工作的原因，在于它把状态从上下文窗口搬到了外部环境：

- `progress.txt` 记录尝试过程和 learnings
- `prd.json` 记录结构化任务状态
- Git 提交提供差分、回滚点和客观变更历史

这其实是在把“记忆”从模型内存迁移到仓库与文件系统。这个思路和 [[wiki/ai/harness-engineering-codex-agent-first|在智能体优先的世界中利用 Codex]] 中“仓库即记录系统”的判断高度一致，只不过这里更进一步，把仓库当成循环恢复点和进度账本。

## 文章的实用价值

这篇最大的实践价值，不是教你写一个 while 循环，而是提醒了三件事：

- 长任务要有显式 completion criteria
- completion criteria 最好由测试、结构检查或标记信号验证
- agent 的“持久工作能力”更多是 orchestration 问题，不只是模型能力问题

这也和 [[wiki/ai/ai-code-review-aacr-bench|AI 代码评审实践与 AACR-Bench]] 形成互补：那篇关注生成后的质量治理，这篇关注生成过程如何避免半途而废。

## 我的判断与保留

我认为这篇对 `Ralph Loop` 的解释是成立的，但它也有边界：

- 它更适合有清晰完成标准的工程任务
- 对探索性、开放式、很难客观验证的任务，循环未必能自然收敛
- 如果完成条件本身写得差，`Ralph` 只会更顽固地朝错误目标迭代

换句话说，`Ralph Loop` 解决的是“别太早停”，不是“自动知道该往哪里走”。它需要和清晰规格、测试、规则文件或任务清单配套使用。

## 阅读评级

🟡 值得追问 — 如果你关心的不是单轮 agent demo，而是“怎样让 AI 在工程任务上持续工作直到真正交付”，这篇很值得保留。它最有启发性的地方，是把持续性问题从模型推理问题改写成了外部控制与状态管理问题

## 与其他页面的关联

- [[wiki/ai/harness-engineering-codex-agent-first|在智能体优先的世界中利用 Codex]] — 两者都强调仓库和外部约束比“再写一条更好的 prompt”更关键
- [[wiki/ai/spec-driven-development-redefining-ai-coding|Spec-Driven Development 重新定义 AI 编程]] — `SDD` 负责把目标和验收讲清，`Ralph` 负责没达标前继续推进
- [[wiki/ai/ai-fullstack-dev-practice-harness-sdd-multi-repo|Harness + SDD + 多仓管理：AI 全栈开发实践]] — 那篇讨论约束与并行协作，这篇补的是持续执行控制
- [[wiki/ai/ai-code-review-aacr-bench|AI 代码评审实践与 AACR-Bench]] — 前者偏运行中控制，后者偏运行后治理，合起来更接近完整闭环
