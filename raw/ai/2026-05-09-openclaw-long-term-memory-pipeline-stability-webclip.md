---
title: OpenClaw长期记忆：优秀管线与玄学效果
source_url: https://mp.weixin.qq.com/s/pLqKTe1J2FkiiquoT4dOJA
saved: 2026-05-09
tags: [ai]
---
城决 *2026年4月15日 18:00*

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naL1VmsZnicicfz6xwf9v3cyiaDticKhPOLC71cN5jS9N1Ky4AqEcCEhHmatgVoB6PUKxDic1oIF0PuUciaw/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0)

> "Memory is limited — if you want to remember something, WRITE IT TO A FILE. ‘Mental notes’ don’t survive session restarts. Files do."
> 
> —— OpenClaw AGENTS.md 默认模板

对于 AI Agent 来说，“记住”是最基础也是最难做好的能力之一。当前的大语言模型在单轮对话中表现出色，但一旦会话结束，所有上下文都从窗口中消失。如何让 Agent 在多轮、跨天的交互中稳定地记住用户的偏好、事实和决策，以及值得记录的事件？OpenClaw 给出了一套以 Markdown 文件为载体的多层记忆体系，其管线覆盖记录、演进、召回全流程——设计理念优秀，但其全流程以 LLM 弱约束的方式进行决策，实际记忆效果往往不够稳定。

本文将从源码层面拆解这套记忆系统的全链路，分析其中的不确定性环节，并介绍 RDSClaw 记忆插件如何补强这些环节，其在LoCoMo10评测中得到了13.90%的提升效果。

一、OpenClaw 记忆系统全景

OpenClaw 的核心设计原则是：一切持久状态都是磁盘上的 Markdown 文件。 Agent 的身份、规则、记忆、工具配置——全部以明文 `.md` 文件的形式存放在工作区目录下，每次会话启动时按优先级注入系统提示词。

完整的文件体系如下：

| 文件 | 用途 | 加载时机 |
| --- | --- | --- |
| `AGENTS.md` | 工作区规则、安全边界、红线指令 | 每次会话（最高优先级） |
| `SOUL.md` | Agent 个性、价值观、沟通风格 | 每次会话 |
| `IDENTITY.md` | Agent 身份元数据（名字、角色、头像） | 每次会话 |
| `USER.md` | 用户档案（名字、昵称、时区、个人背景） | 每次会话 |
| `TOOLS.md` | 环境配置（设备信息、SSH 主机、TTS 偏好） | 每次会话 |
| `MEMORY.md` | 长期记忆（已验证事实、决策、持久学习） | 仅 DM 主会话 |
| `memory/YYYY-MM-DD.md` | 日记忆（当天观察、临时笔记） | 当天 + 昨天自动加载 |
| `DREAMS.md` | 梦境日记（Dreaming 系统输出，仅供人类审查） | 不自动注入 |

可以看到， `AGENTS.md` 、 `SOUL.md` 、 `USER.md` 等文件定义的是 Agent 的身份、规则和用户档案，它们在每次会话启动时被加载，用户和 Agent 都可以在对话中更新（比如 `USER.md` 的模板明确写着 "Update this as you go"）。而 `MEMORY.md` 和 `memory/YYYY-MM-DD.md` 则是另一套机制——它们承载的是 Agent 在对话中积累的动态记忆，并且有一套专门的写入、演进和召回管线。

下面逐层展开。

---

二、记忆写入：两条路径

OpenClaw 的记忆写入有两条主要路径，它们共同负责将对话中的信息写入 `memory/YYYY-MM-DD.md` 日记忆文件。

**2.1 Agent 主动写入（LLM 决策）**

这是最常用的写入路径。在对话过程中，Agent 可以随时主动调用 `write` 工具将信息写入记忆文件：

- 用户显式要求：用户说“记住我偏好 TypeScript”，Agent 主动写入
- Agent 自主判断：Agent 在对话中认为某些信息值得保存，自行决定写入

