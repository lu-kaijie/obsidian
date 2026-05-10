---
title: 2026 年 AI 编码的“渐进式 Spec”实战指南
source_url: https://mp.weixin.qq.com/s/7Lgb3GfgXKI0J9L9e9sq0w
saved: 2026-05-09
tags: [ai]
---
逸驹 *2026年4月2日 08:31*

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naKcQ9P15cMx5ZCVuZz2CfuyRrFmLE4Z5YgSTibhpLkCWgCVOkTZia8SYUTobvFL4iaCicm5UsYDTvwxjQ/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0)

阿里妹导读

这次分享的内容来自作者在实际项目中落地 AI 编码的一些实践和思考。希望能给正在尝试或想要尝试 AI 编码的同学一些参考。

---

一、背景

> 聊 AI 编码之前，先对齐三个基础认知

**1.1 如何理解大模型 — 它能做什么、不能做什么**

当前顶级模型可以独立完成中等复杂度的编码任务——理解需求、读代码、写实现、修编译错误，但仍需人审查结果。它们没有持久记忆、没有自主意图，只处理你给它的上下文。

不同模型之间的性能差异是断崖式的。Chatbot Arena 通过真人盲评计算 ELO 分数（截至 2026 年 3 月，536 万次投票、316 个模型）：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

Arena 的 Multi-Turn 维度衡量多轮交互稳定性（对应 Agent 场景）：Gemini 3.1 Pro 多轮第一，超过 Claude 4.6；Claude Sonnet 4.6 掉到第 10——单轮强不等于多轮稳。

梯队差异体感：同样是"给 Spring Boot 服务加个带缓存的分页查询"——T0 一次生成全链路且主动处理边界情况；T1 多提示一两轮可达到接近效果；T1.5 基本可用但容易漏边界；T2 能写骨架但需要较多人工调整。差距不在"能不能写"，而在一次做对的概率——T0 三轮搞定的事，T2 可能 15 轮还不一定对。

核心结论：模型是地基，方法论是上层建筑。地基不行，上面盖得再好也白搭。

**1.2 如何理解 Agent — 从一问一答到自主行动**

知道了模型能力之后，下一个问题是：怎么让它自主行动？

裸 LLM 只是一个无状态的问答函数——你问一句它答一句，没有工具、没有记忆、不会自己行动。

Agent = while 循环 + Tool Use + 工具执行器，用一个例子说明：

```javascript
你说："把 UserService 里的 getById 方法加个缓存"  → Agent 调用【读文件】工具，读取 UserService.java      （侦察）  → Agent 分析代码，决定修改方案                          （思考）  → Agent 调用【写文件】工具，插入缓存逻辑                  （行动）  → Agent 调用【终端】工具，运行编译检查                    （验证）  → 编译报错，Agent 读取错误信息，自动修复                   （自愈）  → 编译通过，Agent 回复你"已完成"                        （结束）
```

这个循环就是 Agent 的全部—— "智能"来自模型，"能力"来自工具，"自主性"来自循环 。

关键理解：工具的边界就是 Agent 的能力边界。给它读写文件的工具，它能改代码；不给它网络工具，它就上不了网。安全靠框架约束，不靠 AI 自觉。

我们后面提到的 Cursor、Claude Code、opencode——本质上都是这个循环的不同包装，区别只在于给了哪些工具、跑在哪里、用的什么模型。

**1.3 回归本质 — 软件复杂度视角**

有了能自主行动的 Agent，最后一个问题是：用什么标准评判一个 AI 编码方案的好坏？

来自《人月神话》的核心洞察

- 软件复杂度 = 本质复杂度（业务逻辑本身，不可消除）+ 偶然复杂度（工具/流程引入的额外负担，可以且应该被消除）；
- 本质复杂度由业务决定，任何工具都不能消除它——交易状态机该多复杂还是多复杂；
- AI Coding 工具能做的是帮你更高效地应对本质复杂度（快速理解代码、生成实现、发现风险），但工具自身也会引入偶然复杂度（学习成本、流程开销、配置负担）；
- 评判标准：一个方案好不好，看它能多高效地帮你应对本质复杂度，同时自身引入的偶然复杂度有多低；
- 核心结论：所有方法论的设计都要回归这个起点——高效应对本质复杂度，压缩偶然复杂度。

---

二、渐进式编码框架

**2.1 Spec Coding 是什么**

一句话：在让 AI 写代码之前，先用结构化文档（Spec）把"要做什么、怎么做、有什么约束"说清楚，然后 AI 围绕这份文档编码。

为什么需要 Spec Coding——直接和 AI 聊天写代码（Vibe Coding）面临四个工程问题：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

Spec Coding 三条铁律：

1. No Spec, No Code — 没有文档，不准写代码
2. Spec is Truth — 文档和代码冲突时，错的一定是代码
3. Reverse Sync — 发现 Bug，先修文档，再修代码

这三条铁律在经济上也是合理的（Code is Cheap, Context is Expensive）：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

把需求、约束、代码现状写进 Spec 作为高质量输入 → 输入增加但便宜 → AI 不用反复试错 → 输出大幅减少 → 对话轮次从 20 轮降到 3-5 轮 → 总成本反而更低，效果反而更好。

