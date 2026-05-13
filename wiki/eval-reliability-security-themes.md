---
tags: [分析, ai, eval, reliability, security, sandbox]
sources:
  - wiki/overview.md
  - wiki/ai/demystifying-evals-for-ai-agents.md
  - wiki/ai/evaluation-agent-unified-automation.md
  - wiki/ai/production-prompt-ab-evaluation-engineering-practice.md
  - wiki/ai/quantifying-infrastructure-noise-in-agentic-coding-evals.md
  - wiki/ai/eval-awareness-in-claude-opus-4-6-browsecomp-performance.md
  - wiki/ai/opensandbox-agent-sandbox.md
  - wiki/ai/making-claude-code-more-secure-and-autonomous-with-sandboxing.md
  - wiki/ai/claude-code-auto-mode-a-safer-way-to-skip-permissions.md
  - wiki/ai/a-postmortem-of-three-recent-issues.md
updated: 2026-05-12
---

# Eval、可靠性与安全治理主题综述

## 任务说明

这篇文档聚焦当前知识库里另一条很强但容易被忽视的主线：`大模型应用从 demo 走向 production，真正的门槛不在模型，而在 eval、可靠性、sandbox 和治理。`

它试图回答：

- 为什么这个知识库里关于评测、事故、沙箱和权限的内容会越来越多
- 为什么这些内容不是附属话题，而是 Agent 应用的基础设施
- 从应用开发角度看，什么才算“生产级 LLM 系统”

## 一、这个知识库对“生产化”的定义非常清楚

如果把整套知识库压缩成一句关于生产化的判断，那就是：

`能跑通不算本事，能稳定比较、持续验证、控制风险，才算系统。`

这也是为什么知识库后期的高密度内容都在向这些主题集中：

- eval
- benchmark
- badcase
- A/B
- sandbox
- permissions
- observability
- postmortem
- infra noise

这些东西看起来不像“模型能力”，但它们恰好决定了系统是否能长期演进。

## 二、为什么说 `Eval` 是第一类基础设施

[[wiki/ai/demystifying-evals-for-ai-agents|Demystifying evals for AI agents]] 提供了这一主题最稳的起点。它把 eval 从“跑几个 benchmark 分数”重新拆回了最朴素的结构：

- 任务
- grader
- capability eval
- regression eval

它的潜台词非常重要：  
如果一个系统没有评测集、没有 grader、没有回归机制，那它并不是在迭代，而是在“靠感觉尝试”。

[[wiki/ai/evaluation-agent-unified-automation|评测 Agent 的统一自动化架构]] 又更进一步，说明评测在成熟系统里甚至会长成独立 Agent 或统一 workflow。因为当业务场景和样本规模膨胀后，评测本身也会从一堆脚本变成一个需要编排、抽样、验收和 badcase 分析的系统。

这说明在 Agent 时代，eval 不再是发布前的最后一道工序，而是持续运行的治理层。

## 三、为什么 benchmark 分数不等于系统质量

知识库里和 eval 相关的页面，并没有把 benchmark 当神谕。恰恰相反，它们反复在提醒 benchmark 的局限。

[[wiki/ai/quantifying-infrastructure-noise-in-agentic-coding-evals|Quantifying infrastructure noise in agentic coding evals]] 指出，agentic coding eval 的波动经常来自基础设施噪声，而不是模型能力本身。  
这意味着当你看到分数上升或下降时，先要问：

- 环境是否一致
- 工具链是否一致
- 依赖是否一致
- 网络、缓存、并发是否引入噪声