这意味着：是否写入、写入什么、用什么格式写入，完全由 LLM 在对话中自主决定。 没有结构化的提取规则，没有强制的写入模板——Agent 可能记得很详细，也可能完全不记。

OpenClaw 的默认 AGENTS.md 模板中对记忆写入的指导如下：

```diff
📝 Write It Down - No "Mental Notes"!- Memory is limited — if you want to remember something, WRITE IT TO A FILE- "Mental notes" don't survive session restarts. Files do.- When someone says "remember this" → update memory/YYYY-MM-DD.md or relevant file
🧠 MEMORY.md - Your Long-Term Memory- You can read, edit, and update MEMORY.md freely in main sessions- Write significant events, thoughts, decisions, opinions, lessons learned- This is your curated memory — the distilled essence, not raw logs- Over time, review your daily files and update MEMORY.md with what's worth keeping
```

```
可以看到，这些指导是方向性的建议（"capture what matters"、"write significant events"），而非结构化的提取规则——写什么、怎么写，仍然完全取决于 LLM 的理解和判断。
```

**2.2 Memory Flush 自动写入（LLM 决策）**

Memory Flush 是 Compaction（上下文压缩）前的安全网——确保在激进的上下文裁剪之前，重要信息已被保存。触发条件：

- Token 阈值： `softThresholdTokens` （默认 4000）—— 距离 Compaction 的 token 距离
- 文件大小阈值： `forceFlushTranscriptBytes` （默认 2MB）—— 防止 token 计数器过时

触发时，系统向 LLM 发送一段特殊的提取指令，要求它将当前会话中值得持久化的信息写入 `memory/YYYY-MM-DD.md` ：

```cs
Pre-compaction memory flush turn.The session is near auto-compaction; capture durable memories to disk.Store durable memories only in memory/YYYY-MM-DD.md.Treat MEMORY.md, DREAMS.md, SOUL.md, TOOLS.md, AGENTS.md as read-only during this flush.If nothing to store, reply with NO_REPLY.
```

Flush 期间， `write` 工具被包装为仅追加模式（ `appendOnly` ），只允许写入当天的日记忆文件，不能覆盖已有内容。

**两条路径的关系**

- 主动写入是日常对话中的主要写入方式，但完全依赖 Agent 的自主判断；
- Memory Flush 是上下文压缩前的安全网，确保“最后一次救济机会”；
- 两者都写入相同的 `memory/YYYY-MM-DD.md` 日记忆文件，是后续 Dreaming 系统的输入源。

---

三、默认晋升方式：Agent 主动整理

在不启用 Dreaming 的默认配置下，日记忆到长期记忆的晋升完全依赖 Agent 自主判断：

1\. 对话中直接写入：Agent 在主会话中可以自由读写 `MEMORY.md` ，AGENTS.md 模板明确指导："You can read, edit, and update MEMORY.md freely in main sessions"。当 Agent 认为某条信息足够重要时，可以跳过日记忆，直接写入长期记忆。

2\. 心跳维护：AGENTS.md 模板建议 Agent 在心跳（Heartbeat）期间定期回顾日记忆并更新 `MEMORY.md` ：

```markdown
🔄 Memory Maintenance (During Heartbeats)Periodically (every few days), use a heartbeat to:1. Read through recent memory/YYYY-MM-DD.md files2. Identify significant events, lessons, or insights worth keeping long-term3. Update MEMORY.md with distilled learnings4. Remove outdated info from MEMORY.md that's no longer relevant
```

这套默认机制的特点：灵活但不确定。Agent 可以根据语境自由判断哪些信息值得长期保留，但是否执行、何时执行、整理质量如何，完全取决于 LLM 的自主决策。

---

四、Dreaming 梦境系统：三阶段异步演进

> Dreaming 是 opt-in 功能，默认禁用（DEFAULT\_MEMORY\_DREAMING\_ENABLED = false）。启用后，系统自动创建一个 Cron 任务，默认频率为 0 3 \* \* \*（默认凌晨 3 点执行一次完整扫描，具体时区依配置）。