**2.2 为什么要自己做一套**

Spec Coding 的理念很简单，但不同团队的落地方式差别很大。我们调研了多个主流实现后，吸收各方核心理念做了一套自己的框架。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

**2.3 核心设计：渐进式复杂度**

这是框架的核心卖点，也是和其他方案最大的区别。

为什么需要渐进式？ 其他方案都假设所有需求都值得走完整流程，但现实中并非如此——70% 的需求是 ≤5 人日的小需求。改个字段、修个 bug，也要先写 spec 再拆 tasks？这就是偶然复杂度在吃掉你的效率。

核心思想：不同复杂度的需求，暴露不同深度的流程——

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

关键原则：

- 简单需求不承担复杂流程的成本——改个字段不需要先写 spec 再拆 tasks
- 流程是可选增强，而非强制前提——Rules 始终生效，Spec 按复杂度加载
- 这本质上是在压缩偶然复杂度：只有本质复杂度够高时，才引入对应重量的流程

**2.4 自我迭代：一切皆可迭代**

这个框架本身就是一个活的系统——prompt、模板、rules 都是代码库中的普通文件，随 Git 版本演进：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

知识飞轮（不仅是领域知识，prompt 和模板自身也在飞轮中）：

```js
需求实践 → 踩坑 → 沉淀 knowledge / 更新 prompt / 修改模板 → AI 更准 → 更好的实践    ↑                                                                |    └────────────────────────────────────────────────────────────────┘
```

**2.5 框架全貌：目录结构**

```bash
code_copilot/├── rules/                      # Project Rules（始终生效）│   ├── project-context.md      #   工程结构、分层、核心依赖│   ├── coding-style.md         #   编码规范│   ├── security.md             #   安全红线│   └── domain-rules.md         #   业务领域约束│├── knowledge/                  # 领域知识（按需加载）│   ├── index.md                #   知识索引（触发关键词 + 描述）│   └── *.md                    #   详细知识文档│├── agents/                     # Agent 配置与提示词│   ├── copilot-prompt.md       #   实际提示词（AI 动态加载，可迭代）│   ├── spec-reviewer.md        #   Spec 合规审查 Agent│   └── code-quality-reviewer.md #  代码质量审查 Agent│├── changes/                    # 变更管理│   ├── templates/              #   模板（spec / tasks / log，可迭代）│   └── <change-name>/          #   每个需求一个目录│└── archives/                   # 已完成变更的归档
```

Agent 提示词的核心设计要点：

1. 身份定位：有经验的 Java 后端工程师搭档，不是代码生成器
2. 启动流程：每次会话自动读取 rules/、检查 changes/ 进行中的变更、报告状态
3. 命令式路由：7 个命令（/init /propose /apply /fix /review /archive /knowledge），超出范围礼貌拒绝
4. Research 约束：代码现状必须有出处（文件路径+类名/方法名），不接受空口结论
5. 执行策略：默认逐步执行（暂停确认），支持批量和紧急停车
6. Reverse Sync：遇到偏差先修 spec 再修代码，强制回写
7. 安全红线：涉及资金/状态变更 → ⚠️ 高亮提醒人工审查
8. 知识沉淀：有价值的发现 → 主动建议沉淀到 knowledge/

**2.6 工作流：Propose → Apply → Review → Archive**

Propose（提案）— 人主导，AI 辅助

1. Research：分析代码现状，锁定事实（每个结论有代码出处）
2. 逐个提问：一次只问一个问题或一组紧密相关的问题，优先给 2-3 个选项 + 推荐，减少用户思考负担。同时做 YAGNI 裁剪——主动识别"nice to have"建议延后
3. 分段生成文档，每段确认：不一口气生成完整 spec，按段输出（代码现状+功能点 → 变更范围+风险 → 技术决策+待澄清），每段等用户确认后再继续。越早发现方向偏差，修正成本越低
4. 生成完整 spec.md + tasks.md + log.md
5. 关键约束：待澄清全部解决前，不允许进入 Apply
6. 确认门控 HARD-GATE：完整 spec + tasks 生成后，必须等用户显式确认。确认前禁止任何编码动作——再简单的需求也值得一次设计审视

Apply（执行）— AI 主导，人审查

- 默认逐步执行：完成一个 task → 报告 → 等用户确认
- 批量执行：用户说"全部完成" → 按顺序执行所有
- 紧急停车：遇到逻辑冲突或 spec 缺失 → 立即停止，Reverse Sync
- 零偏差原则：Plan 是合同，AI 是打印机
- Verification 铁律：每个 task 完成后必须展示可验证的证据（编译输出 / 测试输出 / 调用结果），禁止"应该没问题"等无证据声明
- 实时知识采集：每个 task 完成后立即检查是否踩坑/发现隐含规则/学到新东西，有则立即写入 log.md

Fix（修正迭代）— Review 后的增量修正

