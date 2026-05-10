---
tags: [摘要, ai, coding, agents-md, context, harness]
sources: [raw/ai/2026-05-10-agents-md-practice-guide-webclip.md]
updated: 2026-05-10
---

# 一个文件让 AI Coding 效率翻倍：AGENTS.md 实践指南

**来源：** [岛风, 2026-05-06](https://mp.weixin.qq.com/s/fBBBSfQajYjYtngZAitZCA)

## 核心结论

这篇文章最值得保留的不是 “AGENTS.md 是给 AI 看的 README” 这句口号，而是它把 `AGENTS.md` 的正确职责讲清楚了：`它应该是地图，不是手册`。

## 标准化背景讲得很清楚

文中把 `CLAUDE.md`、`.cursorrules`、`copilot-instructions`、`AGENTS.md` 的演进脉络梳理了一遍，适合作为上下文文件标准化的背景页。对当前知识库来说，它能和 [[wiki/ai/harness-engineering-codex-agent-first|在智能体优先的世界中利用 Codex]]、[[wiki/ai/qoder-harness-engineering-guide|Qoder Harness Engineering 指南]] 形成直接互证。

## “地图而非手册”是最重要的实践原则

文章明确主张：

- 只把项目全貌和硬约束写进 AGENTS.md
- 详细内容放进被引用文档
- AI 不知道就会直接写错的内容，才放在 AGENTS.md 里

这个判断和现有知识库里反复出现的“短 AGENTS、渐进式披露、仓库即记录系统”完全一致，但这篇更适合当专项实践页。

## 仓库聚合和私域源码引入也很有价值

它不只是写文件技巧，还讨论了为什么把前后端聚合到 monorepo、把私域组件源码放进参考项目，会显著改善 AI coding 效果。这说明 AGENTS.md 单独存在不够，它依赖一个 Agent 友好的仓库结构。

## 我的判断与保留

我认为这篇最值得留下的是三个点：

- `AGENTS.md` 的核心职责是导航，不是百科全书
- 规则文件要和仓库结构、源码可见性一起设计
- “AI 友好项目”是上下文文件、目录组织和验证脚本的组合，不是单文件魔法

## 阅读评级

🔴 建议深读 — 如果你已经接受要写 AGENTS.md，但还不清楚里面该放什么、不该放什么，这篇很值

## 与其他页面的关联

- [[wiki/ai/harness-engineering-codex-agent-first|在智能体优先的世界中利用 Codex]] — 那篇给原则，这篇给 AGENTS.md 专项实践
- [[wiki/ai/qoder-harness-engineering-guide|Qoder Harness Engineering 指南]] — 都强调短 AGENTS 和外链文档
- [[wiki/ai/claude-code-prompt-context-harness-practice|Claude Code 的 Prompt / Context / Harness 设计实践]] — 可对照 `CLAUDE.md` 与 `AGENTS.md` 两种项目上下文载体