日记忆并不是最终形态。OpenClaw 设计了一套名为"Dreaming"的后台记忆巩固系统，通过 Cron 定时任务自动运行，将短期信号逐步转化为长期记忆。

Dreaming 由三个阶段组成，每次扫描按顺序依次执行：Light → REM → Deep。

**4.1 浅睡眠（Light Sleep）—— 摄取与去重**

Light Sleep 负责从多个信号源中搜集候选记忆片段：

信号源：

- 日记忆文件：扫描 `memory/YYYY-MM-DD.md` ，逐行提取候选片段（最小 8 字符，最大 280 字符，最多 4 行合并为一个块）
- 会话转录：按 Agent 和 Session 聚合的历史消息（每次最多扫描 240 条，每个文件 12~80 条）
- 短期回忆存储： `memory/.dreams/short-term-recall.json` 中已有的记录

去重：

- 使用 Jaccard 相似度（默认阈值 0.9）进行机械去重
- 重复项合并时取最高的 recallCount、maxScore，合并 queryHashes 和 recallDays

输出：

- 写入日文件的 `## Light Sleep` 块
- 为每个候选记录 `lightHits` 计数（供后续 Deep Sleep 加权使用）

注意：Light Sleep 阶段不调用 LLM。 摄取和去重完全是确定性的文本处理——这意味着它无法理解语义近似，只能依赖字面重叠度来判断重复。

**4.2 快速眼动睡眠（REM Sleep）—— 反射与候选真理**

REM Sleep 对所有候选信号做模式分析，识别反复出现的主题和高置信度的"候选真理"。

主题反射（Theme Reflection）：

系统统计所有候选中的 concept tags（从文件路径和片段内容自动提取）的出现频率，计算主题强度：

```apache
strength = min(1, (count / totalEntries) × 2)
```

仅保留强度 ≥ minPatternStrength 的主题。

候选真理选择（Candidate Truth Selection）：

对每个候选计算置信度分数：

```python
confidence = avgScore × 0.45 + recallStrength × 0.25 + consolidation × 0.20 + conceptual × 0.10其中：  recallStrength = min(1, log1p(recallCount) / log1p(6))  consolidation  = min(1, recallDays.length / 3)  conceptual     = min(1, conceptTags.length / 6)
```

- 去重阈值提高到 Jaccard 0.88（比 Light Sleep 更严格）
- 仅保留置信度 ≥ 0.45 的候选
- 最多选取 3 条候选真理

输出：

- 写入日文件的 `## REM Sleep` 块
- 为每个候选记录 `remHits` 计数

### LLM 决策：梦境日记叙事生成

Light Sleep 和 REM Sleep 在处理完成后，如果有足够的候选材料，会调用一个后台子智能体（subagent）生成"梦境日记"叙事，追加到 `DREAMS.md` 。这是一次 LLM 调用，但生成的内容仅用于人类阅读，不参与后续的晋升评分。

**4.3 深度睡眠（Deep Sleep）—— 六维评分与晋升**

Deep Sleep 是决定一条记忆能否成为"长期记忆"的最终关口。它使用六个加权信号计算综合分数：

| 信号 | 权重 | 计算方式 | 含义 |
| --- | --- | --- | --- |
| 频率 (Frequency) | 0.24 | `min(1, ln(signalCount + 1) / ln(11))` | 被回忆的总次数（recall + daily + grounded） |
| 相关性 (Relevance) | 0.30 | `totalScore / max(1, signalCount)` | 每次被检索时的平均质量分 |
| 多样性 (Diversity) | 0.15 | `min(1, max(uniqueQueries, recallDays) / 5)` | 不同查询/日期上下文的覆盖宽度 |
| 时效性 (Recency) | 0.15 | `exp(-λ × ageDays)` ，λ = ln(2)/14 | 指数衰减，半衰期 14 天 |
| 巩固度 (Consolidation) | 0.10 | `max(0.55×spacing+0.45×span, groundedCount/3)` | 多日重现 或 grounded 信号强度 |
| 概念丰富度 (Conceptual) | 0.06 | `min(1, conceptTags.length / 6)` | Concept 标签密度 |