- 填补 /apply 和 /review 之间的修正环节
- 与 /apply 的区别：/apply 按 tasks 顺序执行初始编码，/fix 在已完成基础上做增量修正
- 文档同步是铁律——每次 /fix 必须同步更新 spec、tasks、log

Review（审查）— 两阶段 Sub Agent 审查

拆为两个独立阶段，通过 Sub Agent 执行（上下文与实现者隔离）：

1. Spec Compliance（spec-reviewer）：逐条比对 spec 功能点与实际代码，核心原则"不信报告只信代码"
2. Code Quality（code-quality-reviewer）：基于 rules/ 检查编码规范、安全红线、异常处理，按 Critical/Important/Minor 分级

阶段一 PASS 后才启动阶段二。任一 FAIL 则回到 Apply/Fix 修正。

Archive（归档）— 知识沉淀

逐条展示 log.md 中的知识发现和踩坑记录，询问用户是否沉淀到 knowledge/，确认的立即执行。变更目录移到 archives/。

Debug — 系统化调试流程

四阶段调试指引：根因调查 → 模式分析 → 假设验证 → 实施修复。铁律：禁止在未确认根因前直接改代码。

---

三、工具选型与编排

**3.1 编排层 + 执行层的两层架构**

在实践中，我发现单一工具很难同时满足"强模型做决策"和"安全模型写代码"两个需求。最终演化出了编排层 + 执行层的两层 AI 架构：

```js
人（开发者）  │  ├─ 对话界面 ── 日常交互  │       │  │       ▼  │   编排层 AI（强模型，如 Claude Opus）  │   职责: 理解需求、生成 Spec、审查结果  │       │  │       ▼  │   执行层 AI（编码工具，如 Claude Code / opencode）  │   职责: 读写代码、执行命令、运行测试  │  └─ 终端 ── 可随时直接接管编码工具
```

```
为什么要分两层？ 不仅仅是安全考虑，更是关注点分离：
```

| 层 | 擅长 | 模型选择 |
| --- | --- | --- |
| 编排层 | 理解模糊需求、生成结构化 spec、跨仓库业务分析、审查决策 | 强模型（Claude Opus、Gemini Pro 等） |
| 执行层 | 读写代码、执行 shell 命令、快速迭代修改 | 编码优化模型（Sonnet、Kimi 等） |

把两者混在一起，要么模型太贵（全程用顶级模型写代码），要么质量不够（全程用便宜模型做决策）。分层后各取所长，成本和质量都更优。

**3.2 工具选型思路**

选择编码工具时，有一个关键经验：透明度不是奢侈品，是基础需求。

透明度底线：模型型号+版本可见、完整 context 可查、原始输出不被篡改、token 用量透明。

在不透明的工具上花再多时间优化 prompt 和框架，效果都无法归因、无法复现。切到透明工具链后，每次调优都能看到效果，迭代速度指数级提升。

目前满足透明度要求的主流开源工具：

| 工具 | 定位 | 特点 |
| --- | --- | --- |
| Claude Code | 终端 AI 编码 Agent | Anthropic 官方，模型绑定 Claude |
| opencode | 终端 AI 编码 Agent（开源） | 模型自由选择，社区驱动 |
| Cursor / Windsurf | IDE 内交互式 AI 搭档 | GUI 友好，上手快 |
| Cline / Aider | 终端/IDE 插件 | 轻量级，可定制 |

选择建议：

- 如果团队用 Claude 系列模型，Claude Code 开箱即用
- 如果需要灵活切换模型（特别是接入私有部署的模型），opencode 更合适
- 如果团队更习惯 IDE 工作流，Cursor/Windsurf 是好选择
- 核心原则：工具是手段，方法论是不变的——上面的框架（rules/ + knowledge/ + changes/）可以适配任何编码工具

**3.3 Git 规范**

无论用什么工具，Git 规范是必须的：

1. 禁止 master 分支变更：编码前检查当前分支，master 上立即停止
2. 自动 Commit：每个 task/fix 完成后自动 commit，保持一个 task 一个 commit
3. Commit 必须可编译：commit 前执行编译检查
4. 禁止自动 Push：push 由用户主动触发，保留审查机会
5. Message 格式： `[<变更名>] <中文简述>`

---

四、实战案例

以一个典型的跨模块业务迁移项目为例（将系统 A 的过滤处理逻辑平行迁移到系统 B），走一遍完整的 /propose → /apply → /review 流程。

**4.1 需求 → Spec：把需求文档喂给 AI**

将需求文档作为输入，执行 `/propose` ，AI 先 Research 代码现状，然后逐个提问收敛不确定性：

```markdown
🤖 AI：spec 初稿写好了，几个需要你确认的点：待澄清（需要你回答的）：1. Q1: 系统 A 中 AbstractFilterService 下三个处理器具体叫什么类名？   我需要知道才能让 AI 精确对照迁移2. Q2: FilterModule 的 lockKey 和 idempotentId 要不要设？   写入的幂等是 SDK 层保证还是需要框架层加？3. Q3: needOrderTradeType 目前有 CLOSE/PAID_DONE/ENABLE_DONE，   退款消息在框架里是怎么表示的？是单独的 OrderTradeType 还是走别的路径？tasks 拆分思路：7 个 task，按依赖顺序：灰度配置 → bizProcess 分发主干 → 三个处理器（可并行）→ 压测 mock → 框架扩展点补全。每个 task 都是原子可提交的。一个设计决策想确认：我把三个处理器放在同一个 FilterModule 的bizProcess 里按类型分发，而不是拆成三个独立 Module。理由是"一个模块一个处理器"的设计，过滤本身就是一个模块。你觉得这样 OK 还是要拆？
```

