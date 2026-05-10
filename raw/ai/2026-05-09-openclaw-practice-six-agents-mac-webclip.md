---
title: OpenClaw 实战：一个人、一台 Mac、六个 AI Agent — 从"能聊天"到"能干活"的工程实战
source_url: https://mp.weixin.qq.com/s/S6XKuXHa4QqZ_qno1yGIFQ
saved: 2026-05-09
tags: [ai]
---
岚遥 *2026年4月10日 08:31*

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLWk9nIzroPoh40Ue8UtxmkVtP6u2rFNPLFJPtjEO6OZeZFRpITKSDG9Wfq4ueZLQKHV1Muhnbgnw/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0)

阿里妹导读

文章内容基于作者个人技术实践与独立思考，旨在分享经验，仅代表个人观点。

你的一天被接管了

你睡着的时候，交易蜘蛛已经出了股市收盘报告。你醒来之前，宏观分析师已经写完了 A 股晨报。你还没刷手机，管家蜘蛛已经推来了天气、日程和今日待办。与此同时，AI 哨兵已经扫完了 GitHub Trending、arXiv 最新论文和 100+ 个信息源——18+ 条技术情报按重要性排好序等你看。内容蜘蛛已经在跟踪 54 个平台的热榜和 X 热点。

这是我最关心的部分——AI 动态和技术趋势的自动追踪。哨兵发现有价值的开源项目或论文后，不仅推送新闻，还会评估对我们现有系统的影响，给出 P0/P1/P2 行动建议。有价值的发现会进入 Zoe 的 Tech Radar，走评估 → 决策 → 委派编码落地的完整链路。

从凌晨 3 点的自动备份到 23:45 的全团队反思，52 个 cron 任务每天自动轮转。而且 Agent 在自己进化——犯过的错会被记住，同类问题的复发率显著下降。这不是我写的规则，是 Agent 自己从.learnings/promote 到MEMORY.md的自主迭代。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

1 个编排者 + 5 个专业 Agent + 6 类 ACP 编码专家（最大 6 并发）、52 个 cron 定时任务、118 个 Skills（33 全局共享 + 85 Agent 专属）、29 个注册 LLM 模型、每天几千次 LLM 调用、2086 行运维脚本 + 半个月 23 次自动恢复。

Agent 自己在进化才是真正有意思的部分：

- 自己设计通信协议— 两个 Agent "收到/确认"刷了十几轮，Zoe 自主诊断根因，设计三态协议（request → confirmed → final → 静默），smoke test 通过后沉淀到 AGENTS.md
- 自研 Skill 并发布到 ClawHub— Content 调研 7 个"去 AI 味"工具、A/B 测试、固化为 Skill 发布到 ClawHub，全团队次日自动共享
- 圆桌讨论产出策略报告— Macro 和 Trading 按协议进行下周 A 股策略讨论，产出含数据快照、仓位建议、止损纪律的完整报告
- Task Watcher 异步监控— Agent 承诺"审核通过后通知你"但实际做不到异步回调，Zoe 设计了 cron 级 Task Callback Event Bus 下沉监控

我的角色是搭好框架、设好约束、在关键节点确认方向——具体的需求发现、方案调研、协议设计、代码实现，都是 Agent 自己完成的。

团队：1+5+6 阵型

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

Zoe（大龙虾）— CTO / 首席编排者

不只是"管理员"。Zoe 负责技术方案设计、任务编排、圆桌主持、系统运维和记忆系统维护——每天 3 次巡检（10:00/14:00/22:00），检查所有 Agent 的 cron 执行状态、workspace 磁盘使用、session 健康度。每周分析各 Agent 的 MEMORY.md 是否超限并执行分层压缩。更关键的是，Zoe 会消费 ainews 提供的技术发现，评估哪些值得应用到我们的系统上，征得我同意后指派 ainews 深入调研，然后自行设计方案、委派 ACP 编码专家实现——从技术发现到方案落地的完整链路。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

每次巡检覆盖 6 大维度：cron 任务执行状态（是否有失败/跳过的任务）、workspace 磁盘使用（文件增长异常检测）、session 大小与健康度、Chrome CDP 进程是否泄露、.learnings/中是否有待处理条目、shared-context/时间戳是否正常（检测 Agent 是否"静默失联"）。

Zoe 最有价值的能力是方案设计——三态通信协议、Task Watcher、通信 Guardrail 框架，都是 Zoe 发现问题后自主设计的。下面是 Zoe 设计 Task Watcher 时的方案讨论：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

AI 哨兵（ainews）— 情报中枢

这是我最关心的 Agent。它不是"推新闻"——每天从 100+ 个信息源（GitHub Trending、arXiv、RSS、HackerNews、Reddit 等）采集信息，按 5 星制评估后产出晨报、午间论文解读、晚间趋势分析。7 个 cron 任务覆盖晨午晚三报（08:30/12:00/20:00），每份报告末尾都预留了「改写要点（供 Content 参考）」接口。