其中，巩固度取两个分支的最大值：

```sql
// 分支 1：基于 recallDays 的时间跨度spacing = min(1, ln(recallDays.length - 1) / ln(4))span    = min(1, (maxDay - minDay) / 7天)consolidation_a = 0.55 × spacing + 0.45 × span
// 分支 2：基于 grounded 信号计数consolidation_b = min(1, groundedCount / 3)
consolidation = max(consolidation_a, consolidation_b)
```

阶段信号加权提升：

Light Sleep 和 REM Sleep 的命中记录会为候选额外加分：

```javascript
phaseBoost = LIGHT_BOOST_MAX(0.06) × lightStrength × lightRecency           + REM_BOOST_MAX(0.09) × remStrength × remRecency// 衰减半衰期同为 14 天
```

最终晋升分数：

```ini
score = Σ(weight_i × component_i) + phaseBoost
```

晋升门控：

一条候选必须同时满足以下条件才能晋升到 `MEMORY.md` ：

- `score ≥ 0.80` （Dreaming 配置默认最低综合分）
- `totalSignalCount ≥ 3` （合并信号计数，即 recallCount + dailyCount + groundedCount ≥ 3）
- `max(uniqueQueries, recallDays.length) ≥ 3` （独立查询数与有召回记录的天数取较大者）

通过门控的候选会被重新水合（从实时日文件中重新读取片段内容，确保不会写入过时或已删除的内容），然后追加到 `MEMORY.md` 。

### LLM 决策：Deep Sleep 梦境日记

与 Light/REM 类似，Deep Sleep 在晋升完成后也会调用子智能体生成一段叙事性梦境日记，记录本次晋升的摘要。同样，这段内容仅供人类阅读。

---

五、记忆召回与反馈环

持久化的记忆最终通过 `memory_search` 工具被检索和召回：

- 召回工具：在 `MEMORY.md` + `memory/*.md` 上执行检索。OpenClaw 支持 builtin（SQLite + FTS，可选配 sqlite-vec 向量扩展）和 QMD 两种搜索后端；当 embedding 不可用时，builtin 自动降级为 FTS 全文索引 + 词法排名，仍然保持基本的召回能力；
- 信号记录：每次 `memory_search` 返回结果后，系统在后台异步调用 `recordShortTermRecalls` ，对符合短期记忆路径规则的结果进行信号记账（查询、结果路径、评分），写入 `memory/.dreams/short-term-recall.json；`
- Dreaming 反馈环（仅在启用 Dreaming 时生效）：上述记录的召回信号会被 Dreaming 系统消费，影响六维评分中的频率、相关性等指标，形成"越被检索 → 越容易晋升"的正向反馈环；

记忆搜索是 Agent 召回已有记忆的主要通道（启用 Active Memory 插件时，系统会在主回复前自动用子 Agent 调用 `memory_search` 预取相关记忆）。搜索质量直接决定了 Agent 能"记起"多少信息——而搜索质量本身受 embedding 配置、查询精准度等因素制约。

---

六、"随机性"的代价：原生系统的不确定性汇总

上面的管线设计理念是优秀的：异步演进、多阶段过滤、正向反馈。但从工程实现的角度回看，管线中存在多个不确定性环节，它们叠加起来构成了记忆稳定性的核心挑战：

**记忆写入除人为显式提醒外完全依赖 LLM 主观判断**

无论是 Agent 主动写入还是 Memory Flush 自动写入，写什么、怎么写、写多少，都完全由 LLM 在单次推理中自主决定。没有结构化的提取规则，没有强制的输出格式。不同的模型、不同的上下文长度、甚至同一模型的不同推理轮次，写入的内容可能完全不同。你告诉 Agent 自己的名字、城市和饮食偏好，Agent 可能只记住了名字，漏掉了其余两个——即使用户没有明确说“记住这个”。