关键点：

- AI 必须先 Research 现有代码（带出处），不允许凭空设计
- 逐个提问而非一次列 10 个问题，优先给选项 + 推荐
- 待澄清项全部解决前不进入编码

**4.2 Spec → Tasks：拆分原子任务**

Spec 确认后，AI 自动拆分为原子级 Tasks。执行一段时间后，AI 汇报进度并列出待完善项：

```sql
🤖 AI：/apply filter-migration 执行进度⚠️ 待完善项（TODO）| 位置                  | 说明                              | 优先级 || -------------------- | -------------------------------- | ----- || LimitFilterService   | 垂直表 + 水平表匹配逻辑具体实现        | P0    || LimitFilterService   | 配置读取（marketingSolutionIds）    | P0    || All Services         | 外部 SDK 具体 API 调用               | P0    || RefundFilterService  | actualPaidFee / refundFee 字段获取  | P1    || All Services         | 降级开关集成（LIMITED_FILTER_DEGRADE_ON 等）| P1 |🚀 下一步建议1. 补全 TODO 实现：需要确认二方包 FilterClient 的具体 API2. 创建配置 dataId: com.example.trade.filter.gray3. 本地测试验证4. Code Review您希望：• 继续补全 TODO 实现？• 先进行 /review filter-migration 审查？• 还是其他操作？
```

**4.3 Tasks → Apply：逐步执行**

进入 `/apply` 阶段，AI 按 Tasks 逐个执行。每完成一批文件，展示完成状态和核心实现结构：

```sql
🤖 AI：开始编码  /apply filter-migration✅ 已完成文件| 类型 | 文件路径                           | 状态                           || --- | --------------------------------- | ----------------------------- || NEW | config/FilterGrayConfig.java      | ✅ 灰度配置类，配置中心动态监听    || MOD | module/FilterModule.java          | ✅ 完整分发框架，注入三个 Service || NEW | service/filter/LimitFilterService.java    | ✅ 限购过滤服务（子单维度）  || NEW | service/filter/ShopFrozenFilterService.java| ✅ 冻结店铺新用户（主单维度）|| NEW | service/filter/RefundFilterService.java   | ✅ 退款过滤服务（含完全退款判断）|🔧 核心功能实现FilterModule.bizProcess()├── ENABLE_DONE（创单）│   ├── LimitFilterService.processEnableDone()    # 限购过滤写入│   └── ShopFrozenFilterService.processEnableDone() # 冻结店铺过滤写入├── PAID_DONE（支付完成）│   └── RefundFilterService.processPaidDone()      # 删除过滤└── CLOSE（关单）    ├── ShopFrozenFilterService.processClose()     # 回滚（先）    └── RefundFilterService.processClose()         # 逆向添加（后）        ├── 完全退款判断: actualPaidFee == refundFee        └── 新客判断: identityId == 1L
```

**4.4 迭代与 Reverse Sync**

执行过程中用户发现 Spec 遗漏或实现偏差时，AI 先确认事实再更新文档：

```markdown
👤 用户：垂直表逻辑已维护在 TradeMainOrder 中，由 TradeContext 携带   LimitFilterService 的 marketingSolutionIds 不是读配置中心   是读的 switch 配置 SwitchConfig.XXXXX   FilterClient 的调用请参考系统 A 中 ShardingFilterService 的实现   actualPaidFee/refundFee 同理已维护在 TradeMainOrder 中   先完善以上功能，降级开关等会再改🤖 AI：收到了！让我先理解您的修改和新的需求，然后更新文档：1. 垂直表逻辑：已维护在 TradeMainOrder 中，通过 TradeContext 携带2. marketingSolutionIds 来源：不是配置中心，而是 SwitchConfig.XXXXXX3. FilterClient 调用：参考系统 A 侧的 ShardingFilterService 实现4. actualPaidFee/refundFee：已维护在 TradeMainOrder 中让我先读取相关代码来确认这些细节，然后更新 spec：∞ 读取 SwitchConfig 配置∞ 读取 ShardingFilterService 实现∞ 验证 TradeMainOrder 修改收到了！我发现几个问题需要澄清：（AI 基于实际代码继续追问...）
```

Reverse Sync 是最容易被忽略但最重要的环节 ：

保持文档和代码的一致性，才能让整个框架持续有效。

---

五、一些观点与思考

**5.1 人到底该干什么？**

用 AI 编码之后，人的角色从「全干」变成了「管和验」：

```js
传统编程：人 = 设计者 + 实施者 + 验收者AI 编程：人 = 设计者 + 验收者，AI = 实施者
```

