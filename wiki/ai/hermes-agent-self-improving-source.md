---
tags: [摘要, ai, agent, memory, skill, self-improving]
sources: [raw/ai/2026-05-10-hermes-agent-self-improving-source-webclip.md]
updated: 2026-05-10
---

# 深入源码：Hermes Agent 如何实现 Self-Improving

**来源：** [三剑, 2026-04-23](https://mp.weixin.qq.com/s/Qi68ptxQRyiA932JU49SYQ)

## 核心结论

这篇文章最值得保留的，是它把 `self-improving agent` 从口号拆成了一个可实现闭环：`短记忆 + Skill 自生长 + 触发/提醒机制`。相比 OpenClaw 那种更偏文件记忆与人工维护的体系，Hermes 更像在把“做过的事”自动沉淀成“以后会做的事”。

## Memory 和 Skill 的分工讲得很清楚

文中最有价值的一点，是把：

- Memory 讲成事实和偏好
- Skill 讲成程序性知识和操作手册

这和 [[wiki/ai/openclaw-long-term-memory-pipeline-stability|OpenClaw 长期记忆：优秀管线与玄学效果]] 正好形成对照：OpenClaw 更强调 file-first memory 与 promote，Hermes 更强调把踩坑经验升格为可复用 Skill。

## 容量上限 + 显式 replace/remove 是个好设计

Hermes 的记忆不是无限追加，而是强制容量受限，超限时要求模型自己合并、删除、替换。这一点很值得记，因为它把“压缩”变成了主动整理过程，而不是静默丢弃。

## 自生长 Skill 是这篇最关键的亮点

文章指出 Hermes 会在复杂任务、踩坑修复、用户纠正后，把经验提炼成 Skill，并且在后续使用中继续 patch。这个设计很强，因为它把 agent 的长期增益从“记住事实”推进到“改进 procedure”。

## 我的判断与保留

我认为这篇最值得留下的是三个点：

- self-improving 的关键不只是记忆，而是程序性知识能否自动沉淀
- 容量受限和显式替换机制，比无上限记忆更接近可维护设计
- Hermes 和 OpenClaw 是两条不同路线：一个更像自生长技能树，一个更像文件化长期环境

## 阅读评级

🔴 建议深读 — 如果你关心 agent 怎么从“记得住”进化到“越用越会做”，这篇很值

## 与其他页面的关联

- [[wiki/ai/openclaw-long-term-memory-pipeline-stability|OpenClaw 长期记忆：优秀管线与玄学效果]] — 一篇讲 OpenClaw 的 file-first memory，一篇讲 Hermes 的 self-improving loop
- [[wiki/ai/ai-agent-memory-systems-architecture-practice|AI Agent 记忆系统]] — 那篇给总览，这篇给一个偏技能生长的具体实现样本
- [[wiki/ai/memory-architecture-ledger-views-policy|Memory 架构与思考]] — 可对照事实记忆与程序性知识各归哪一层