**Memory Flush 作为安全网仍有盲区**

Memory Flush 本身是 Compaction 前的救济机制，但它只在上下文接近压缩阈值时触发。如果一次对话很短（未触发 Compaction），而 Agent 又没有主动写入，那么对话中的信息就不会被持久化。换句话说，Flush 只能保证“长对话压缩前不丢”，不能保证“短对话也能记住”。

**日记忆到长期记忆的晋升缺乏保障**

无论选择哪条晋升路径，日记忆向 `MEMORY.md` 的晋升都存在不确定性：

默认路径

晋升完全依赖 Agent 在对话中或心跳期间自主决定是否整理日记忆。Agent 可能长期不回顾，也可能整理时遗漏重要信息——没有任何机制保证晋升一定发生。

Dreaming 路径（默认禁用）

即使启用，也面临以下三个环节的不确定性：

Dreaming 演进依赖 Cron 周期

从日记忆写入，到经历 Light Sleep 摄取、REM Sleep 反射、Deep Sleep 晋升，虽然单次 Cron 就会依次执行全部三个阶段，但晋升门控要求合并信号计数 ≥ 3（含 daily、grounded 信号）且 max(独立查询数, 召回天数) ≥ 3——这意味着一条记忆通常需要多次跨日信号积累才能通过门控。对于"我下周二飞杭州"这样的时效性信息，等到信号积累完成，飞机可能已经起飞了。

Jaccard 去重无法捕捉语义近似

Light Sleep 和 REM Sleep 使用 Jaccard 相似度做去重，这是一种基于词汇重叠的方法。"用户喜欢苹果"和"用户爱吃苹果"在语义上是同一件事，但 Jaccard 可能判定它们不相似——结果是同一事实在系统中存在多个版本。

六维评分基于统计信号，而非语义重要性

Deep Sleep 的晋升评分完全基于统计信号（检索次数、出现天数、查询多样性等），没有 LLM 参与语义判断。一条对用户极其重要但只被提及一次的信息（比如"我对花生过敏"），在六维评分中可能远低于一条反复出现但并不重要的信息。

**不确定性链路图**

```js
对话内容  ↓ ❶ LLM 自主判断是否写入（可能遗漏）  ↓ ❷ 短对话无 Flush 安全网（可能不触发）日记忆文件  ↓ ❸ 晋升不确定或不准确  │   ├─ 默认路径：Agent 自主判断，可能不执行或整理不全  │   └─ Dreaming 路径（默认禁用）：  │        Jaccard 机械去重（无语义理解）  │        六维统计评分（无 LLM 语义判断）MEMORY.md  ↓ ❹ 召回受限（无 embedding 时降级为词法匹配）对话上下文
```

**记忆召回受搜索配置制约**

上述不确定性集中在记忆的写入和晋升环节，但记忆的召回同样存在不确定性。 `memory_search` 的召回质量高度依赖配置：有 embedding 模型时走向量语义搜索，无 embedding 时降级为 FTS 词法匹配，可能遗漏语义相关但字面不同的记忆。此外，Agent 是否意识到需要检索、检索时使用的查询词是否精准，也都影响最终的召回效果。

这些不确定性并不是 OpenClaw 的设计缺陷——通用框架为了兼容大部分场景，必须在提取精度和系统复杂度之间做出权衡。但对于需要精确、稳定记住用户事实的场景，这些不确定性覆盖了从写入、晋升到召回的完整链路，叠加效应会显著影响用户体验。

七、为OpenClaw的记忆注入稳定性：

RDSClaw记忆插件openclaw-memory-alibaba-local

RDSClaw的 `openclaw-memory-alibaba-local` 插件可以与 OpenClaw 原生记忆系统共同协作，针对上述不确定性环节提供互补增强。插件从 User 消息中提取两类个人记忆：