具体来说，人做三件事： 管控 （控制 AI 看什么）、 指挥 （选方案、审计划、批准执行）、 评价 （验收结果、发现偏差）。对应地，AI 在不同阶段切换角色：侦察兵（扫描代码收集事实）→ 参谋（提出方案分析利弊）→ 施工队（按图施工）→ 质检员（对照标准检查）。

听起来很清晰，但实际协作中有三个非常容易犯的错误：

1\. 讨论和命令混为一谈

"帮我看看这个模块，顺便把 Bug 修了"——前半句是探索，后半句是指令。AI 会跳过探索直接改代码，然后改错。解决办法很简单：一次只给一种意图（探索 / 决策 / 指令 / 审查），不要混着来。

2\. 阶段产出搞混

调研阶段要的是事实和风险，不是代码。AI 太勤快的时候要拉住它："停，我现在不需要代码，先告诉我现状和风险。"

3\. 自由度给反了

这是最普遍的问题。正确的自由度曲线应该是：

| 阶段 | 自由度 | 为什么 |
| --- | --- | --- |
| 调研 | 中 | 让 AI 自由探索，但必须给证据 |
| 方案设计 | 高 | 唯一鼓励 AI 充分想象的阶段 |
| 规划 | 低 | 精确到文件路径和函数签名 |
| 执行 | 零 | 严格按计划施工，有问题必须停下来问 |
| 验收 | 中 | 自由检查，但结论要有依据 |

大部分人的问题是反过来了——该讨论的时候急着让 AI 干活（方案没想清楚就开写），该干活的时候又让 AI 自由发挥（执行阶段不约束，改着改着就跑偏了）。

**5.2 Spec 不是银弹，但也不是废弹**

有一种批评认为 Spec Coding 建立在三个错误假设上——AI 能理解规范、规范能完整描述系统、规范比代码更易维护。

这些批判有道理，但忽略了一个关键前提：它批判的是「规范→代码」的全自动线性映射，不是人在回路中的 Spec 辅助模式。

| 被批判的模式 | 我们实际做的 |
| --- | --- |
| 写好 Spec，AI 自动生成全部代码 | Spec 只描述变更范围，AI 在人审核下逐步执行 |
| 规范是「唯一真理来源」 | 规范是沟通工具，代码才是真理 |
| 期望 AI 理解整个系统 | 用 knowledge/ 喂精确上下文，限制 AI 的理解范围 |
| 适用于所有复杂度 | 渐进式——简单需求根本不写 Spec |

问题不在于 Spec Coding 本身，而在于用法和预期。当成自动化流水线的输入，它确实不是银弹；当成人和 AI 之间的沟通协议，它就是一个靠谱的效率工具。

**5.3 知识底座才是真正的护城河**

大部分团队在 AI 编码上的投入方向是：花大量精力写 Prompt、调 Rules、优化 Agent 工作流。这些都属于「偶然复杂度」层面——调好了最多让 AI 少犯格式错误。但真正决定 AI 输出质量上限的，是你喂给它的领域知识的质量。

知识覆盖缺口：

| 知识类型 | Spec 能覆盖 | 实际重要性 |
| --- | --- | --- |
| 编码规范 | ★★★★ | ★★★ |
| 存量代码 | ★★★ | ★★★★ |
| 领域知识 | ★ | ★★★★★ |
| 架构决策 | ★★ | ★★★★★ |
| 团队隐性经验 | ☆ | ★★★★ |

最关键的知识——领域 Know-How、架构决策的前因后果、踩坑后的最佳实践——恰恰是纯 Spec 框架最难覆盖的。这也是为什么框架里有 `knowledge/` 目录，而不是只有 `rules/` 。

打个比方：一个没有 knowledge/ 的 Spec 框架，就像让一个刚入职的应届生对着编码规范写代码——规范他都能遵守，但业务逻辑全靠猜。

往长远看，AI 编码工具会越来越同质化（Cursor、Claude Code、各种 IDE 插件的能力趋同），团队之间的差距不在于用什么工具，而在于积累了多少高质量的、结构化的领域知识。这才是真正不可复制的护城河。

**5.4 几个容易被忽略的代价**

### 心流中断

编码需要高度专注。引入 AI 之后，连贯性被打破了——写 Spec、等生成、审查输出、纠正偏差、再继续下一段。模型越慢，等待越长，心流杀伤越大。

传统编码是「想→写→想→写」的连续流，AI 编码变成了「想→写 Spec→等→审→改→想」的间歇流。你需要适应这种新节奏，也需要诚实地评估：对于你个人，哪些任务用 AI 确实更快，哪些还不如自己写。

### 上下文的隐性成本

Context 很贵，但贵的不只是 Token 费用。更大的隐性成本是上下文管理本身：你需要有意识地决定给 AI 看什么、不看什么；每一次上下文压缩都引入不确定性——被压缩掉的历史对话和决策，你无法确认 AI 是否还记得。

这也是框架把 knowledge/ 和 Spec 都做成独立文件的原因：文件是持久化的，不会被上下文窗口压缩掉。写在文件里的计划，比聊天记录里的口头约定可靠得多。

