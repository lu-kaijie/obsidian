---
tags: [摘要, ai, coding, sdd, spec, harness]
sources: [raw/ai/2026-05-09-spec-driven-development-redefining-ai-coding-webclip.md]
updated: 2026-05-09
---

# Spec-Driven Development 重新定义 AI 编程

**来源：** [王砚舒（彦纾）, 2026-05-09](https://mp.weixin.qq.com/s/hVizUucsy8rwFOUR-VZ6wA)

## 核心结论

这篇文章的价值不只是再讲一遍 `SDD`，而是把它明确放在 `vibe coding -> context engineering -> harness engineering` 的演进链里：`SDD` 不是单独的文档写作习惯，而是给 AI 编码建立可共享约束、可增量迭代规格和可验证验收标准的一层控制面。

文章最强的论点是：当代码可以被 AI 快速重写时，真正稀缺的资产不再是代码本身，而是“为什么这么做、边界在哪里、做到什么算完成”这套可复用决策。`Spec` 因而变成了代码之上的压缩表示。

## 为什么作者认为 SDD 是结构性需求

文章以 QoderWork 的案例开场：5 人在 7 天内完成过去约 20 人周的工作量，关键不是“AI 写得快”，而是团队在 `DAY 0` 先把 `MVP` 边界、模块拆解和各模块 `Spec` 写清楚，再让 `Quest` 并行执行。

作者的判断很明确：

- 没有规格时，AI 只能靠局部上下文猜需求
- 规格写得差，在传统开发里主要伤沟通效率
- 规格写得差，在 AI 编码里会直接变成质量问题和返工

所以 `SDD` 的真正作用不是提高文档完整度，而是减少模型的猜测空间。

## SDD 的标准结构

文章把 `SDD` 拆成一套很完整的四阶段与四层文档体系：

- `Specify`：定义问题、成功标准、边界和非目标
- `Plan`：基于规格生成架构、模块和接口方案
- `Implement`：AI 按任务清单生成代码和测试
- `Validate`：自动化测试加人工 review

在文档层面，它强调：

- `spec.md` 解决“做什么”和“为什么做”
- `plan.md` 把规格转成技术实现方案
- `tasks.md` 把实现拆成可并行、可验证的原子任务
- `constitution.md` 固化项目级不可违背原则

这和 [[wiki/ai/harness-engineering-codex-agent-first|在智能体优先的世界中利用 Codex]] 中“短 AGENTS + 仓库内真实约束”的思路一致，只是这里把约束层分得更显式。

## 文章最有价值的判断

### 1. 好 Spec 的标准是“可测试”，不是“看起来完整”

作者反复强调，`Spec` 的好坏分水岭在于：

- 成功标准是否可量化
- `Non-Goals` 是否真的把边界排除掉
- `Constraints` 是否限制了外部条件，而不是偷写实现细节

一个很好的检验问题是：换一个技术栈，这份 `Spec` 是否仍成立。如果不成立，说明你把 `HOW` 混进了 `WHAT`。

### 2. 它正面回应了 “SDD 就是 Markdown 版瀑布” 的批评

这篇不是盲目吹捧。它承认 `SDD` 有被做成瀑布模型的风险，但指出关键差异在于 `Spec` 是否保持活性。

作者给出的可行解释是：

- 瀑布的问题在于需求冻结太久
- `SDD` 的正确实践是把 `Spec` 当作增量更新的活文档
- 新需求和 bug 都应该先改 `Spec`，再驱动实现

也就是说，`SDD` 的迭代粒度不是“整个项目”，而是“单个规格单元”。

### 3. 它比很多方法论文章更诚实地写了失败面

文中列出的五个陷阱很有参考价值：

- 过度规格化，把 `Spec` 写成伪代码
- 规格腐烂，代码和 `Spec` 漂移
- 规格官僚化，任何小修改都强制走全流程
- 虚假信心，以为有 `Spec` 就可以降低 review 强度
- 工具复杂性过高，花太多时间折腾工具链

这让它比纯宣传式 `SDD` 文章更有实操意义。

## 和 Vibe Coding 的关系

文章对 `vibe coding` 的处理比较成熟，不是简单否定，而是把它看作一个更适合探索期的小项目策略。

作者的结论是：

- 探索阶段可以先用 `vibe coding` 快速试错
- 一旦要长期维护，就应该尽快补 `Spec`
- 生产开发阶段，变更应由 `Spec` 驱动，而不是继续靠“感觉”

这个判断和 [[wiki/ai/ai-coding-practice-vibe-coding-to-sdd|AI 编码实践：从 Vibe Coding 到 SDD]] 是互补的。后者更像团队演进复盘，这篇则更像为那条演进路径提供理论骨架。

## 和现有页面的关系

这篇最适合放在当前知识库里作为 `SDD` 主题的中心页，因为它补上了几层之前分散在不同文章中的关系：

- 它为 [[wiki/ai/ai-fullstack-dev-practice-harness-sdd-multi-repo|Harness + SDD + 多仓管理：AI 全栈开发实践]] 提供了更完整的 `SDD` 理论解释，不再只停留在 workflow 经验
- 它把 [[wiki/ai/ai-coding-practice-vibe-coding-to-sdd|从 Vibe Coding 到 SDD]] 中的混合策略，提升成“探索期 vs 生产期”的阶段划分
- 它也能和 [[wiki/ai/ai-engineering-vs-traditional-engineering|AI 工程 vs 传统工程：道法术中的变与不变]] 对上：`SDD` 本质上是在“法”和“术”之间，给不确定模型加确定性结构

## 我的保留判断

这篇文章的问题在于，它给出的很多数据和外部引用更像方法论动员材料，而不是严格证据综述。比如安全漏洞率、错误率下降和代码重复率这些数字，对方向判断有帮助，但如果后续要把它们当成决策依据，最好单独补原始论文或报告来源。

另外，QoderWork 的“5 人 7 天”案例很强，但更像高约束、高投入、强工具链条件下的上限展示，不应直接外推到一般团队。

## 阅读评级

🟡 值得追问 — 这篇适合作为 `SDD` 的方法总览页来读。它真正有价值的地方不是案例炫技，而是把 `Spec` 为什么会在 AI 编码时代从“可选文档”升级成“执行控制层”讲清楚了

## 与其他页面的关联

- [[wiki/ai/ai-coding-practice-vibe-coding-to-sdd|AI 编码实践：从 Vibe Coding 到 SDD]] — 提供团队从补全、rules 到 `SDD` 的现实演进路径，这篇补的是理论总图
- [[wiki/ai/ai-fullstack-dev-practice-harness-sdd-multi-repo|Harness + SDD + 多仓管理：AI 全栈开发实践]] — 那篇偏多仓和并行协作，这篇偏 `Spec` 结构与流程设计
- [[wiki/ai/harness-engineering-codex-agent-first|在智能体优先的世界中利用 Codex]] — 都在讨论如何给 agent 施加仓库级约束，只是这篇更突出规格层
- [[wiki/ai/ai-engineering-vs-traditional-engineering|AI 工程 vs 传统工程：道法术中的变与不变]] — 可以把 `SDD` 看作用工程确定性收敛模型不确定性的具体机制