- 个人画像（UserImageExtraction）：用户的偏好、个人详情、计划意图等，走 LLM CRUD 整合，个人事实 Evergreen 免衰减
- 世界记忆（WorldImageExtraction）：用户提及的事件、实体、第三方信息等，同样走 LLM CRUD 整合，按策略淘汰

两者共享同一套提取器和分流逻辑（含 `User/用户` 的条目走个人画像，其余走世界记忆）。

**个人记忆管线**

插件采用两阶段实时管线设计，在每轮对话结束时（ `agent_end` 钩子）稳定触发：

```sql
对话内容  ↓ ① 提取器（Extractor）  LLM 结构化提取，配合强制规则  ↓ 分流：个人记忆管线 / 世界记忆管线  ↓ ② 整合器  对每条新事实，向量检索已有记忆 → LLM 判定 CRUD 动作：  INSERT（新事实）/ UPDATE（信息更丰富）/ SKIP（已存在）/ DELETE（矛盾过时）  ↓LanceDB（向量 ANN + BM25 FTS + 标量索引）
```

```
与原生系统最大的区别在于：每轮对话结束即稳定触发记忆提取（不依赖 Agent 自主判断或 token 阈值），每一步都有 LLM 参与语义判断，且整个管线在当轮对话结束时即完成，无需等待 Cron 调度。
```

**核心差异**

| 维度 | OpenClaw 原生 | 插件 |
| --- | --- | --- |
| 提取时机 | Agent 主动写入 + Compaction 前 Flush（被动） | 每轮对话结束即触发（主动） |
| 提取方式 | LLM 自由写入（无结构约束） | LLM 结构化提取 |
| 演进方式 | Cron 三阶段 → 六维统计评分 | 实时 LLM CRUD 整合（INSERT/UPDATE/SKIP/DELETE） |
| 演进周期 | 数天 | 分钟级（当轮对话结束即完成） |
| 去重 | Jaccard 字面相似度 | 向量近似 + 精确匹配 + LLM 语义判断 |
| 矛盾处理 | 依赖统计评分自然淘汰 | LLM 显式识别矛盾 |
| 时间衰减 | Dreaming 评分默认 14 天半衰期；检索侧另有可选时间衰减（默认关闭，半衰期 30 天） | 个人事实 Evergreen（免衰减），世界事件按策略淘汰 |
| 存储后端 | Markdown + SQLite/QMD | LanceDB（向量 ANN + FTS 全文索引 + 标量索引） |
| 召回方式 | memory\_search 单通道 | 混合召回 |

**插件如何补强核心不确定性**

| 原生系统不确定性 | 插件的互补方式 |
| --- | --- |
| LLM 主观写入 | 结构化 Prompt 约束 + 强制规则 |
| 短对话无 Flush 安全网 | 每轮 `agent_end` 钩子自动触发，不依赖 token 计数或 Agent 自主判断 |
| Cron 演进延迟 | 提取→整合→存储在同一轮完成，无需等待调度 |
| Jaccard 机械去重 | 向量相似度 + LLM CRUD 语义整合 |
| 统计评分无语义 | LLM 参与每次整合决策（INSERT/UPDATE/SKIP/DELETE） |
| 召回受搜索配置制约 | 混合召回，不依赖单一搜索后端 |

---

八、不只记住“你”，也记住 AI 工作流：

RDSClaw记忆插件自进化记忆管线

个人记忆管线关注的是 User 说了什么。但对于 AI Agent 来说，还有一类同样重要的信息：AI 自己做了什么、犯了什么错、学到了什么。RDSClaw记忆插件的自进化记忆管线注重从 Assistant 消息中提取这类信息，让 Agent 在后续对话中避免重复犯错、复用已验证的工作流。

**三类提取目标**

| 类别 | 含义 | 示例 |
| --- | --- | --- |
| 最佳实践（learnings） | AI 总结出的可复用行为规则 | "上线前必须重启服务使新代码生效" |
| 错误经验（errors） | AI 犯过的错误和应避免的模式 | "Do not assume X; always check Y first" |
| 行为诉求（feature\_requests） | 用户对 AI 行为的期望和约束 | "删除前必须确认，不要直接执行" |

