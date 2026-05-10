---
tags: [摘要, ai, agent, coding, claude, vibe-coding]
sources: [raw/ai/2026-05-09-mini-claude-vibe-coding-rebuild-webclip.md]
updated: 2026-05-09
---

# 赛博鸡生蛋，7小时用Claude Vibe Coding一个Mini-Claude

**来源：** [诚昊, 2026-04-17](https://mp.weixin.qq.com/s/IUjdKUcPKZrBE7GZZnQG-w)

## 核心结论

这篇文章的价值不在“7 小时复刻了个 Mini-Claude”这种标题党，而在它给出了一个 `极简 coding agent` 的拆解顺序：

- 先打通 LLM API
- 再补工具层
- 再做 session / message manager
- 再做 tool loop
- 最后接 CLI 和日志 / dashboard

如果你想快速验证 coding agent 最小闭环，这篇是合格的入门实战。

## 它把 Coding Agent 拆回了最朴素的几个部件

文章实际上是在用一个小项目解释：所谓 Claude CLI 这类工具，并不神秘，最小内核通常就是：

- 请求格式封装
- 工具描述与执行
- 历史消息拼接
- tool call 循环
- 会话存档
- 终端交互壳

这个拆法和 [[wiki/ai/claude-code-source-runtime-multi-agent|Claude Code 源码拆解：运行时与多 Agent 扩展层]] 恰好构成高低配对照：那篇讲成熟 runtime 的复杂制度，这篇讲最小可运行原型需要哪些零件。

## 对新手最有帮助的是“构建顺序”，不是具体代码

文章最实用的部分，是它没有一上来堆一大坨工程，而是按依赖顺序推进：

- API 先通
- tool use 再通
- session manager 负责消息与循环
- CLI 只是外壳
- dashboard 是后续观测层

这个顺序很合理，因为它先验证 agent loop，再去美化交互。很多人做这种项目的问题恰好相反：先做 UI，最后才发现 loop 本身没闭合。

## 它本质上仍然是 Vibe Coding 原型法，不是稳定工程法

文章一再强调让 Claude 边写边修、根据 API 报错持续试错，这很符合 `Vibe Coding` 的原型推进方式。优点是快，缺点也明显：

- 依赖模型临场修补
- 对设计约束和测试的强调有限
- 更像“先跑起来”，不是“先定义清楚”

所以它和 [[wiki/ai/ai-coding-practice-vibe-coding-to-sdd|AI 编码实践：从 Vibe Coding 到 SDD]]、[[wiki/ai/spec-driven-development-redefining-ai-coding|Spec-Driven Development 重新定义 AI 编程]] 的关系很明确：这篇适合做探索和教学样本，不适合被误读成复杂生产系统的完整方法论。

## Session 日志与 dashboard 的加入，说明作者已经摸到“观测层”了

虽然整体项目很轻，但文章把 session 文件和 dashboard 单独拿出来，是个好信号。因为一旦 agent 开始调用工具、循环调试、维护会话，光有结果还不够，还需要：

- 知道每轮发了什么
- 工具调用如何回灌
- 哪一轮开始偏掉
- 哪些错误是协议层而不是业务层

这说明作者已经开始从“能跑”往“可排查”过渡。

## 我的判断与保留

我认为这篇最值得保留的是三个点：

- 给出了一个适合教学的 coding agent 最小闭环搭建顺序
- 清楚展示了 tool loop 和 session manager 为什么是核心
- 说明 Vibe Coding 在探索期很有效，但天然不是高约束工程方法

它的局限也很明显：

- 更偏新手向复刻，不是体系化架构设计
- 强依赖特定模型 / 接口细节，泛化性一般
- 对权限、安全、恢复、长期维护这些成熟 runtime 议题涉及较少

## 阅读评级

🟡 值得追问 — 如果你想快速理解“一个最小 coding agent 到底由哪些部件构成”，这篇很适合；如果你要的是生产级 runtime 设计，它只够做热身

## 与其他页面的关联

- [[wiki/ai/claude-code-source-runtime-multi-agent|Claude Code 源码拆解：运行时与多 Agent 扩展层]] — 一篇看成熟系统，一篇看极简复刻
- [[wiki/ai/claude-code-qoder-cli-advanced-practice|Claude Code 与 Qoder CLI 的工作流抽象]] — 那篇给组件地图，这篇给一个轻量实现路径
- [[wiki/ai/ai-coding-practice-vibe-coding-to-sdd|AI 编码实践：从 Vibe Coding 到 SDD]] — 这篇更适合放在 Vibe Coding 的探索阶段
- [[wiki/ai/spec-driven-development-redefining-ai-coding|Spec-Driven Development 重新定义 AI 编程]] — 那篇能作为这篇原型法的反面补全