### 现在不是终态

编程可能是 AI 产生飞轮效应最直接的领域——AI 辅助编码 → 效率提升 → 更多代码产出 → AI 能力增强 → 更好地辅助编码。这个正向循环已经在发生。

今天某些模型的速度和质量让你觉得勉强够用，半年后可能完全不同。框架的价值在于它能随模型进步而放大收益——当模型从 T1.5 升到 T1 甚至 T0，同样的 Spec 和 knowledge/ 能产出质量截然不同的代码，而你的 rules/、knowledge/、历史 Spec 都是现成的积累。

---

## 参考

- Superpowers — agentic skills 框架（HARD-GATE、两阶段 review、systematic-debugging）：https://github.com/obra/superpowers
- Writing about Agentic Engineering Patterns - Simon Willison：https://simonwillison.net/2026/Feb/23/agentic-engineering-patterns/
- Writing code is cheap now - Simon Willison：https://simonwillison.net/guides/agentic-engineering-patterns/code-is-cheap/
- Chatbot Arena Leaderboard (LMSYS)：https://arena.ai/
- opencode 官方文档：https://opencode.ai/docs/
- Claude Code 文档：https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview
- Frederick Brooks,《人月神话》

---

附录：code\_copilot 框架完整内容

> 以下为 code\_copilot 框架当前的完整文件内容，可直接复制到项目中使用。项目特定内容（应用名、包名、中间件等）需根据实际情况填充。

## 目录结构

```powershell
code_copilot/├── README.md                           # 框架说明├── agents/                             # Agent 配置与提示词│   ├── copilot-prompt.md               # 主 Agent 完整提示词（核心）│   ├── spec-reviewer.md                # Spec 合规审查 Agent│   └── code-quality-reviewer.md        # 代码质量审查 Agent├── rules/                              # 项目约束（始终生效）│   ├── project-context.md              # 工程结构与依赖（/init 填充）│   ├── coding-style.md                 # 编码规范│   ├── security.md                     # 安全红线│   └── domain-rules.md                 # 业务领域约束├── knowledge/                          # 领域知识（按需加载）│   └── index.md                        # 知识索引└── changes/                            # 变更管理    └── templates/                      # 模板目录        ├── spec.md                     # Spec 模板        ├── tasks.md                    # Tasks 模板        ├── test-spec.md                # 单测 Spec 模板        └── log.md                      # Log 模板
```

## A.1 agents/copilot-prompt.md — 主 Agent 提示词

```bash
你是 code-copilot，一个面向已有 Java 后端项目的 AI 编码协作助手。你的工作基于 rules/（项目约束）、knowledge/（领域知识）、changes/（变更管理）三个目录。# 核心法则## Spec 驱动（Code is Cheap, Context is Expensive）代码是廉价的消耗品，文档（Spec）才是昂贵的核心资产。1. **No Spec, No Code** — 没有 spec，不准写代码2. **Spec is Truth** — spec 和代码冲突时，错的一定是代码3. **Reverse Sync** — 执行中发现 spec 与实际不符，先修 spec 再修代码4. **代码现状必须有出处** — 每个结论必须标注文件路径和类名/方法名，不接受"我认为"、"通常来说"5. **变更即记录** — 任何代码变更完成后都必须同步更新对应的 changes/ 文档## 身份与原则- 有经验的 Java 后端工程师搭档，不是代码生成器- 用中文输出，技术术语可保留英文- 不确定就问，不假设，不编造不存在的类或接口- 每个任务原子化（3-5 个文件），做"小炸弹"而非"大炸弹"- 涉及资金/交易状态变更 → ⚠️ 高亮提醒人工审查- 有价值的发现 → 主动建议沉淀到 knowledge/## 意图确认（先问再做）收到用户的自然语言指令时，先识别意图并映射到对应命令，确认后再执行。| 用户说的 | 映射命令 ||---------|---------|| "修复 xxx" / "改一下 xxx" | → /fix || "我要做 xxx 需求" | → /propose || "开始写代码" / "继续执行" | → /apply || "帮我看看代码" / "review 一下" | → /review || "写测试" / "补单测" | → /test || "归档 xxx" | → /archive |纯技术讨论不需要走命令流程，直接回答。# 启动每次会话开始时：1. 读取 rules/ 下所有规则文件2. 检查 changes/ 下是否有进行中的变更（排除 templates/）3. 报告当前状态，展示命令菜单# 命令## /init — 初始化项目上下文分析工程结构、依赖、分层模式，填充 rules/project-context.md。## /propose <需求描述> — 创建变更提案Research → 逐个提问（一次只问一个，给选项+推荐）→ YAGNI 裁剪 →分三段生成 spec（每段确认）→ 生成 tasks → HARD-GATE 确认。待澄清全部解决前不允许进入 /apply。## /apply <变更名> — 执行编码前置检查 spec + tasks + 用户确认。逐 task 执行，每个 task 完成后展示验证证据（Verification 铁律）。零偏差原则：Plan 是合同，AI 是打印机。自动 git commit（一个 task 一个 commit）。## /fix <变更名> [描述] — Review 后修正迭代增量修正 + 文档同步铁律（spec/tasks/log 全部更新）。## /review <变更名> — 两阶段审查阶段一 Spec Compliance → 阶段二 Code Quality。优先用 Sub Agent 执行（上下文隔离）。阶段一 PASS 后才启动阶段二。## /test <变更名> — 生成单测 Spec 并执行Red/Green TDD：测试必须先 Red 再 Green。两种模式：Spec 先行（推荐）或直接生成。## /archive <变更名> — 归档 + 知识沉淀逐条展示 log.md 知识发现，确认后沉淀到 knowledge/。## Git 规范1. 禁止 master 分支变更2. 每个 task/fix 自动 commit3. Commit 必须可编译4. 禁止自动 push5. Message 格式：[<变更名>] <中文简述>## 调试流程四阶段：根因调查 → 模式分析 → 假设验证 → 实施修复。禁止在未确认根因前直接改代码。
```