**提取与召回**

插件支持两种提取方式：

- LLM 提取（默认）：将 User + Assistant 消息组合发给 LLM，结构化输出 `{category, text, importance}` 数组，每轮最多提取 5 条
- 正则提取（轻量级）：通过关键词模式（如 `学习：` 、 `错误：` 、 `lesson:`）快速匹配，不依赖 LLM

提取结果经向量去重（相似度 ≥ 0.92 视为近似重复）后存入 LanceDB。在后续会话的 `before_prompt_build` 阶段，自进化记忆与个人记忆一起被召回并注入系统上下文，影响 Agent 的后续行为。

**与个人记忆的对比**

| 维度 | 个人记忆 | 自进化记忆 |
| --- | --- | --- |
| 消息来源 | User 消息 | User + Assistant 消息 |
| 提取目标 | 用户是谁、关心什么、发生了什么 | AI 学到了什么、犯了什么错、应该怎么做 |
| 典型内容 | "用户偏好 TypeScript"、"用户对花生过敏" | "上线前必须重启服务"、"删除前必须确认" |
| 演进方式 | LLM CRUD 语义整合 | 向量去重 + 存储 |
| 价值 | 让 Agent 记住用户 | 让 Agent 越用越好 |

---

九、LoCoMo10 评测结果

我们使用 LoCoMo10 长对话记忆基准对两套系统进行了对比评测。

LoCoMo

（https://github.com/snap-research/locomo/blob/main/README.MD）是一个用于评估人工智能系统在长上下文会话中记忆与推理能力的基准测试。它被广泛应用于AI记忆系统领域的性能评测，通常包含10个对话集，旨在全面检验模型在单跳回忆、多跳推理、时序推理和开放域生成等多方面的能力。LoCoMo10 覆盖事实查询、时间推理、逻辑推理和描述性问答四大类别，是目前评估 AI Agent 长期记忆能力的主流 benchmark 之一。

| Category | 类型 | OpenClaw 原生记忆 | RDSClaw 记忆插件 | 准确率差值 |
| --- | --- | --- | --- | --- |
| Category1 | 事实查询 | 34.04% | 62.54% | +28.50% |
| Category2 | 时间相关 | 57.01% | 67.07% | +10.06% |
| Category3 | 推理性 | 43.75% | 65.35% | +21.60% |
| Category4 | 描述性 | 68.37% | 78.18% | +9.81% |
| 总体 | 全部类别汇总 | 58.18% | 72.08% | +13.90% |

几个值得注意的数据：

- 事实查询（Category1）提升最大（+28.50%）：这正是插件双管线 + 实时 CRUD 整合的核心优势——用户的个人事实被结构化提取并 Evergreen 存储，不会因为统计信号不足而丢失。
- 推理性问题（Category3）提升显著（+21.60%）：混合召回（向量 + BM25）和 LLM 语义去重让相关记忆的召回更完整，为推理提供了更充分的上下文。
- 总体准确率从 58.18% 提升到 72.08%：在不改变底层 LLM 的前提下，仅通过记忆管线的工程优化就实现了近 14 个百分点的提升。

---

十、推荐实践：RDSClaw 开箱即用

`openclaw-memory-alibaba-local` 插件已内置在 RDSClaw 中：

- 零配置启动：安装即用，LLM 提取和向量索引开箱可用
- 本地 + 远程双模式：支持本地 GGUF 嵌入模型（离线可用）或远程 DashScope 兼容 API
- 多通道覆盖：钉钉、飞书、企业微信——无论在哪个群对话，记忆统一管理
- 安全保障：记忆注入时自动标记为"不可信历史数据"，防止 prompt 注入；敏感信息硬编码排除

如有任何问题，可钉钉加入RDSClaw技术交流群，群号170415008314，欢迎进群交流！

继续滑动看下一个

阿里云开发者

向上滑动看下一个

![kimi](data:image/png;base64,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)