更关键的能力是主动评估技术发现对我们系统的影响——本周就发现了 ReMe（记忆管理框架），主动向 Zoe 提出评估建议，从发现到决策到落地，我只需要在关键节点确认，执行全部由 Agent 完成。每日趋势分析中的有价值项目会自动更新到shared-context/tech-radar.json，供 Zoe 每周 Tech Radar 审查：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E) ![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

核心采集工具链：github\_trending.py（--ai-only过滤 +--since weekly周趋势）、rss\_aggregator.py（多源并发采集）、arxiv\_papers.py（多关键词搜索）、Tavily（AI 优化搜索首选）、agent-browser（Playwright 驱动，JS 渲染页面采集）。防幻觉硬约束：每条新闻 MUST 带原文 URL，发布前自检可达性，无法交叉验证的标注「单源，建议核实」。

交易蜘蛛（Trading）— 量化分析师

团队中任务最密集的 Agent——21 个 cron 任务。20 个原子量化工具（quant.pyCLI）、15 个专属 Skills（68000+ 行代码）、65/35 混合评分模型（65% 工具量化 + 35% AI 判断）。覆盖 A 股全时段（集合竞价→盘中每 10 分钟扫描→尾盘速报）+ 美股（盘前→盘中每 30 分钟→盘后夜报）+ 大宗商品（每小时白天+夜盘）。

核心方法论是四步分析框架：读取 Macro 宏观因子 → 多维评分（技术面 25% / 资金面 30% / 基本面 10% / 情绪面 20% / 市场面 15%）→ 逆向检验（与共识一致吗？若错最可能原因？）→ 输出标的+评分(0-100)+止损位+置信度。硬性规则：NEVER 给出没有止损的买入建议，NEVER 编造数据（工具失败时直接报告原因），置信度 <60% 标注「低置信度，建议观望」。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

Macro（首席经济学家）— 数据驱动的四层映射

提供宏观→传导→国内→市场四层映射因子包，供 Trading 直接引用。9 个 cron 任务覆盖晨报(07:50)→午间(12:30)→财经晚报(18:00)→美股盘前(22:00)→美股收盘(次日05:20)。周日 18:30 率先产出周度宏观复盘，Trading 19:30 引用其结论做市场复盘——形成宏观→微观→技术的递进链路。

分析纪律：每个判断标注数据来源和时效性，区分事实（有数据）和判断（有逻辑无直接数据），标注置信度（高>70%/中50-70%/低<50%），每个判断提出反面论据。真实案例——伊朗局势分析时，传统框架预测"地缘→避险→黄金涨"，实际油价+14%但黄金-5%。Macro 的分析指出"油价涨幅>10%，通胀逻辑主导，市场交易的是通胀而非避险"。这个洞察被沉淀到 MEMORY.md 成为持久知识。

内容蜘蛛（Content）— 内容策略师

不是自己"想"内容，而是从团队情报链中提取素材——ainews 提供改写要点、Macro 提供深度分析、Trading 提供市场观点。9 个 cron 任务驱动 Research→Ideate→Write→Reflect 四阶段流水线：09:00 从 54 个平台热榜（微博/知乎/B站/抖音/百度/头条...）+ X 热点抓取 → 10:30 消费 ainews 改写要点生成创意 → 14:00 产出初稿经 Ripple 传播预测引擎评分后投递 → 22:10 反思。

最有意思的是自主进化能力——发现 AI 味太重后，Content 自行调研"去 AI 味"工具，编写 Skill，发布到 ClawHub，并沉淀为全团队通用能力。从发现问题到解决方案再到发布，Agent 自己走完了整个流程：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

另一个自主进化的产物是X 五篮子热点雷达——Content 最初只抓 AI 热点然后摘抄，经过一次反馈后，它自己设计了覆盖五个维度的情报采集框架，并交叉读取其他 Agent 的当日情报：

| 篮子 | 覆盖范围 | 配额 |
| --- | --- | --- |
| AI/科技 | OpenAI / Claude / Agent / LLM | ≤40% |
| 产品/创业 | startup / founder / product launch | 按热度 |
| 一人公司/效率 | solopreneur / productivity / automation | 按热度 |
| 投资/市场/宏观 | stocks / macro / bitcoin / fed | 按热度 |
| 社会情绪/国际 | geopolitics / layoffs / tariffs | 按热度 |

关键约束：AI/科技部分不超过整份输出的 40%。这不是我写在 SOUL.md 里的硬编码，是 Content 自己在反思中迭代出来的——它发现产出全是 AI 内容后，主动给自己加了配额限制。

管家蜘蛛（Butler）— 生活管家

不只是"喝水提醒"——深度集成 Apple 生态（Reminders / Calendar / Health / Notes / Shortcuts），是真正的个人生活助理。7 个 cron 任务：早安问候(08:00)→日程规划(08:30)→5 次喝水提醒（每次换花样：温馨/幽默/知识科普/名人名言/emoji 五种风格随机切换）→健康检查(20:00)→晚安总结(22:00)。

核心理念是不多不少，刚刚好——单次提醒 <50 字，间隔 ≥1.5 小时，23:00–07:00 仅发送紧急事项。如果用户没回复，不会连续催促：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

ACP 编码专家阵型

Pi / Claude Code / Codex / OpenCode / Gemini / GPT-5.3-Codex，通过 ACP 协议按需委派，最大 6 并发实例、120min TTL。分析 Agent 不写代码，编码全部通过sessions\_spawn委派给专家——每种编码 Agent 都可以开多个并发实例。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E) ![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

团队设计教训

一个关键决策是不要让分析 Agent 直接编码。早期我额外设了 coding、architect、PM 三个技术角色，结果发现这几个角色基本没什么实际产出——它们的能力和 Zoe + ACP 编码专家的组合高度重叠，反而增加了通信复杂度和调试成本。后来全砍了，编码通过 ACP 委派给 Claude Code 等专业工具。PM 和架构师？Zoe 兼任就够了。

复杂度随人数快速上升。3 个 Agent = 3 对交互关系，6 个 = 15 对。整个系统从零到 6 个 Agent 稳定运行，花了大约半个月的下班时间——每加一个新 Agent 都需要半天到一天的调试，处理和现有 Agent 的通信冲突、共享资源竞争、规则兼容。

一天是怎么过的

从凌晨 03:00 的自动备份到 23:45 的全团队反思，52 个 cron 任务覆盖 A 股 + 美股双时区自动轮转。周日还有 Macro → Trading → Trading 的三级递进周报。

每天结束时，每个 Agent 独立反思当天的.learnings/待审条目，Zoe 最后汇总全团队产出：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

从凌晨 03:00 的自动备份到 23:45 的全团队反思，52 个 cron 任务覆盖 A 股 + 美股双时区自动轮转。周日还有 Macro → Trading → Trading 的三级递进周报。

每天结束时，每个 Agent 独立反思当天的 `.learnings/` 待审条目，Zoe 最后汇总全团队产出：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

让这套系统跑起来：三个核心工程问题

上面展示的是最终效果。但从"装好 OpenClaw"到"系统流畅运作"，再到"Agent 自主进化"——中间有巨大的鸿沟。三个核心工程问题，每一个都不是"写好 prompt"能解决的。

核心问题一：上下文是 Agent 的操作系统

问题：Agent 系统的热力学第二定律

不加约束，entropy 只增不减。持续运行的 Agent 系统会确定性地走向崩溃——不是"可能"，是"一定"。

Agent 就像一个没有操作系统的进程：它能处理输入、产出输出，但谁来管理它的内存（上下文）？谁来做垃圾回收（Session 清理）？谁来防止 OOM（膨胀保护）？不设计就没有人管。

三个真实事故，按严重度排序：

P0 — 全团队瘫痪 8 小时

ainews 的 session 因为连续处理新闻和论文累积到235K tokens。Gateway 启动时对所有 session 做 compaction，这个 session 永远超时 → crash → macOS 守护进程ThrottleInterval=1每秒重启 → 无限循环。所有 Agent 全部离线。

修复要四层：手动清理膨胀 session →ThrottleInterval1→10 →idleMinutes180→30 →exec.securityfull→allowlist。这不是某一个参数的问题，是四个独立的防线全部缺失。

P1 — 3500 字报告被框架"优化"到 800 字

交易蜘蛛的收盘速报包含完整的数据表格、资金流向、个股评分。OpenClaw 在文本超过textChunkLimit时自动做 content compaction（LLM summarize），数据表格被"智能压缩"掉了。框架认为"帮你优化了"——但在数据密集场景下，AI 的"智能"是灾难。

P2 — 信息过载后关键规则失效

当 SOUL.md 里堆满了各种操作规范、当 session 膨胀到几万 tokens，Agent 开始"选择性遵守"规则。管家蜘蛛越界做投资分析。交易蜘蛛忽略数据验证规则。不是模型变笨了，而是关键信息被噪声淹没了——这是 Context Engineering 要解决的根本问题。

解决：双层控制 — Context Engineering + Harness

两层协同，两个都不能少。

第一层 — Context Engineering（设计 Agent 的信息架构）

设计 Agent 每次推理时看到的完整信息结构——系统级的信息架构设计：

- SOUL.md放在 system prompt 最前面，是 Agent 的"宪法"——身份定义、决策框架、绝对禁止项。保持精简，只放最核心的约束
- AGENTS.md跟在 SOUL.md 后面，定义操作规范和协作协议
- Skills通过extraDirs配置按需加载——Trading 有 15 个 Skills 共 68000+ 行内容，不可能全放在 system prompt 里。只在 Agent 需要用到某个 Skill 时才注入上下文
- shared-context/是跨 Agent 的共享状态，Agent 通过工具主动读取
- Obsidian Vault是冷存储，归档产出但不参与推理

LLM 的上下文窗口不是等价的。system prompt 前面的信息权重远高于后面的。session 膨胀到几万 tokens 后，早期消息的影响力被稀释——类似操作系统的内存管理：热数据放缓存，冷数据放磁盘，关键数据常驻内存。

Trading 的 15 个 Skills 中，stock\_analysis（技术面评分）在每日扫描时才需要，bilateral\_analysis（龙虎榜分析）只在异动时触发。全部常驻上下文的话，噪声淹没了真正重要的规则。通过extraDirs按需注入，每次推理只加载相关的 1-3 个 Skill。

规则措辞必须面向最弱的模型。多模型 fallback 环境下（GPT-5.4 超时 → Qwen3.5+ → Ollama qwen3:8b），规则遵循率随模型能力递减：

- "建议不要编造数据"→ GPT-5.4 基本遵守，Qwen3.5+ 偶尔遵守，qwen3:8b 几乎无视
- "MUST: 不编造数据"→ 各模型遵守率显著提升
- "MUST + P0 + NON-NEGOTIABLE"→ 即使弱模型也能保持较高的遵守率

多模型 fallback 时，不知道哪次推理会用弱模型，所以所有关键规则都要按最弱模型的理解能力来写。显式 > 隐式，硬规则 > 软建议。

第二层 — Harness（框架自动管理）

Agent 7×24 运行，session 会持续膨胀——你设计得再好，运行一天之后上下文就变了。框架替 Agent 管，OpenClaw 的 harness 配置提供自动化的上下文生命周期管理：

| 机制 | 触发条件 | 动作 | 为什么需要 |
| --- | --- | --- | --- |
| **compaction memoryFlush** | Session 超过 40K tokens | 提取精华到 `memory/YYYY-MM-DD.md` | 防止 session 无限膨胀 |
| **contextPruning** | 上下文超过 6 小时 | cache-ttl 裁剪，保留最近 3 条 | 防止旧上下文干扰新推理 |
| **session reset** | 每天 5:00 或空闲 30 分钟 | 自动重置 | 防止跨天数据残留 |
| **session maintenance** | 文件超过 7 天 | 自动清理，磁盘上限 100MB | 防止磁盘被撑满 |
| **self-improving-agent Skill** | Agent 启动时 | 注入 `.learnings/` 历史经验 | 确保学到的东西不丢失（额外安装的 Skill） |

只有 Context Engineering 没有 Harness，session 膨胀到 235K tokens 后一样崩溃；只有 Harness 没有 Context Engineering，所有信息堆在一起、关键规则被噪声淹没。Context Engineering 定义信息的结构，框架管理信息的生命周期。

实际的openclaw.json配置（每个参数背后都是真实事故）：

```json
{  "compaction": {    "mode": "safeguard",    "memoryFlush": {      "enabled": true,      "softThresholdTokens": 40000,      "prompt": "Distill to memory/YYYY-MM-DD.md. Focus: decisions, state changes, lessons, blockers."    }  },  "contextPruning": { "mode": "cache-ttl", "ttl": "6h", "keepLastAssistants": 3 },  "session": {    "reset": { "mode": "daily", "atHour": 5, "idleMinutes": 30 },    "maintenance": { "pruneAfter": "7d", "maxDiskBytes": 104857600 }  },  "hooks": {    "bootstrap": ["self-improving-agent"]  }}
```

四个机制的执行顺序很重要——compaction 先于 contextPruning，确保有价值的内容先被提取到memory/，再被清理。self-improving-agent 的 bootstrap hook 在新 session 启动时触发，把.learnings/和MEMORY.md注入上下文——这是 Agent"记住上次学到的东西"的关键机制。

跨会话记忆恢复链（Agent 重启后如何"想起来自己是谁"）：

```objectivec
新 session 启动  ↓ self-improvement hook: 读 SOUL.md → AGENTS.md → MEMORY.md → .learnings/  ↓ memorySearch: 从 memory/ + sessions 中检索与当前任务相关的历史上下文  ↓ 读取 shared-context/ (团队实时状态)Agent 恢复到"知道自己是谁、做过什么、团队现在在干什么"的状态
```

Agent 不需要"记住"所有对话——memoryFlush 提取的是 decisions/lessons/state changes，不是完整对话。memory/中的文件通常只有几百行，而原始 session 可能有几万 tokens。

核心问题二：让 Agent 真正"记住"并"成长"

问题：Agent 今天犯的错，明天还会犯

这是一个比上下文管理更深层的问题。chatbot 每次对话从零开始，所以它每次都犯同样的错是正常的。但如果你的 Agent 7×24 运行、每天处理几千次 LLM 调用，你会无法接受它反复犯同一个错误。

交易蜘蛛5 次把龙虎榜 API 字段名搞错——BILLBOARD\_BUY\_AMT写成BUY\_AMT，每次 session 重置后记忆丢失，下一次又犯。用户纠正"昨天建议买军工，今天跌了就转空"，Agent 当场改了，但三天后遇到类似场景又给出同样的单向建议。

chatbot 和 Agent 的分水岭就在这里：Agent 应该能从错误中学习，并且下次不犯。但怎么实现？

解决：五层记忆 — 从人类认知模型中借鉴

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

设计记忆系统时，我参考了人类记忆的分层模型：工作记忆（短期）→ 长期记忆（经验）→ 程序性记忆（技能）。Agent 的记忆也应该有不同的时间尺度和管理方式：

| 层 | 存储 | 时间尺度 | 管理方式 | 典型内容 |
| --- | --- | --- | --- | --- |
| L1 身份层 | SOUL.md (精简核心) | 永恒 | **人工确认**  修改 | 身份 + 硬约束 + 决策框架 |
| L2 长期记忆 | MEMORY.md (<3000 tokens) | 长期 | Agent 自主维护 | 结构化经验（✅ 成功模式 / ❌ 失败教训） |
| L3 中期记忆 | memory/YYYY-MM-DD.md + memory.db | 中期 | **Harness 自动**  提取 | Session 超过 40K tokens 时的精华快照 |
| L4 短期记忆 | .learnings/ (ERRORS/LEARNINGS/FEATURES) | 短期 | Agent 即时记录 | 错误记录、用户纠正、最佳实践 |
| L5 持久化 | Skills + Obsidian + ontology + vector\_store.db | 持久 | 共享/归档 | 技能库 + 知识归档 + 知识图谱 + 向量检索 |

层与层之间的衔接才是关键——这形成了一个完整的自主进化循环。

记忆自主迭代——6 步循环

这是整个系统最核心的机制。没有这个循环，Agent 只是 chatbot；有了这个循环，Agent 才是真正的 Agent。

记忆自主迭代 — 6 步循环

1）触发事件

操作失败 · 用户纠正 · 发现更优做法 · 需要新能力

任意一种都会触发即时记录

2）.learnings/ 即时记录

ERRORS.md · LEARNINGS.md · FEATURE\_REQUESTS.md

状态: pending — 低成本、高频率、不审查直接写入

3）每日反思 Cron (23:00-23:45)

扫描.learnings/ 所有 pending 条目 → 评估复现频率和重要性 → 验证 ≥3 次？

✓ ≥3 次 promote 到 MEMORY.md

✗ <3 次 保留 pending，继续观察

4）promote 到 MEMORY.md

长期记忆 · <3000 tokens 硬上限 · 超限时 Agent 自主精简（合并相似、删除过时）

5）下次 Session 加载

self-improvement hook · bootstrap 注入 · 检查.learnings/ · 引用历史经验

6）Agent 行为改进

下次遇到同样问题时自动避免 — 迭代完成

🔒 SOUL.md 修改— 需要用户确认（Agent 不能自己改自己的"宪法"）

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

Agent 可以自主更新.learnings/、MEMORY.md、memory/、knowledge/——但绝对不能改 SOUL.md。SOUL.md 是身份和硬约束，修改需要用户确认。我们真的遇到过 Agent 把自己的"人格"改松的情况——行为立刻变得不可控。

Zoe 每周还会做记忆系统维护——分析各 Agent 的 MEMORY 状态，做归档和压缩：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

L5 的 ontology 知识图谱记录实体关系（Agent/Task/MarketInsight/Decision），vector\_store.db提供语义级检索——Agent 不需要记住精确措辞，通过向量相似度找到相关历史决策。

真实进化案例

```makefile
用户纠正: "昨天建议买军工，今天跌了就转空"  ↓即时记录 (.learnings/):  "[LRN-20260303-001] correction | priority: high | status: pending   军工策略在地缘利好场景下未充分强调短线拥挤与顶背离风险   Suggested Action: 条件单模板 — 入场带失效位 + bullish/base/bear 三情景"  ↓每日反思 cron (23:30): promote 到 MEMORY.md  ↓MEMORY.md: "❌ 事件驱动标的必须用条件单模板"  ↓三周后遇到类似板块轮动:  交易蜘蛛直接引用了这条经验
```

这不是我写的规则——是 Agent 从纠正中自己提炼出方案，自己写入长期记忆。每日反思 cron 会审查.learnings/中的 pending 条目并决定是否 promote：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

更多真实进化记录：

| 发现 | 进化 | 来源 |
| --- | --- | --- |
| SOUL.md 信息过载导致规则失效 | 精简核心，非核心规则迁移到 Skills 按需加载 | 行为异常排查 |
| `delivery=ok`  ≠ 知识库已落库 | 反思同时核对投递 + 文件存在性 | 管家零产出事件 |
| butler 零产出但反思说"正常" | 建议性兜底 → 硬门禁（产出为空 = 失败） | 运维巡检 |
| macro→trading 引用缺乏追溯 | 强制结构化引用 + 验证字段 | 数据追溯困难 |
| trading 频道刷屏（双发送链路） | 幂等键 + 单层重试 + 节流 | P0 事故 |
| 军工策略只看事件驱动 | 条件单模板：入场+失效位+三情景 | **Agent 自提方案** |

记忆系统还在进化

记忆系统不是"设计好就不变的"。最近的一个例子：

Zoe 每周生成Tech Radar 报告，从 ainews 的情报中提取技术趋势，分为 Adopt/Trial/Assess 三级。本周报告中，ainews 发现了（Agent 记忆管理框架），Zoe 立刻评估其与现有系统的对比：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E) ![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

结论：ReMe 架构优秀（223K → 1.1K tokens，99.5% 压缩率），但与 AgentScope 深度绑定，直接接入不划算。走"参考设计自研"——先实现收益最高的tool\_result 压缩（超长工具输出自动截断 + 外存，上下文占用降 80%+），再逐步引入结构化摘要模板和异步持久化。

用户同意后，Zoe 立即通过 ACP 协议委派 Claude Code 做架构评审：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

这个过程本身就是多 Agent 协作的真实案例：ainews 情报发现 → Zoe Tech Radar 评估 → 用户确认 → ACP 委派编码专家 → 分阶段落地，涉及 3 个 Agent + 1 个 ACP 编码专家。

假设驱动的迭代 — 从修 Bug 到科学方法

记忆系统最有深度的进化不是修某个具体的 Bug，而是 Agent 学会了主动提出假设并验证——这是从"被动修复"升级到"主动改进"的关键一步。

每日反思中，Agent 会基于当天的工作提出 3-5 条可验证假设，晚间反思时用实际数据评估：

| 假设 | 验证结果 | 后续动作 |
| --- | --- | --- |
| 评分报告加上推理过程可降低用户质疑 | **已验证**  ：Trading 加上推理链后用户追问显著减少 | 固化为评分模板硬性要求 |
| Macro→Trading 引用上游结论可减少重复分析 | **已验证**  ：从"每次重新分析"变为"引用+增量" | 写入协作协议 |
| 端到端评测集比单点指标更能提升日报质量 | **验证中** | 正在建立评测基准 |

验证通过的假设被固化为规则或 Skill，验证失败的被标注原因并淘汰。Agent 不只是"犯错后改正"，而是在主动寻找改进空间——从 reactive 到 proactive 的跃迁。

自主进化机制：哪些是 OpenClaw 自带的，哪些是我们加的

理解这套记忆系统需要区分框架能力和运营优化——很多人问"OpenClaw 是不是开箱即用"，答案是"框架能力开箱可用，但要跑好需要大量运营层设计"。

| 能力 | OpenClaw 框架自带 | 我们的运营层优化 |
| --- | --- | --- |
| **Session 管理** | compaction（memoryFlush）、contextPruning、session reset、session maintenance | 调优参数（40K/6h/30min）来自多次事故复盘 |
| **self-improving-agent Skill** | ❌ 框架不自带 | **额外安装的 Skill**  ——Agent 启动时注入 `.learnings/` 提醒，驱动学习记录和持续改进 |
| **Memory flush** | 超过阈值自动提取精华到 `memory/` | 定制 flush prompt 强调"decisions/state-changes/lessons/blockers" |
| **Skills 加载** | `extraDirs`  按需注入 | 15 个 Trading 专属 Skill、33 个全局共享 Skill，按需加载 |
| **ACP 委派** | `sessions_spawn`  \+ 编码 Agent session 管理 | 委派策略（什么任务用什么编码专家）、TTL 和并发调优 |
| **反思 + 自我迭代** | ❌ 框架不自带 | **完全自建**  ——每日 23:30 每个 Agent 独立反思 + Zoe 汇总，包括.learnings 审查、MEMORY 精简、Tech Radar 技术趋势提取、ReMe 等新技术评估 |
| **三态通信协议** | ❌ 框架不自带 | **Zoe 自主设计**  ——从刷屏问题出发，迭代到 V1 线程协议 |
| **Task Watcher** | ❌ 框架不自带 | **Zoe 设计 + ACP 编码落地**  ——task-watcher Skill |
| **MEMORY.md 容量管理** | ❌ | `memory_maintenance.py`  每周压缩 + Agent 自主精简 |
| **ontology 知识图谱** | ❌ | 自建 schema.yaml + graph.jsonl 实体关系 |

OpenClaw 提供了优秀的框架级基础设施（Session 管理、Harness、ACP），但让 Agent 真正"活"起来的进化机制——反思迭代、协作协议、Task Watcher、记忆压缩——都是在框架之上的运营层设计。

核心问题三：多 Agent 协作是协议问题，不是群聊问题

问题：把 Agent 放进群聊 ≠ 协作

大多数人对 Multi-Agent 的直觉是"给几个 Agent 一个聊天群，它们就能协作"。实际上，这和把几个工程师拉到一个没有流程规范的群聊里没有区别——有沟通能力不等于有协作能力。

Macro 和 Trading 在"伊朗局势对 A 股影响"上互相"收到/确认/感谢"刷了十几轮。分析早就做完了——Macro 判断"油价涨幅 >10% → 通胀逻辑主导 → 黄金反跌"（实际走势：油价 +14%，黄金 -5%，判断准确）——但分析之后两个 Agent 客套的 token 比分析本身还多。

根因不是"Agent 太客套"。根因是缺乏终态协议。Discord 配置中requireMention=true表示 Agent 只在被 @ 时才回复。当两个 Agent 互相 @，A → B → A → B......这就是经典的 ACK storm，和 TCP 协议早期遇到的问题是一样的。

解法也是一样的：设计协议。不是告诉 Agent "少说话"——实际观察中"建议"式规则在弱模型上几乎不起作用——而是设计一个有状态机的通信协议。

解决：协议级设计 → 真实案例

实例：下周 A 股策略圆桌讨论

Zoe 发起圆桌 → Macro 提供宏观研判 → Trading 回应策略建议 → 按固定三态协议有序收敛：

Step 1 — Zoe 发起议题 + Macro/Trading 按协议 confirmed：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

Step 2 — Trading 基于 Macro 研判给出详细策略（confirmed 输出）：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

Step 3 — Trading 输出 final（DRI 结论 + 完整推理过程）：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

Step 4 — 协议收敛（final 后全员静默）：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

协议设计

固定三态通信协议 + V1 线程协议（被刷屏教训后设计，已迭代到 V1 版本，沉淀到 AGENTS.md）：

```swift
固定三态协议（强制）━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[request]    @对方 + ack_id + 期望动作 + 截止时间             模板: @agent [state=request] [ack_id=topic-v1-202603081430][confirmed]  @发起方 + 相同 ack_id + 版本号/生效时间/关键结论             模板: @requester [state=confirmed] [ack_id=...] 版本=v2[final]      @相关方 + 相同 ack_id + 终态收敛（全线程仅 1 条）             发出后全员进入静默，"收到/感谢/OK" → NO_REPLYV1 线程协议（2026-03-08 起）━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━• 同一线程只允许一个 ack_id，新一轮必须新开• final 后禁止续话；必须补充时优先 edit 既有消息• sessions_send 超时 ≠ 失败 → 同一 ack_id 不得重试• 同一内容最多重试 1 次；第二次超时 → shared-context/ 文件投递⏰ 5 分钟无 confirmed → 催办 1 次 · 10 分钟仍无 → 升级 Zoe 仲裁
```

子线程策略：多轮协作的内容任务默认开专用子线程（命名:<主题>-<负责人>-<日期>），主频道只同步三次状态：\[Dispatch\]→\[ACK\]→\[DraftReady\]。

三种通信机制：

| 机制 | 用途 | 例子 |
| --- | --- | --- |
| `sessions_send` | 实时任务分派/圆桌讨论 | Zoe → Macro "分析伊朗局势" |
| `shared-context/` | 异步状态共享 | Macro 写入宏观因子包 → Trading 直接读取 |
| 知识归档 | 结构化素材接口 | ainews 报告末尾留"改写要点" → content 消费 |

shared-context/的核心价值：从消息驱动升级到状态驱动。Trading 不需要每次问 Macro"今天宏观怎么样"，直接读intel/finance\_news\_latest.json。sessions\_send 适合实时触发，但不可靠（超时、重复）——关键数据走文件才可追溯。

Zoe 主导了shared-context/的标准化——从最初零散的文件目录，演进为结构化的跨 Agent 协作基础设施：

```cs
shared-context/├── agent-sessions/       # ACP 编码专家的 session 状态（30 个 claude/codex session）├── agent-runs/           # Agent 运行记录├── monitor-tasks/        # Task Watcher 持久化存储│   ├── tasks.jsonl       # 任务注册（小红书审核/ACP完成/cron健康等）│   ├── watcher.log       # 轮询日志│   ├── audit.log         # 审计追踪│   ├── dlq.jsonl         # 死信队列（处理失败的任务）│   └── notifications/    # 通知记录├── intel/                # 情报共享（finance_news_latest.json 等）├── roundtable/           # 圆桌讨论记录├── decisions/            # 重大决策存档├── job-status/           # cron 任务状态├── knowledge-base/       # 共享知识├── status/               # 各 Agent 当前状态 JSON├── tech-radar.json       # 技术雷达（Adopt/Trial/Assess 三级）├── memory-maintenance-latest.json  # 最近一次记忆压缩报告└── PROJECT_STATUS.md     # 项目全局状态（Zoe 维护）
```

这不是一次性设计出来的——是 Zoe 在实际运营中逐步标准化的。每增加一个新的协作场景（ACP 编码、Task Watcher、Tech Radar），Zoe 就在shared-context/中增加对应的标准化目录和文件格式。

DRI 原则：一个问题只有一个 Directly Responsible Individual 出最终结论。非 DRI 只能补充，不能覆盖。Zoe 组织和归档，不替代专业 Agent 出专业意见。

协议优化后的自主反思

协议落地后，Agent 不只是"按规则执行"——它们会主动反思效果并提出改进方向：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E) ![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

从初版"禁止客套"到 V1"线程级收敛"，每一步协议优化都来自 Agent 的.learnings/经验沉淀——这正是五层记忆系统的价值所在。

最新进展（2026-03-08）：Zoe 刚完成了一次全员通信标准下沉——修改了 26 个文件（6 个 Agent 的 AGENTS.md + SOUL.md + 相关 Skill 文档），把通信硬规则统一写入每个 Agent 的本地配置。同时执行了全员通信路径审计，识别出main/SOUL.md与圆桌 Thread 规则的冲突并修复。

五条联动链路

Agent 之间不是各干各的，是上游主动为下游准备接口：

| 链路 | 流转 | 机制 |
| --- | --- | --- |
| ainews → content | ainews 每份报告末尾留"改写要点" | 约定接口格式 |
| ainews → Zoe | Tech Radar 技术雷达 → Zoe 评估与决策 | shared-context/tech-radar.json |
| Macro → Trading | 宏观因子包（DXY/US10Y/油价方向/Fed路径/板块映射） | shared-context/intel/ |
| Trading → Macro（美股跨时区） | Trading 05:10 夜报 → Macro 05:20 宏观复盘 | 时序联动 |
| Macro → Trading（周末递进） | 18:30 宏观→ 19:30 市场→ 20:30 技术 | 递进链路 |
| Zoe → 全团队 | 23:45 跨 workspace 读取 6 Agent 产出 +.learnings/ | 反思汇总 |

Tech Radar 真实示例——ainews 每日维护的技术雷达，分为 Adopt（已验证可用）、Trial（值得尝试）、Assess（持续观察）三级：

```json
{  "adopt": [    { "name": "MCP Protocol", "reason": "MCP 2.0 发布，三大框架推进标准化" }  ],  "trial": [    { "name": "OpenAI Skills Catalog", "reason": "582→947 星，可参考 Skill 格式" },    { "name": "ReMe Agent 记忆管理", "reason": "独立记忆工具包，评估替代方案" }  ]}
```

Zoe 消费 Tech Radar 后做源码级评估，判断对现有系统的影响——本周就基于此发起了 ReMe 评估并委派 Claude Code 落地 PoC。

安全边界 — Agent 能改什么、不能改什么

安全在于限制 Agent 能触碰的范围：

| 安全层 | 机制 | 教训 |
| --- | --- | --- |
| **执行权限** | `exec.security: allowlist` | Content 曾自己改坏配置 → 改为白名单执行 |
| **配置保护** | SOUL.md / openclaw.json 不允许 Agent 修改 | Agent 把自己的"人格"改松 → 行为失控 |
| **密钥隔离** | API keys 在 env 中，不在文件中 | 防止暴露到 session 或 Discord |
| **代码审查** | ACP 编码走 review 流程 | Agent 生成的代码不直接部署 |

Task Watcher — 解决"Agent 说了会做但实际没做"

Agent 最难发现的问题不是崩溃或报错，而是\*\*"说了会做但实际没做"\*\*。Content 蜘蛛发完小红书说"审核通过后通知你"——但 session 已经结束了，它根本做不到异步回调。更隐蔽的是 cron 任务"执行了"但零产出——反思也说"一切正常"。

Zoe 主导设计了一套Task Callback Event Bus——插件式架构，5 个组件各司其职，把异步监控下沉到 cron 级别：

```bash
注册任务 → tasks.jsonl（shared-context/monitor-tasks/）                ↓Cron (*/3 min) → Watcher → Adapter 检查状态 → 状态变化？                                                   ↓ Yes                                      Policy 决策 → Notifier 发 Discord
```

- Adapter 插件：小红书审核状态、GitHub PR、ACP 编码任务——想监控什么加一个 Adapter
- Policy 策略：通知频率、升级、重试都可配
- 6 小时超时保护（默认），3 次投递失败自动升级，不会死循环、不会卡死

这套系统由 Zoe 自行设计 → 委派 Claude Code 实现 →130 个单元测试→ 开源发布为 OpenClaw Skill。从需求到代码到测试到发布，我只在方案评审时介入，其余由 Agent 团队完成。

通信 Guardrail + 异步状态链（最新进展）

Task Watcher 解决了"异步任务有没有产出"的问题，但更深层的问题是：Agent 之间的通信本身缺乏系统级约束。message被误用为内部控制面、timeout不等于失败却被当作失败处理、completed和delivered无法区分——这些都不是靠文档规则能解决的。

Zoe 自主设计并委派 ACP 编码专家实现了一套通信 Guardrail + 请求生命周期状态链（~3000 行 Python），核心组件：

| 模块 | 行数 | 作用 |
| --- | --- | --- |
| `agent_comm_guardrail.py` | 383 | 5 条硬性规则：拒绝 message 误用、阻止身份伪造、拦截 ack\_id 重发 |
| `agent_request_models.py` | 289 | 11 态生命周期模型： `accepted → routed → queued → started → completed → delivered` |
| `agent_request_store.py` | 529 | 文件级状态存储， `requests.jsonl` + `events.jsonl` 全链路审计 |
| `completion_bus.py` | 507 | 异步完成投递总线，producer-consumer 模式 |
| `acp_state_bridge.py` | 502 | ACP 编码任务的状态桥接 |
| `dead_letter_queue.py` | 271 | 投递失败兜底队列 |

设计决策：

- timeout ≠ failed：超时只是控制面观察结果，任务可能已经在执行——引入ambiguous\_success语义
- completed ≠ delivered：工作完成 ≠ 结果送达，分离两个状态避免投递失败覆盖工作成果
- 通信生命周期独立于业务 TaskState：不污染现有 Task Watcher 的submitted → completed → failed状态机
- 文件级状态源：先用shared-context/agent-requests/跑通，不依赖 Redis/MQ 等重量级设施

这套系统从"Zoe 发现问题 → 设计方案 → 委派编码 → 代码实现 → 测试验收"，我只在方案确认环节介入，其余环节由 Agent 自主完成——是 Agent 从"执行者"进化为"系统设计者"的典型案例。

## 整体架构总结 — 五层工程视角

回过头看，整个系统的核心设计可以归结为五个层次：

| 层 | 核心机制 | 解决什么问题 |
| --- | --- | --- |
| **通信层** | 三态协议 + ack\_id + 四层一体化 + shared-context/ | Agent 之间如何可靠地协作 |
| **记忆层** | 五层分层存储 + Harness 自动管理 + 反思迭代 | Agent 如何记住经验并持续成长 |
| **自愈层** | 三层自愈架构 + heartbeat-guardian + memory\_maintenance | 系统如何 7×24 稳定运行 |
| **进化层** | .learnings → promote → MEMORY + Skill 自研 + ClawHub 发布 | Agent 如何从"执行者"变成"设计者" |
| **编排层** | Zoe 巡检 + 圆桌主持 + Task Watcher + ACP 委派 | 谁来管理和协调这一切 |

这五层不是独立的——它们相互依赖、相互强化。通信层的三态协议是 Zoe 在编排层中发现问题后设计的。记忆层的压缩策略是自愈层的一部分。进化层的 Skill 自研能力来自通信层的跨 Agent 协作。

跑了半个月之后的认知变化

1\. 90% 的时间花在工程问题上，不是 AI 问题上。Session 膨胀、消息风暴、配置被改坏——解法在分布式系统和 SRE 经典知识中，不在 AI 论文里。Agent 系统的瓶颈不是模型能力，是基础设施的成熟度。模型升级是锦上添花，通信协议、记忆架构、自愈机制才是决定成败的底座。

2\. AI 的"智能"在生产环境中经常是灾难。Discord 消息被"智能压缩"砍掉了数据表格，Agent "智能修复"自己的配置改坏了工具名，管家蜘蛛在 session 膨胀后"智能地"越界做投资分析。在需要精确、可预测输出的场景下，"智能"反而是负面特性。显式 > 隐式，硬规则 > 软建议，可预测 > 可解释。

3\. 持续运行的系统必然退化——不是 bug，是热力学。配置堆积、记忆过长、session 膨胀、磁盘撑满——这些会确定性地发生。对策不是"一次设置好"，而是建立反退化机制栈：compaction 管 session，maintenance 管记忆，heartbeat-guardian 管配置，巡检管行为漂移。用 Agent 运维 Agent，用 cron 监控 cron——每层兜底机制都需要自己的兜底。

4\. 协作是协议问题，不是 prompt 问题。两个 Agent 放在同一个 Thread 里不写协议，等价于两个进程共享内存不加锁。Macro 和 Trading 用同一个模型、同一个知识库，刷屏时每条回复都言之有物——加上三态协议后产出从十几轮废话变成了一份可执行策略文档。模型没变，变的是规则。

5\. Agent 最大的价值不是执行力，而是"参与设计"。当 Agent 从"你让我做什么我就做什么"进化到"我发现了问题，调研了三种方案，推荐 B，你确认我就落地"——这时它才真正成为团队成员。十个进化案例里，大多数的起因不是"我让它做什么"，而是"它遇到了问题然后自己想办法"。系统设计的目标不是让 Agent 听话，而是让它有能力自己解决问题。

如果你也想试试

不需要照搬 6 个 Agent。从 1 个开始。整个系统从零到 6 个 Agent 稳定运行花了大约半个月的下班时间——不是开发时间，是调试和填坑时间。

第 1-2 天：先让 1 个 Agent 稳定运行

最重要的三件事：

1\. SOUL.md 保持精简，只放核心约束。把它当"宪法"不是"操作手册"——非核心规则放 Skills 按需加载

2\. Session 管理参数第一天就设好：idleMinutes=30、pruneAfter=7d、maxDiskBytes=100MB。不设 = 定时炸弹

3\. 从第一天就启用.learnings/+ 反思 cron。没有反思的 Agent 只是 chatbot，不是 Agent

第 3-5 天：加第 2 个，开始处理协作

4\. Discord 配置比你想的复杂 10 倍。每个 Agent 需要独立 Bot 账号。requireMention、textChunkLimit、delivery.mode、子 Thread 创建、Bot 权限——每个都有坑，而且组合起来的症状让你根本猜不到是哪个配置的问题

5\. 协作需要协议，不是群聊。两个 Agent 在群聊里会互相 ACK 到死。固定三态协议 + ack\_id + 超时升级，两个都不能少

第 2 周起：逐步扩展到完整阵型

6\. 规则用最强措辞，面向最弱模型。LLM 对"建议"式规则的遵循率远于"MUST"，尤其在长上下文和弱模型上

7\. "成功"要严格定义。投递成功 ≠ 归档成功，无报错 ≠ 有产出，Agent 说"正常" ≠ 真正正常

8\. Agent 不回复是常态——准备好 Task Watcher 和重试机制

9\. shared-context/ 是协作基石——sessions\_send不可靠（超时、重复），关键数据走文件才可追溯

10\. 每加一个 Agent 都需要半天到一天的调试。急于求成 = 浪费更多时间在排查上

装好不难，跑通也不难。难的是：让 6 个 Agent 在没有你盯着的时候也能稳定产出、自我修正、协作不打架。这不是 prompt 问题，是系统工程问题。

从理解原理开始

如果你想先理解 OpenClaw 的核心原理再动手，可以看看——~2700 行 Python 实现了 OpenClaw（43 万行 TypeScript）的 11 个核心架构模式：Gateway Hub-and-Spoke、Workspace 契约文件、Agent Loop、Skills 触发、Compaction 上下文管理、Multi-Agent & Spawn、Heartbeat、Cron、Hooks EventBus、自动反思。不需要看原始代码就能理解"为什么要这么设计"。

附录：技术栈快速参考

> 以下是正文中提到的技术组件的集中索引，方便快速查阅。详细说明见正文对应章节。

LLM 模型分层

| 任务类型 | 模型 | 成本 |
| --- | --- | --- |
| 主对话 / 反思 / 圆桌 | GPT-5.4 |  |
| ACP 编码任务 | K2.5 / GPT-5.4 | $ |
| Cron 日常任务 | Qwen3.5+ / K2.5 | $ |
| 心跳 / 健康检查 | Ollama qwen3:8b | 免费 |

Fallback 链：gpt-5.4 → k2.5 → qwen3.5-plus → ollama/qwen3:8b

核心 Harness 配置

```json
{  "compaction": { "mode": "safeguard", "memoryFlush": { "softThresholdTokens": 40000 } },  "contextPruning": { "mode": "cache-ttl", "ttl": "6h", "keepLastAssistants": 3 },  "session": { "reset": { "atHour": 5, "idleMinutes": 30 }, "maintenance": { "pruneAfter": "7d", "maxDiskBytes": 104857600 } },  "acp": { "maxConcurrentSessions": 6, "ttlMinutes": 120 }}
```

数据源

| 市场 | 数据源 | 覆盖 |
| --- | --- | --- |
| A 股 | AKShare + TuShare Pro | 实时行情 + 历史 + 财务 + 龙虎榜 + 北向 |
| 美股/港股 | yfinance + Finnhub | 行情 + 新闻 + 基本面 |
| 信息采集 |  | 搜索 + 新闻 + 热点 + 论文 |
| 浏览器 | agent-browser (Playwright) | JS 渲染页面（X/Twitter、雪球等） |

部署配置

| 组件 | 配置 |
| --- | --- |
| 硬件 | Mac 本地 7×24 |
| 进程守护 | `launchctl`  \+ `ThrottleInterval=10` |
| 自愈 | 2086 行脚本（heartbeat-guardian / check\_cron\_health / memory\_maintenance） |
| 备份 | 每日 03:00 全量备份 |
| 监控 | Zoe 3 次/天巡检 + 系统 crontab 15 分钟健康检查 |
| 知识归档 | Obsidian Vault + obsidian-livesync |

继续滑动看下一个

阿里云开发者

向上滑动看下一个

![kimi](data:image/png;base64,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)