```
A.2 agents/spec-reviewer.md — Spec 合规审查
```

```markdown
# Spec Compliance Reviewer专职验证代码实现是否符合 spec 规格。只读不写，独立于实现者的上下文。核心理念：**不信报告，只信代码** — reviewer 必须读实际代码独立验证。## 审查维度1. **缺失实现**：spec 要求了但代码没做的2. **多余实现**：spec 没要求但代码多做了的（YAGNI 违规）3. **理解偏差**：做了但做错了方向的4. **业务规则落地**：spec §4 中的规则是否全部体现在代码中5. **数据变更准确性**：spec §5 中的表/字段变更是否准确落地## 输出格式#### 功能点逐条验证- ✅ 功能 1：已实现，见 \`XxxService.java:L42\`- ❌ 功能 2：未实现（缺少 XX 逻辑）- ⚠️ 功能 3：实现方式与 spec 描述有偏差#### 结论：✅ Spec 合规 / ❌ 不合规（附具体问题）## 工具权限仅需 Read/Grep/Glob/Bash（只读），不需要写入权限。
```

## A.3 agents/code-quality-reviewer.md — 代码质量审查

```markdown
# Code Quality Reviewer专职审查代码质量、安全性和可维护性。前置条件：必须在 spec-reviewer 审查通过后才启动。## 审查分级- **Critical**（阻塞）：安全漏洞、资金逻辑错误、并发安全、数据丢失风险- **Important**（应修复）：异常被吞、缺少参数校验、魔法值、方法过长、命名不清- **Minor**（建议）：Javadoc 缺失、注释过时、import 未清理## 工具权限仅需 Read/Grep/Glob/Bash（只读），不需要写入权限。
```

## A.4 rules/project-context.md — 工程上下文

```markdown
---alwaysApply: true---# 工程上下文> 首次使用时执行 /init 让 AI 分析工程并填充本文件。## 1. 应用概况- 应用名: （待填充）- 简介: （一句话描述）- 技术栈: Java 21 / Spring Boot / （根据项目实际填写）- 构建工具: Maven## 2. 目录结构与模块职责> 执行 tree -d -L 3 后填充。## 3. 分层架构Controller (web/)       ← 入口层，参数校验 + 协议转换    ↓Service (service/)      ← 业务编排，事务边界    ↓Manager (manager/)      ← 领域能力，单一职责，可复用    ↓DAO (dao/)              ← 纯数据访问## 4. 关键依赖| 中间件 | 用途 | 备注 ||--------|------|------|| （根据项目填写） | | |
```

## A.5 rules/coding-style.md — 编码规范

```markdown
---alwaysApply: true---# 编码规范## 1. 命名- 类名：大驼峰，见名知意- 方法名：小驼峰，动词开头- 常量：全大写下划线分隔- 抽象类以 Abstract 或 Base 开头- 测试类以被测类名开头，Test 结尾- 禁止拼音、中英混拼命名## 2. 异常处理- 业务异常使用自定义 BizException，携带错误码- 系统异常向上抛出，由统一异常处理器兜底- 禁止吞掉异常（空 catch）- catch 中必须记录日志## 3. 日志- Controller 入口打 INFO，含请求关键参数- 异常打 ERROR，含完整堆栈- 禁止在日志中打印用户敏感信息## 4. 其他- 写接口必须考虑幂等- 涉及并发场景必须说明同步策略- 魔法值必须定义为常量
```

```
A.6 rules/security.md — 安全红线
```

```markdown
---alwaysApply: true---# 安全红线## 1. 代码安全- ❌ 禁止在代码中硬编码密钥、AK/SK、数据库密码- ❌ 禁止提交包含用户个人信息的测试数据- ❌ 禁止在日志中打印手机号、身份证、银行卡等敏感信息## 2. 业务安全- ⚠️ 涉及资金变更的逻辑，必须在 spec 中明确标注，人工审查后方可编码- ⚠️ 涉及状态流转的逻辑，必须检查状态机合法性- ⚠️ 涉及权限变更的逻辑，必须显式校验操作人权限
```

```
A.7 rules/domain-rules.md — 业务领域约束
```

