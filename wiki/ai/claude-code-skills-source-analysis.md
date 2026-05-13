---
tags: [摘要, ai, claude-code, skills, context, runtime]
sources: [raw/ai/2026-05-13-claude-code-skills-source-analysis-webclip.md]
updated: 2026-05-13
---

# Claude Code 的 skills 源码解析

**来源：** [王君生, 2026-04-08](https://juejin.cn/post/7625838952655912994?share_token=d88b9cf0-a5d8-407e-ac1d-d26859c143b3)

## 核心结论

这篇文章最值得保留的，不是再解释一遍 `skill` 是什么，而是把 Claude Code 里 `skill 从磁盘文件到运行时能力` 的链路拆开了：

- 启动阶段先注册内置 skill，保证能力面先于命令系统装配
- 磁盘 skill 通过多目录并行扫描、frontmatter 解析和 realpath 去重进入命令表
- `SKILL.md` 不是静态文档，而是在调用时被编译成 prompt blocks 注入上下文
- `paths`、`allowed-tools`、`model` 这类 frontmatter 字段，本质上是 runtime 级制度，不只是文档注释

## 它真正解释的是 skill 为什么会成为工程构件

文中沿着 `prompt -> function calling -> MCP -> skills` 的历史线索推进，但真正有价值的落点不是概念演化，而是一个判断：

`skill` 的本质不是“更长的提示词”，而是一个可被发现、可被维护、可被延迟加载的能力包。

这和 [[wiki/ai/agentscope-skills-progressive-disclosure|AgentScope Skills 与渐进式披露]]、[[wiki/ai/equipping-agents-for-the-real-world-with-agent-skills|让 Agent 用上 Skills]] 是同一条线，只是这篇更靠近 Claude Code 的具体实现。

## 文章里最值得记的是加载链路

作者把 Claude Code 的 skill 加载链路拆得比较清楚：

- `initBundledSkills()` 先把内置 skill 注册进内存
- `getSkillDirCommands()` 再从托管目录、用户目录、项目目录和附加目录并行扫描磁盘 skill
- 每个 skill 目录通过 `SKILL.md` frontmatter 解析出描述、模型、权限、上下文等结构化字段
- 最后用 `realpath` 去重，并把条件 skill 放进待激活集合

这个拆法说明，skill 不是附属功能，而是和命令、插件、workflow 并列的 runtime 能力源。

## Conditional Skills 是它最值得追问的一点

带 `paths` 的 skill 不会全量常驻，而是在文件路径命中时才激活。这等于把渐进式披露继续向前推进了一层：

- 不是所有能力都先暴露给模型
- 也不是所有仓库规则都写死进常驻上下文
- 而是让运行时依据当前文件范围动态装配局部制度

这和 [[wiki/ai/claude-code-prompt-context-harness-practice|Claude Code 的 Prompt / Context / Harness 设计实践]] 里的“上下文按任务阶段组装”可以互证。

## 这篇对 frontmatter 的处理也很有价值

文中列出的 `model`、`effort`、`context`、`agent`、`allowed-tools`、`user-invocable` 等字段，说明 skill 文件正在从“说明文档”进化成“声明式能力描述”。

也就是说，`SKILL.md` 同时承载三层东西：

- 给模型看的 SOP
- 给运行时看的控制参数
- 给人类维护者看的接口契约

这让它比普通 prompt 模板更接近一种 repo 内 DSL。

## 我的判断与保留

我认为这篇最值得留下的三个点是：

- skill 的价值在于把能力做成可扫描、可去重、可延迟注入的仓库资产
- Claude Code 并没有把 skill 当“文本外挂”，而是放进了命令系统和 runtime 装配链
- 条件激活和 frontmatter 声明，说明 skill 正在从知识片段走向制度化能力接口

它的局限是：

- 属于二手源码解读，具体模块命名和实现细节不应机械照搬
- 前半部分历史脉络较长，但真正高价值内容集中在加载与注入实现

## 阅读评级

🟡 值得追问 — 如果你已经接受 `skill` 有用，但还想进一步理解它为什么会成为 coding agent runtime 的一级构件，这篇值得保留

## 与其他页面的关联

- [[wiki/ai/agentscope-skills-progressive-disclosure|AgentScope Skills 与渐进式披露]] — 那篇讲能力如何分层暴露，这篇补 Claude Code 里能力包如何被扫描和激活
- [[wiki/ai/equipping-agents-for-the-real-world-with-agent-skills|让 Agent 用上 Skills]] — Anthropic 官方页更偏原则，这篇更偏本地 skill runtime 的装配细节
- [[wiki/ai/claude-code-source-runtime-multi-agent|Claude Code 源码拆解：从启动到多 Agent 扩展层]] — 一篇看总体 runtime 骨架，这篇补 skills 子系统
- [[wiki/ai/function-calling-mcp-skills-differences-practice|Function Calling、MCP 与 Skills 的边界]] — 那篇解释分层概念，这篇说明 skill 这一层在工程上如何落地