[[wiki/ai/eval-awareness-in-claude-opus-4-6-browsecomp-performance|Eval awareness in Claude Opus 4.6's BrowseComp performance]] 又从污染角度提醒：当模型更强、agent 更复杂、检索更广之后，评测不只是会被训练集污染，还会被环境信号和多 agent 过程中的间接泄漏污染。

也就是说，benchmark 是必要的，但从来不充分。生产系统更需要的是：

- 自己的 regression set
- 自己的 badcase 集
- 与业务目标对齐的 graders

## 四、坏例子为什么比好例子更值钱

[[wiki/ai/production-prompt-ab-evaluation-engineering-practice|生产级 Prompt 自动化推理评估]] 这篇文章很有代表性，因为它没有把“让模型帮忙看 A/B 结果”写成天才创意，而是把重点放在 badcase 的复盘与反向修正上。

这里最成熟的认识是：

- 好样本决定系统能不能跑通
- 坏样本决定系统能不能真的变稳

很多 LLM/Agent 团队的问题不是没有模型、没有工具，而是 badcase 没有产品化地回流到评测集、prompt、工具设计和规则系统里。结果就是系统每周都在犯相似的错误。

所以从应用开发视角看，badcase 应该被当作一类资产管理，而不是 bug ticket 的副产物。

## 五、为什么安全问题在 Agent 系统里从“内容风险”升级成“动作风险”

对传统聊天系统来说，安全往往集中在输出内容是否违规、是否误导。  
对 agent 系统来说，风险重心明显变了。因为一旦系统能：

- 调工具
- 写文件
- 执行命令
- 访问外部系统
- 操作浏览器

那么真正危险的就不是“说错”，而是“做错”。

这也是为什么 [[wiki/ai/opensandbox-agent-sandbox|OpenSandbox：面向 AI Agent 的通用沙箱平台]]、[[wiki/ai/making-claude-code-more-secure-and-autonomous-with-sandboxing|Making Claude Code more secure and autonomous with sandboxing]]、[[wiki/ai/claude-code-auto-mode-a-safer-way-to-skip-permissions|Claude Code auto mode]] 这些页面在知识库里非常重要。它们共同说明：

- 自主性越强
- 风险半径越大
- 因而越需要更强的制度化控制

这不是保守，而是要让系统有资格获得更高自主性。

## 六、Sandbox 不是附属品，而是 Agent 自主性的前提

知识库对 sandbox 的看法非常一致：它不是“上线前模拟一下”的工具，而是很多 agent 系统真正的主执行面。

[[wiki/ai/opensandbox-agent-sandbox|OpenSandbox]] 从平台角度讲得最全。它把沙箱定义成一层面向 AI 应用的通用执行环境，用来承接：

- 代码执行
- 文件读写
- 浏览器自动化
- 远程运行时

它真正的工程价值在于：

- 控制文件系统范围
- 控制网络权限
- 控制资源消耗
- 控制副作用半径

[[wiki/ai/making-claude-code-more-secure-and-autonomous-with-sandboxing|Claude Code 沙箱化]] 则把这个逻辑压成一句非常强的结论：  
`更自主，不是因为限制更少，而是因为环境更可控。`

这其实是一种成熟系统观。系统之所以敢放权，不是因为相信模型不会犯错，而是因为就算犯错，执行环境也先把损失限制住了。

## 七、权限治理真正解决的是“谁能在什么条件下做什么”

[[wiki/ai/claude-code-auto-mode-a-safer-way-to-skip-permissions|Claude Code auto mode]] 对这件事的说明很有代表性。它不是简单粗暴地“跳过权限”，而是通过分类器、固定模板、风险分级和可配置槽位，去决定哪些动作能自动过、哪些动作必须确认。

这种设计说明一个更深的原则：

`Agent 权限治理不是问“要不要放开”，而是问“什么条件下可以放开到什么程度”。`

从工程上看，一套成熟的权限系统至少应包含：

- 最小权限原则
- 高风险操作人工确认
- 工具级和资源级授权
- 可回放审计
- 与 sandbox 协同的执行边界

## 八、事故复盘告诉我们：问题可能根本不在模型

[[wiki/ai/a-postmortem-of-three-recent-issues|A postmortem of three recent issues]] 是这条主线上非常关键的一页，因为它强迫人把注意力从“模型能力”移到“系统复杂性”。

它展示的不是简单 bug，而是多层失效：

- 上下文窗口路由问题
- 输出链路损坏
- XLA/TPU 编译和硬件栈问题

这意味着一个非常重要的现实：

当用户感知到“模型变差了”，根因不一定在模型本身，也可能在：

- serving
- routing
- tool layer
- runtime
- infra

所以做大模型应用，必须接受一个事实：  
你在运营的不是“一个模型接口”，而是一条多层供应链。

## 九、把这条主题压缩成生产级 LLM 系统的最小定义

基于全库这条主线，我会把“生产级 LLM / Agent 系统”定义成同时具备下面这些能力的系统：

- 有自己的评测集和回归机制
- 有 badcase 回流机制
- 有工具和模型变更后的比较能力
- 有日志、trace、cost、latency 观测
- 有 sandbox 和权限分级
- 有失败恢复与停止条件
- 有对基础设施噪声和环境污染的认知

如果缺了其中大半，它更像一个 demo，而不是系统。

## 十、这条主线对应用开发岗最有价值的表达

如果面试官问“你怎么理解 production-ready 的大模型应用”，一个成熟答法不该只说“接监控、加缓存”。更完整的说法应该是：

`我理解的大模型应用生产化，核心在于把模型输出的不确定性收敛到可比较、可验证、可审计、可隔离的系统里。具体来说，一边要有 eval、regression、badcase 和 observability 这些质量治理设施，一边要有 sandbox、权限分级、执行边界和事故复盘能力。没有这些层，系统只能偶尔跑通，无法稳定迭代。`

## 关联页面

- [[wiki/ai/demystifying-evals-for-ai-agents|Demystifying evals for AI agents]]
- [[wiki/ai/evaluation-agent-unified-automation|评测 Agent 的统一自动化架构]]
- [[wiki/ai/production-prompt-ab-evaluation-engineering-practice|生产级 Prompt 自动化推理评估]]
- [[wiki/ai/quantifying-infrastructure-noise-in-agentic-coding-evals|Quantifying infrastructure noise in agentic coding evals]]
- [[wiki/ai/eval-awareness-in-claude-opus-4-6-browsecomp-performance|Eval awareness in Claude Opus 4.6's BrowseComp performance]]
- [[wiki/ai/opensandbox-agent-sandbox|OpenSandbox：面向 AI Agent 的通用沙箱平台]]
- [[wiki/ai/making-claude-code-more-secure-and-autonomous-with-sandboxing|Making Claude Code more secure and autonomous with sandboxing]]
- [[wiki/ai/claude-code-auto-mode-a-safer-way-to-skip-permissions|Claude Code auto mode: a safer way to skip permissions]]
- [[wiki/ai/a-postmortem-of-three-recent-issues|A postmortem of three recent issues]]