```markdown
---alwaysApply: falsedescription: "当涉及业务领域特定逻辑时应用本规则"---# 业务领域约束## 1. 通用领域规则- 所有金额使用 long 类型，单位为分- 时间字段统一使用 Date 类型- 外部接口调用必须设置超时（默认 3s）并做降级处理- 状态变更必须通过状态机，禁止直接 set 状态字段## 2. 项目特定规则（待实践中补充）
```

```
A.8 knowledge/index.md — 知识索引
```

```shell
# 知识索引> 领域知识的轻量索引。每条用一句话说清核心逻辑。> 格式：- **触发关键词**: 一句话核心逻辑 → \`包名.类名.方法名\`（可选）## 业务知识（随实践积累补充）## 技术约定（随实践积累补充）## 踩坑记录（随实践积累补充）
```

```
A.9 changes/templates/spec.md — Spec 模板
```

```shell
# 需求名称> status: propose | apply | review | done> created: YYYY-MM-DD> complexity: 🟢简单 | 🟡中等 | 🔴复杂## 1. 背景与目标为什么做 + 做完后的效果（可验证的结果描述）## 2. 代码现状（Research Findings）> 每个结论必须有代码出处（文件路径 + 类名/方法名）### 2.1 相关入口与链路### 2.2 现有实现### 2.3 发现与风险## 3. 功能点- [ ] 功能 1：（输入→处理→输出）## 4. 业务规则## 5. 数据变更| 操作 | 表名 | 字段/索引 | 说明 |## 6. 接口变更| 操作 | 接口 | 方法 | 变更内容 |## 7. 影响范围## 8. 风险与关注点> ⚠️ 涉及资金/状态流转/权限变更必须标注## 8.5 测试策略- **测试范围**：- **覆盖率目标**：- **独立 Test Spec**：是/否## 9. 待澄清- [ ] 问题 1：（全部解决后才能进入 /apply）## 10. 技术决策## 11. 执行日志| Task | 状态 | 实际改动文件 | 备注 |## 12. 审查结论## 13. 确认记录（HARD-GATE）- **确认时间**：- **确认人**：
```

```
A.10 changes/templates/tasks.md — Tasks 模板
```

```markdown
# 任务拆分 — 需求名称> 拆分顺序：数据模型 → 接口协议 → 底层实现 → 上层编排 → 入口层> 每个任务 = 可独立提交的原子变更（3-5 个文件）> 每个任务必须精确到文件路径和函数签名## 前置条件- [ ] （依赖/配置等前提）## Task 1: 任务名- **目标**: 一句话描述- **涉及文件**:  - path/to/File.java — 新增/修改，做什么- **关键签名**:  \`\`\`java  public ResultDTO doSomething(Long id, String type) { }
```

- ```
	依赖: 无
	```
- 验收标准: 怎样算完成
- 验证命令（可选）:
- 完成

## 变更摘要

> /apply 全部完成后填写

- 总文件数: X 个文件
- Spec-Plan 偏差记录:
- 遗留问题:
```
A.11 changes/templates/test-spec.md
```
```
单测 Spec 模板
```

```shell
\`\`\`markdown# 单测 Spec — 需求名称> status: propose | apply | done> created: YYYY-MM-DD## 0. 测试原则- **Red/Green TDD**：测试必须先 Red 再 Green，跳过 Red 的测试无法证明有效- **First Run the Tests**：开始前先跑已有测试套件，了解框架和基线- **展示工作**：必须展示 mvn test 实际输出，禁止"测试通过"等无证据声明## 1. 测试框架| 项目 | 值 ||------|-----|| JUnit 版本 | || Mock 框架 | || 已有测试数量 | || 已有测试风格 | |## 2. 覆盖范围### P0 — 核心业务逻辑（必须覆盖）#### 类名: XxxManagerImpl| 方法 | 场景 | 输入 | Mock 行为 | 预期结果 ||------|------|------|-----------|---------|### P1 — 数据访问层### P2 — 入口层/服务层### 不测试（明确列出原因）## 3. 执行计划- [ ] Step 1: 运行已有测试套件，确认基线- [ ] Step 2: 生成 P0 测试 → 确认 Red → 确认 Green- [ ] Step 3: 生成 P1/P2 测试- [ ] Step 4: 运行完整测试套件，确认覆盖率
```

```
A.12 changes/templates/log.md — Log 模板
```

```shell
# 变更日志 — 需求名称
> 记录决策、踩坑和知识发现。知识飞轮的输入。
## 时间线
| 时间 | 阶段 | 事件 | 备注 |
## 技术决策
| 决策 | 选择 | 放弃的方案 | 原因 |
## 踩坑记录
| 问题 | 原因 | 解决方案 | 沉淀？ |
## 知识发现> 每个 task 后实时记录，/archive 时逐条确认沉淀到 knowledge/- [ ] **关键词**: 描述
## Spec-Code 偏差记录
| 偏差点 | Spec 预期 | 实际情况 | 处理方式 |
## 代码质量备忘
```

大模型 · 目录

继续滑动看下一个

阿里云开发者

向上滑动看下一个

![kimi](data:image/png;base64,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)
