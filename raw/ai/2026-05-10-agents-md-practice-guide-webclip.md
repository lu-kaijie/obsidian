---
title: 一个文件让 AI Coding 效率翻倍：AGENTS.md 实践指南
source_url: https://mp.weixin.qq.com/s/fBBBSfQajYjYtngZAitZCA
saved: 2026-05-10
tags: [ai]
---
岛风 *2026年5月6日 08:30*

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naJAHlYToq2PsT0gy2byUsPL8tjPPVGCwZL5OC8b24SF8xzE2V8FZaaHf0pnchX2TNX3ZBkGugZ33Q/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0)

阿里妹导读

文章内容基于作者个人技术实践与独立思考，旨在分享经验，仅代表个人观点。

![AGENTS.md 编写指南](https://mmbiz.qpic.cn/sz_mmbiz_png/j7RlD5l5q1xHFHGegJLicgt9bNzF1LpuM4F9E6zhGsmvaUYxFddS0wO7X2pibRU3NHJHYMkvtsjSpXvJCLSVe02PKddm0Nc5TpwcgHnvJsfd8/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=1)

前言

本文主要围绕一个具体的问题展开：怎么写好一份 AGENTS.md？

「在代码仓库中放一份上下文文件，告诉 AI 工具这个项目是什么、怎么构建、有什么规矩」——这个做法现在已经有了一个统一的名字：AGENTS.md。在展开实践之前，先花一点篇幅介绍它的前世今生，已经了解的同学可以跳过。

AGENTS.md 是什么？

AGENTS.md 是一个简单的开放格式，用于指导 AI Coding Agent 在你的项目中工作。你可以把它理解为 **给 AI 看的 README** ——README.md 是给人类看的项目说明，AGENTS.md 则是给 AI Agent 看的项目指令，包含构建命令、编码规范、测试要求、安全注意事项等 AI 需要知道的上下文。

官方建议的使用方式很简单：

1. 在仓库根目录创建一个 `AGENTS.md` 文件
2. 写上对 Agent 有用的内容：项目概述、构建测试命令、代码风格、安全注意事项
3. 补充额外指引：commit 规范、部署步骤、安全陷阱——任何你会告诉项目新成员的东西
4. 大型 monorepo 可以在子目录放嵌套的 AGENTS.md，Agent 会读最近的那个（OpenAI 自己的仓库有 88 个 AGENTS.md）

格式上没有任何强制要求，就是标准的 Markdown，用什么标题、写什么内容完全自由。

**前世今生**

这个概念最早由 Anthropic 通过 Claude Code 的 **CLAUDE.md** 普及。Claude Code 运行时会自动加载当前目录下的 CLAUDE.md，把内容注入到发给模型的请求中。这个设计简单而有效——维护好一份上下文文件，Agent 的表现就会变好；表现变好了，你就更愿意用它，进而更愿意维护这份文件，形成正向循环。

随后各家 AI Coding 工具跟进了自己的版本，一度各自为政：

| 工具 | 上下文文件 |
| --- | --- |
| Claude Code | `CLAUDE.md` |
| Cursor | `.cursorrules`  / `.cursor/rules` |
| Copilot | `.github/copilot-instructions.md` |
| Gemini CLI | `GEMINI.md` |
| Cline | `.clinerules` |
| AMP (Sourcegraph) | `AGENT.md`  （单数） |
| OpenAI Codex | `AGENTS.md`  （复数） |

这种碎片化意味着团队需要为不同工具维护多份内容相同的配置文件，改一次规则要同步好几个地方。

2025 年 5 月，Sourcegraph 旗下的 AMP 率先提议统一标准，建议用 `AGENT.md` （单数），并注册了 agent.md 域名。随后 OpenAI 宣布买下了 agents.md 域名，提议用 `AGENTS.md` （复数），理由是多个 Agent 会共用同一份配置。AMP 随即主动让步对齐，将 agent.md 重定向到 agents.md。

最终 AGENTS.md 成为事实标准，由 Linux Foundation 下属的 Agentic AI Foundation 托管。截至 2026 年初，GitHub 上已有超过 6 万个开源项目使用这个格式。Cursor、Kiro、灵码、Qoder、Copilot 等主流工具均已支持。Claude Code 虽然仍用 CLAUDE.md，但内容完全通用，一个软链接即可兼容： `ln -s AGENTS.md CLAUDE.md` 。

过去半年里，我为手头的多个项目都维护了 AGENTS.md——有管控系统、有内核引擎代码、有产品基线、也有文档系统。不同项目的技术栈、仓库结构、团队规模各不相同，但在 AGENTS.md 的实践上逐渐收敛到了一套相似的方法论。这篇文章我挑了其中投入最多、也最通用的一个场景——管控系统（Spring Boot + React 的前后端分离项目）来展开介绍，希望对正在写或者想写 AGENTS.md 的同学有参考价值。

没有 AGENTS.md 的日子

![没有 AGENTS.md 的日子](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

在聊怎么写之前，先说说为什么要写。

管控系统项目最初引入 AI Coding 工具时，我的体感是： **有了 AI，但效率提升远没有预期那么大** 。问题不在工具本身，而在于项目对 AI 不友好。回头看，痛点集中在以下几个方面：

**前后端上下文割裂**

最初后端和前端分属不同的 Git 仓库。AI Coding 时只能打开一个仓库，改一个涉及前后端联动的功能——比如后端新增一个接口，前端加一个对应的页面——需要在两个窗口之间来回切换。切换的过程中 AI 丢失上下文，你得重新描述一遍背景，效率很低。

后来我把前端仓库直接放到了后端仓库的子目录下，再后来干脆重构成了 monorepo。配合 AGENTS.md 中维护的项目结构说明，AI 在同一个窗口中就能看到 Controller 定义和对应的前端 API 调用。效果立竿见影——团队现在已经不区分前后端了，大家就是在一个仓库里提交代码，AI 也是在一个上下文里全栈编码。

**AI 不认识私域组件**

项目前端大量使用了私域组件库（ProTable、ProForm、ProAction 等），这些组件是闭源的，AI 工具的训练数据里没有，也查不到公开文档。最初我维护了一些私域组件的使用文档给 AI 参考，但文档总是滞后于实现，AI 写出来的代码经常用错 prop 或者漏掉必要的配置。

后来我直接把私域组件库的源码放到了参考项目中。AI 不会写私域组件的代码时，可以直接读源码里的 TypeScript 定义和实现—— **源码永远不会过时，它就是最准确的文档** 。这个改变之后，AI 写前端代码的质量有了质的提升。

**AI 不知道项目的规矩**

每个项目都有自己的编码规约——异常必须通过统一的 BusinessException 抛出而不是直接抛 RuntimeException、响应体由框架统一包装禁止手动构造、分层架构禁止跨层依赖。这些规矩在团队成员脑子里，但 AI 不知道。

结果就是 AI 写出来的代码风格五花八门：有时候直接

`throw new RuntimeException()` ，有时候用项目约定的 `BusinessException` ；有时候手动 `new Response(code, data)` 包装返回值，有时候又不包；Controller 里直接注入 Repository 跳过 Service 层的情况也时有发生。每次都要人工纠正，纠正完下次还犯。

**AI 不会启动项目、不会自测**

AI 改完代码之后，它不知道怎么构建、怎么启动、怎么验证。每个人的本地环境配置方式不统一，启动命令散落在各种文档和聊天记录里。AI 只能把代码改完就停下来，等人手动验证。

这意味着 AI 的工作闭环是断裂的——它只能完成「改代码」这一步，「构建 → 启动 → 验证 → 修复」这个循环全靠人来驱动。夜间让 Agent 自主执行？不可能，因为它连项目都启动不了。

**痛点总结**

归纳一下，这些痛点的共同根源是： **项目的知识和规范存在于人的脑子里，而不是存在于 AI 能读到的地方** 。

AGENTS.md 要解决的就是这个问题——把项目的结构、规矩、命令、验证方式写成 AI 能读懂的格式，放在仓库里，让 AI 打开项目就能理解、改完代码就能验证。配合仓库聚合、参考项目引入、启动脚本封装等改造，形成一套「打开即理解、改完即验证」的开发体验。

核心理念：地图，而非手册

![地图，而非手册](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

AGENTS.md 的第一原则是 **渐进式披露** ——它是一张地图，不是一本手册。

在我之前的文章中，我介绍过 OpenAI Harness Engineering 的四条原则，其中第一条就是「Map, not Manual」——AGENTS.md 应该是大约 200 行的导航地图，告诉 Agent「去哪里找什么」，详细内容放在链接的文档里。Anthropic 官方博客中也有相同的论述：不仅 Skill 应当采取渐进式披露，CLAUDE.md 也应当存放引用而非手册全文。

什么都重要的时候，什么都不重要。如果把所有内容都塞进 AGENTS.md，它会变成一个 5000 行的巨型文件，AI 的注意力被稀释，真正关键的规则反而容易被忽略。

模型已经足够聪明，它知道什么时候该去查阅详细文档和源码。AGENTS.md 只需要告诉它「文档在哪、源码在哪、什么时候该去看」，不需要把所有内容都搬过来。

**写进 AGENTS.md 的内容**

只有两类内容应该直接写在 AGENTS.md 中：

1. **AI 理解项目全貌的必要信息——技术栈、仓库结构、核心模块、分层架构**
2. **违反会直接导致问题的硬性规则——编码规约、命名约定、禁止项**

**不写进去的内容**

其他详细信息通过 **文档链接和引用** 指向对应的文档：

```bash
AGENTS.md（地图）  → docs/architecture.md          分层架构详细说明  → docs/development.md           开发环境搭建  → docs/design-docs/ref-*.md     参考项目架构说明  → docs/design-docs/*-patterns.md 组件使用模式
```

判断一条信息该放 AGENTS.md 还是放详细文档，有一个简单的标准： **如果 AI 不知道这条信息就会写出错误的代码，放 AGENTS.md；如果只是写出不够好的代码，放详细文档，AGENTS.md 里放链接。**

实践一：仓库聚合——解决上下文割裂

![仓库聚合](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

**方案**

管控系统项目经历了从三仓分离到 monorepo 的演进。早期后端、前端组件库、前端主应用分属三个独立 Git 仓库，AI Coding 时上下文割裂严重。

最初的解决方案是 **脚本聚合** ——通过一个 `setup-repos.sh` 脚本，将前端仓库克隆到后端项目的子目录下：

```bash
project-root/                    # 后端（主仓库）  frontend/    component-lib/               # 前端组件库（独立 Git 历史）    web-app/                     # 前端主应用（独立 Git 历史）
```

关键设计是 `frontend/` 目录已 gitignore，不影响后端 CI/CD，不用 AI 工具的同时完全无感。

后来项目重构时，我们直接采用了 monorepo，前后端代码放在同一个仓库中：

```bash
project-root/  server/                        # 后端（Spring Boot）  web/                           # 前端（React + TypeScript）  user-guide/                    # 用户手册（Markdown）  reference-projects/            # 参考项目（git submodule）  scripts/                       # 构建、启动、检查脚本  docs/                          # 架构文档、设计文档
```

monorepo 天然解决了上下文割裂问题——AI 工具在同一个窗口中就能看到 Controller 接口定义和对应的前端 API 调用，实现真正的全栈编码。把用户手册仓库也放进来还有一个额外的好处：AI 可以直接基于代码变更同步更新用户文档，我现在的用户手册基本都是 AI 基于代码生成的，改完功能代码后让 AI 顺手把对应的用户手册也更新掉，不需要再单独维护一份文档。如果你有机会从零搭建或重构，monorepo 是更简洁的选择。存量项目迁移成本太高的话，脚本聚合是一个务实的折中。

实践二：统一环境配置——让 AI 能启动你的项目

![统一环境配置](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

**问题**

每个人的本地环境配置方式不统一——有人用 IDE JVM 参数、有人用 shell export、有人写在 `.bashrc` 里。AI 工具不知道环境变量在哪、不知道如何启动服务，无法自主完成验证。

**方案**

所有本地环境变量统一配置在 `~/.<project>_env` 文件中（纯 `KEY=VALUE` 格式），启动脚本自动 `source` 。

为什么放在 `~` 下而非项目目录？避免意外提交到 Git。AI 工具通过 AGENTS.md 知道去哪里找配置。

AGENTS.md 中也明确写清楚了优先级：

```bash
### 数据库连接1. 先查 ~/.<project>_env（启动脚本自动 source，文件不存在则跳过）2. 若文件不存在，回退到 application.yml 中的缺省值
```

配套一键启动脚本，封装了 JDK 检测、优雅关闭旧进程、健康检查轮询等逻辑：

```bash
./scripts/start-server.sh                # 构建 + 启动 + 健康检查./scripts/start-server.sh --quick        # 服务健康则秒返回./scripts/start-server.sh --skip-build   # 跳过构建直接重启
```

AI 不需要理解这些细节，只需要调用一个命令。这是 AGENTS.md 中「快速命令」章节的核心价值—— **把复杂的环境操作封装成一条命令，降低 AI 的认知负担** 。

实践三：验证闭环——改完代码不算完，跑通接口才算完

![验证闭环](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

这是我实践中感触最深的一环。

**curl 验证规范**

项目中定义了一套严格的 curl 验证规范，核心原则：

**1\. 每个 curl 独立执行——禁止串联多个 curl，一个命令只做一件事**

**2\. 用临时文件传递数据——curl 输出写入 /tmp/** 下的临时文件，后续用 `python3` 独立解析

**3\. Token 获取模板化——登录 → 写文件 → 提取 token → 后续请求携带**

**4\. 排查路径明确——日志文件位置、数据库连接方式**

为什么要这么严格？因为 AI Agent 在 shell 中执行命令时，经常遇到兼容性问题。比如 zsh 下管道 + 方括号的 glob 问题，会导致 `curl | python3 -c "print(data['key'])"` 直接报错。用临时文件中转虽然多了一步，但稳定性高得多。

```apache
1
2curl -s -X POST http://localhost:8080/auth/login \\
4  -d '{"username":"admin","password":"admin"}' > /tmp/login.json52
7python3 -c "import json; print(json.load(open('/tmp/login.json'))['data']['token'])" > /tmp/token.txt83
10TOKEN=$(cat /tmp/token.txt)11curl -s -X POST http://localhost:8080/providers/list \12  -H "Authorization: Bearer $TOKEN" \\
14  -d '{"page":0,"size":10}' > /tmp/result.json
```

这套规范的目的是让 Agent 在本地环境中稳定地跑通「改 → 构建 → 启动 → 验证」循环，不会因为 shell 兼容性问题卡住。

**验证不止于编译通过**

Claude Code 主创 Boris Cherny 在一次访谈中分享过类似的经验：后端任务可以跑 bash 测试，前端可以接浏览器验证，应用程序可以用 computer use 去检查实际操作结果。当流程变成先完成任务、再自己验证、最后整理结果，Agent 的输出就不只是「看起来做完」，而是更接近真的可用。

对于管控系统来说，验证手段主要是两类：

**后端：bash / curl 验证接口** 。这是最基础也最可靠的验证方式——启动服务，curl 调接口，解析响应，确认数据正确。上面的 curl 验证规范就是为此设计的。

**前端：Agent Browser 验证页面** 。纯 curl 只能验证接口返回值，但前端页面的渲染、交互、布局问题是看不到的。在调试前端疑难杂症时，我会使用 AI 工具的 Agent Browser 能力（如 Qoder 的 `agent-browser` ），让 Agent 自己打开浏览器、操作页面、截屏对比，获取完整的视觉上下文来定位问题。这比让 Agent 猜测 CSS 问题要高效得多。

在我的实践中，验证闭环不仅仅是「代码能编译」，而是「功能能跑通」：

- lint 和格式检查在每次代码变更后自动触发
- 通过启动脚本把应用真正启动起来，用 curl 跑接口验证
- 在 Spec 的 Design 文档里写入验证方案，告诉 Agent「写完代码不算完，自测过功能才算完」

有了这套端到端的验证，Agent 的产出质量完全不同。特别是夜间执行的场景——睡前设计好 Spec，让 Agent 自主执行，第二天早上验收结果——验证闭环是这种工作模式的前提。

实践四：自动化检查——规则的执行力

![自动化检查](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

AGENTS.md 中写的规则，如果没有自动化检查，AI 和人都会违反。

**分层依赖检查**

项目中定义了严格的分层架构规则：

```nginx
L0 - entity/          → 只允许依赖 commonL1 - repository/      → 只允许依赖 entity, commonL2 - core/            → 横切关注点，不允许依赖业务包L3 - config/          → 允许依赖 core, serviceL4 - service/         → 业务核心层L5 - controller/      → 只允许依赖 service, core, common
```

光写在 AGENTS.md 里是不够的。我们用一个 shell 脚本扫描所有 Java 文件的 import 语句，按包路径判断所属层级，检查是否违反依赖方向。违规时输出可操作的错误信息：

```bash
✗ service/client/impl/SomeService.java 导入了 entity.SomeEntity  原因: 客户端实现禁止直接依赖业务 Entity，须通过 DTO 传递数据  修复: 在编排层完成 Entity→DTO 转换，客户端只接收 DTO
```

注意这里的错误信息格式： **WHAT（违规了什么）+ WHY（为什么不允许）+ HOW（怎么修复）** 。这不仅是给人看的，也是给 AI 看的——AI 读到这条错误信息后，能直接按照 HOW 的指引去修复，不需要额外的上下文。

集成到 `make lint-arch` ，一条命令完成检查。AI Agent 改完代码后可以自主运行检查，形成「改 → 检 → 修」的自动闭环。

**质量检查命令矩阵**

通过 Makefile 提供统一入口：

```bash
lint-arch:    ./scripts/lint-deps.sh      # 分层依赖检查lint-format:  mvn spotless:check          # 格式检查format:       mvn spotless:apply          # 格式修复build:        mvn package -DskipTests     # 构建test:         mvn test                    # 测试
```

AI Agent 不需要记住每个检查命令的具体写法，只需要知道 `make lint-arch` 和 `make lint-format` 。

实践五：参考项目引入——给 AI 喂够上下文

![参考项目引入](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

**问题**

前面痛点章节提到过，AI 不认识闭源组件，维护使用文档又总是滞后于实现。但这个问题的范围其实更大——不只是闭源组件，还有开源网关内核的对接细节、其他产品组件的能力同步、相关项目的架构参考，这些都是 AI 训练数据覆盖不到的。靠写文档来补全这些上下文，成本高、覆盖不全，而且很难保持更新。

**方案：直接引入源码**

后来我换了一个思路—— **不写文档，直接把源码放进来** 。在项目中创建 `reference-projects/` 目录，通过 git submodule 引入多个参考项目：

```bash
reference-projects/  higress/                # 开源 Higress 网关内核源码  nacos/                  # 开源 Nacos 注册配置中心源码  pro-components/         # 私域组件库源码（TypeScript）  other-product-backend/          # 其他产品后端（Go）  other-product-frontend/         # 其他产品前端（React）  himarket/               # 开源 HiMarket AI 开放平台（Spring Boot）
```

配合 `ignore = all` 避免 CI/CD 干扰，本地开发按需拉取：

```cs
git submodule update --init                              # 首次拉取全部2git submodule update --init reference-projects/pro-components # 只拉取单个
```

源码永远不会过时，它就是最准确的文档。 AI 不会写私域组件的代码时，可以直接读源码里的 TypeScript 定义和实现；需要对接网关内核时，可以直接查看路由和插件的实际代码。这个改变之后，AI 写代码的质量有了质的提升。

同时，为每个参考项目维护一份架构说明文档（ `docs/design-docs/ref-*.md` ），帮助 AI 快速理解参考代码的结构，而不是让它从零开始探索一个陌生的仓库：

```bash
## 文档导航（参考项目部分）
| 文档 | 说明 ||------|------|| docs/design-docs/ref-higress.md | Higress 网关内核：路由模型、插件机制、CRD 结构 || docs/design-docs/ref-nacos.md | Nacos：配置中心对接、服务发现集成 || docs/design-docs/ref-pro-components.md | 私域组件库：ProTable/ProForm 使用模式、TS 类型速查 || docs/design-docs/ref-other-product-backend.md | 其他产品后端：目录结构、分层架构、核心模块 || docs/design-docs/ref-other-product-frontend.md | 其他产品前端：页面结构、组件体系、路由设计 || docs/design-docs/ref-himarket.md | HiMarket AI 开放平台：多模块结构、领域模型 |
```

这些 ref 文档和 reference-projects 是配套的——ref 文档是「地图」，告诉 AI 参考项目的整体结构和关键模块在哪里；reference-projects 是「源码」，AI 需要细节时直接去读。这些文档本身也是 AI 基于参考项目源码生成的——又一个「AI 基于代码写文档」的例子。

**为什么不只写文档？**

| 方式 | 优点 | 缺点 |
| --- | --- | --- |
| 只写使用文档 | 轻量、聚焦 | 滞后于实现、覆盖不全、边界情况缺失 |
| 引入源码 + 架构说明 | 永远准确、覆盖完整 | 仓库体积增大、需要管理 submodule |

对于 AI 工具训练数据中不存在的闭源组件和内部项目，引入源码是目前最有效的方式。文档可以作为补充（帮 AI 快速定位），但不能替代源码本身。

你可能会担心：引入这么多参考仓库，AI 会不会无从下手？实际体验下来完全不用担心。通过 AGENTS.md 的渐进式披露设计——项目结构树标注了每个目录的用途，ref 文档提供了参考项目的架构概览，参考优先级规则明确了什么时候该看哪个项目——现在的大模型已经足够聪明，知道什么时候该去参考项目里找答案，什么时候该在本项目代码里改动。它不会因为仓库里多了几个参考项目就迷失方向，反而会因为有了充足的上下文而写出更准确的代码。

为什么选择 AGENTS.md

团队使用的 AI Coding 工具比较分散——Qoder、Cursor、灵码、Kiro、Claude Code 都有人用。不同工具各自有配置机制，Skill、Rule、Hook 的存储目录不统一。

选择 AGENTS.md 作为核心入口的原因：

- **足够通用——已被多数主流工具识别，一份文件覆盖大部分工具**
- **零配置成本——不需要安装插件或配置 hook，工具打开项目自动读取**
- **降低维护负担——不用为每种工具各维护一份规则文件**
- **兼容性好——Claude Code 不识别 AGENTS.md，但 ln -s AGENTS.md CLAUDE.md 即可**

基于这个考虑，我们把和特定工具绑定的 rules、hook 等配置作为补充，核心规则全部收敛到 AGENTS.md 一个入口。

AGENTS.md 编写模板

![AGENTS.md 编写模板](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

基于实践经验，提炼出一个通用模板：

```shell
# AGENTS.md
## 1. 项目概述一段话说清楚：项目是什么、技术栈、仓库结构。前 10 行必须让 AI 建立项目心智模型。
## 2. 快速命令构建、启动、格式化、质量检查的命令速查表。环境变量配置说明（env 文件位置、启动脚本自动 source）。
## 3. 后端架构包结构树（ASCII）+ 每个包的用途注释。核心子系统的简要说明 + 详细文档链接。前后端术语映射（如有差异）。
## 4. 前端架构技术栈、路由方案、API 层约定、组件库规范。详细文档链接。
## 5. 关键约定5-10 条硬性编码规则（违反会直接导致问题的）。每条规则附详细文档链接。
## 6. 本地开发及验证流程「改 → 构建 → 启动 → 验证」的完整闭环。curl 验证模板、Token 获取、日志路径。
## 7. 质量检查lint、format、build、test 命令矩阵。
## 8. 参考项目约定参考项目列表 + 优先级规则。
## 9. 文档导航所有详细文档的索引表。
```

建议控制在 200 行以下。超过这个范围，考虑将细节拆分到 `docs/` 下的专题文档。

实施建议

**从 /init 和 harness-creator 开始，逐步优化**

本文介绍的是一个管控系统的实践，你的项目不一定是同样的场景。好消息是，大多数 AI Coding 工具都提供了类似 `/init` 的命令（比如 Claude Code 的 `/init` 、Qoder 的 `qoder init` ），可以自动扫描项目结构并生成一份初始的 AGENTS.md。自动生成的版本通常能覆盖项目概述和基本的构建命令，是一个不错的起点。

如果你想要更完整的起步，可以试试 harness-creator Skill——它不仅生成 AGENTS.md，还会一并生成分层架构约束的 lint 脚本、Makefile、验证脚本、参考文档等配套基础设施，基本上把本文提到的实践打包成了一个一键生成的工具。

然后根据你的项目特点逐步优化：如果是全栈项目，补充仓库聚合和前后端联动的说明；如果用了闭源组件，引入参考项目；如果有分层架构约束，加上 lint 脚本。不需要一步到位，从 bad case 驱动迭代就好。

**通过 Bad Case 驱动**

不要试图一次写完 AGENTS.md。从实际使用中发现的 bad case 出发：

1. AI 犯了一个错误（比如用了错误的命名风格、在错误的层级引入了依赖）
2. 思考：「如果 AGENTS.md 里多写一条 XX 规则，AI 是不是就不会犯这个错」
3. 判断改哪里：全局规则 → AGENTS.md，模块细节 → 对应的 docs/

这是最高效的迭代方式。AGENTS.md 不是一份写完就锁定的文档，它需要随着项目演进持续调整。

**规则要有执行力**

重要的规则要有对应的自动化检查。AGENTS.md 中写「禁止跨层依赖」，如果没有 lint 脚本来检查，AI 和人都会违反。

规则的优先级： **能自动化检查的 > 写在 AGENTS.md 中的 > 口头约定的** 。

**团队共建**

鼓励团队成员在遇到 AI bad case 时主动补充规则。但要遵循「地图」原则：

| 改动类型 | 维护位置 | 举例 |
| --- | --- | --- |
| 全局性的架构约定或编码规约 | AGENTS.md | 「所有 Controller 统一 POST」 |
| 某个模块的具体开发规范 | 对应的 docs/ 文档 | 某个 Service 的调用约定 |
| 前端组件的使用模式 | 组件模式文档 | ProTable 的某个 prop 必须传特定值 |
| 参考项目的架构说明 | 对应的 ref-\* 文档 | 某个开源项目的架构分层介绍 |

如果细节规则都怼进 AGENTS.md，上下文会膨胀，重要的规则反而被淹没。

**标注给谁看**

团队中不是所有人都用 AI 工具。在推广 AGENTS.md 时，明确标注每个文件的目标读者，可以降低团队的理解成本：

| 文件 | 读者 | 说明 |
| --- | --- | --- |
| README.md | 人 | 项目介绍、快速开始，给人类看的入口 |
| AGENTS.md | AI 为主，人可浏览 | AI 工具自动读取的项目指令 |
| docs/\*.md | AI 为主，人可参考 | 各模块的开发手册 |
| scripts/\*.sh | 人和 AI 都用 | 构建、启动、部署脚本 |
| setup-repos.sh | 人执行 | 一键环境搭建 |

README.md 和 AGENTS.md 是互补的——README.md 是给人类看的项目说明，聚焦快速开始和贡献指南；AGENTS.md 是给 AI 看的项目指令，聚焦构建命令、编码规范和验证流程。两者的内容可能有少量重叠（比如项目概述），但侧重点不同，不需要合并。

一句话总结： **脚本是人和 AI 共用的，AGENTS.md 和 docs/ 下的文档主要是给 AI 的上下文，人不需要刻意阅读但可以参考。**

总览：项目结构与 AGENTS.md 全貌

最后，把本文提到的所有实践汇总成一张全景图，方便你对照参考。

**项目目录结构**

```cs
project-root/  AGENTS.md                         # AI Coding 项目指令（核心入口）  README.md                         # 给人看的项目说明  Makefile                          # 质量检查统一入口（lint-arch/format/build/test）
  server/                           # 后端（Spring Boot）  web/                              # 前端（React + TypeScript）  user-guide/                       # 用户手册（Markdown，AI 基于代码生成）
  scripts/    start-server.sh                 # 后端一键启动（构建+启动+健康检查）    start-web.sh                    # 前端一键启动    lint-deps.sh                    # 分层依赖检查脚本
  docs/    architecture.md                 # 分层架构、依赖规则、领域模型    development.md                  # 环境要求、构建运行、数据库    design-docs/      api-design.md                 # 响应格式、错误码、端点详情      controller-conventions.md     # Controller 层编码规范      gateway-integration.md          # 网关对接详细文档      frontend-architecture.md      # 前端架构、组件库规范      ref-higress.md                # 参考：Higress 网关内核      ref-nacos.md                  # 参考：Nacos 注册配置中心      ref-pro-components.md         # 参考：私域组件库      ref-other-product-backend.md          # 参考：其他产品后端      ref-other-product-frontend.md         # 参考：其他产品前端      ref-himarket.md               # 参考：HiMarket AI 开放平台  reference-projects/               # 参考项目（git submodule，只读）    higress/                        # 开源 Higress 网关内核源码    nacos/                          # 开源 Nacos 源码    pro-components/                 # 私域组件库源码    other-product-backend/                  # 其他产品后端    other-product-frontend/                 # 其他产品前端    himarket/                       # 开源 HiMarket AI 开放平台
```

**AGENTS.md 摘要**

以下是管控系统项目 AGENTS.md 的章节结构摘要，供参考：

```shell
# AGENTS.md
## 1. 项目概述  一段话：项目定位、技术栈（Spring Boot + React）、monorepo 结构
## 2. 快速命令  构建、启动、格式化、质量检查命令速查表  环境变量配置：~/.<project>_env 优先级说明
## 3. 后端架构  包结构树（ASCII）+ 每个包的用途注释  核心子系统简要说明  → 详见 docs/architecture.md
## 4. 前端架构  技术栈、路由方案、API 层约定、组件库规范  → 详见 docs/design-docs/frontend-architecture.md
## 5. 关键约定  - 异常统一用 BusinessException，禁止直接抛 RuntimeException  - 响应体由框架统一包装，禁止手动构造  - 分层架构禁止跨层依赖（make lint-arch 自动检查）  - 代码风格：Spotless + Google Java Format  - 安全：无状态 JWT  → 每条规则附详细文档链接
## 6. 本地开发及验证流程  「改 → 构建 → 启动 → 验证」完整闭环  curl 验证模板、Token 获取、日志路径  → 详见 docs/design-docs/api-verification.md
## 7. 质量检查  make lint-arch / lint-format / format / build / test
## 8. 参考项目约定  参考项目列表 + 优先级规则
## 9. 文档导航  所有详细文档的索引表（architecture / design-docs / ref-*）
```

总结

![打开即理解，改完即验证](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

回顾这半年的实践，AGENTS.md 的本质是 **用最小的上下文成本，让 AI 工具获得最大的项目理解** 。

写好它的关键不是写得多，而是写得准——把 AI 最容易犯错的地方堵住，把 AI 最需要的信息放在最容易找到的地方。配合自动化检查、验证闭环、统一环境配置，形成一套「打开即理解、改完即验证」的开发体验。

这套实践和我之前文章中提到的 Harness Engineering 是一脉相承的。AGENTS.md + 文档体系 + lint 脚本 + 启动脚本 + 验证规范，本质上就是在构建一个反馈回路：AI 读 AGENTS.md 理解项目 → 写代码 → 自动检查 → 启动验证 → 根据结果修正。人类的角色是设计这个回路，而不是在回路中的每一步都亲自操作。

一个有意思的观察是：AGENTS.md 的维护过程本身就是一种知识沉淀。过去团队的编码规范散落在 Wiki、聊天记录、口头约定里，新人入职要花很长时间才能摸清这些「潜规则」。现在这些知识被结构化地写进了 AGENTS.md 和配套文档中——虽然初衷是给 AI 看的，但人也能从中受益。某种意义上，为 AI 写好 AGENTS.md 的过程，也是在为团队做一次知识梳理。

如果你还没有为项目写 AGENTS.md，现在就可以开始——用 `/init` 生成一份初始版本，或者试试 harness-creator 一键生成 AGENTS.md 及配套的 lint 脚本、Makefile 和验证基础设施。然后在日常使用中，每遇到一个 AI bad case，就补一条规则。用不了多久，你就会拥有一份真正有用的 AGENTS.md。

相关链接：

- harness-creator skill 下载地址：https://market.hiclaw.io/skills/product-69e7187be4b0d28be543a809

继续滑动看下一个

阿里云开发者

向上滑动看下一个

![kimi](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAIAAAACACAYAAADDPmHLAAAACXBIWXMAAAsTAAALEwEAmpwYAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAEv8SURBVHgB3X0JmBzVde6p6p5Fo22079JolwAtSCxaQAiDQBgjwEu+L9iOlxgc57NjbOA9YkcSYF4+x1sgeU6c5BmMk7B4FRiDhG0Qq4RAQvsuNNp3abTMSDPTXfXOf869Vbeqe6RZJCFy9Y26u7q6lnvOPec//zn3lkf/A9vdd99dVVpaOj4Igirf9wfl8/lKz/Oq8IfvwzCsKvY7/r6aX2r4+xp+j79qPsY23rYcn7///e8vp/9hzaMPeYOwS0pKprOAxrHgphvhVtK5aTX8t5yVCorwKl6/+93vVtOHuH3oFOCBBx6oPHHixHju/FtZ2Lc1NZrPV2PFgzIs5+t44gc/+MFC+pC1D40C3HvvvRjdn+MOv43O3Qhva4PbmMd/z37ve9+bRx+CdkErwH333TeehX4rv72bWiD0+vp62r9/Px09eowOHNjPnxvo2LGj/Plo9D22xQ3dEFKnTp2orKyMysvL5LVnzx78Ws6vPalHjx6yrbnN4ImFmUzmwQvZTVyQCoDRzi9z+W96c/bfsWMHC/oA7di+g/bzK4QNgVrBxrdpX93vfP4LzPYM/+Up2S3x7zt16szK0J0GDBggStG/f39qZlvIfw9eiC7iglKA5goeI3jNmrW0efNmHun75LM237xCoJ75g0AzZhu+D5193M92f/e3rqKQ814/l7UrpwGsBMOGDePXAWJBTteMVXiQo4mf0QXSLggFaI7gVehrWOhbeMQjMnMv3R3ZaHZUp2/PFaa7vx35xX6btiBBdBxPNpdQ6Of45yFbhoF08cUXiYU4nTJcSIrwgSrA/fffX5XL5R6n0wh+546dtHrNahF8ff0pKm6e7TZryt193JGedgn2GPZ77GuthVfkN/rqJX7O+3v51HlDVoRL+G80u4kB1FQDYGSM8I0PEiN8IApgQrmv421T++zcuZPeemsRj/adVDiaIQjXX6dNdFrgaVPumv5iv7XHda1BrDie15R7oOgYYRiIogA8TpgwUSzD6bqkQ4cOj3K/1NB5buddAWDuuQMfbyp+h+BffHG+AXJucwWSHqnpljbzvhFawIJxj+ccXT66gJCMEH1KYouwyLnTyuA2VTa4hMmTJ7EiXEzFGtwC98kXzjdQPG8KYEY9/Pzdxb7XEf+WIPpkS3dsutPTYI6a2Pd02/SzCtsVMDn7+kU+s++nLMX4gZz9POcY9phQhI40c+bMJiMIJrgeYQ7hG3Se2nlRAPh65uNfKTbqjx07RvPnLygieLQ0AENH+6n37n6uQqRHqD2GvleLoJ/D0FoJa1nCM1wL9rOCL4Yr0tfkXqueAy4BFqEYWIQ1YGxw7fnABhk6x+2ee+75HHcwWLHe6e+WLVtGzz//Ah0+fMhsSSP7dIhWbHt6nzQGoCLHNls8++qb0e8qgPmTw/hUHEvY42aoqVCxqTF24MA+Wrt2HeVyAUcNBdagkpNQn58yZUo9W8XFdA7bOVUA9vf/yNr8XX5b7m7HqH/22WdpxYoVlM+7ppwo2YGuMNMd6lPTEUFACSGZzRj1KmzdqEIninFFEgPoj1wz7ipWxvlt2Izr9qNXj60Hzp3P5ySkhSIMGzY8zTSiz2ayEsA1vkrnqJ0TFwB/X1tbC6B3W/q7ZcuWi6+vrz9JVDTWTrN06dFmTaqnv/DM9yEj7wLzb5TBE+nzrva7pgge9zzpCMAKMH+a67atKQuUN0rnKjwrhJ+nduUd6Morr6Dx48dTuiFcbN++/RfORZRw1hXA+PvfsvATdwIiZ9GiRbR06VJKh0yFIyU94h2BQZBF/W6RUBEC98x+PJJDjFoge9daeOa3oRdvE3CXiY4ThhRZjfhc1uy7yhYrZxxOWkVSBfK8+J6BPXwf0UbsviZMHE/XTLuW0u1c4YKzqgBNgT2Y/HnznmW/d5BczS/0q8WAFEX+2WA1/qyC1K+bshZWSPY4xajfUPy7nhkKY49TjHcg53fu8QujicTxo21WASjxvSdn1n7IZkpEUTt27EQf//gnCgDiuVCCs4YBmhI+kjS//vWv6ciRw87WtMktIpiIdInNqfhw05HRr80bz0tjgLRpdsMzI2sRAExwqJYiPB3ALLbNNeXp0NEr+tuYb6DUvr5aKT78yZP1tGXLZskxpHBBJdzqtGnTnn3jjTfOijs4KwrQlPCRrAHYq61Vf5/U/mKdSpS2Al7iPzNWQ2uySQXneyasS/8+Nr9h6I7GeD8l9TzHBVjl8YtcX3zdXoIxLLKfGhdVMC8Uq5W8RnMeLz4urjE0Vq2+4SStXbORunXrSl26dHHu6ewqQZsVoCnhI3Hz/PO/lzDHmvo49iaiM4RJcSsGBIPEHujkEEpQkAs4s4eLr8l1TUSFOIWiz14UVeC7LMXA0vmtZ5TLa/paPO5+C2Q9RxHsvtyvtHHjBnEFoJSddtaUoE0KALTP4G5RMeHPnz+fYh9OTYxQd1sx5EyU9rGhp4ydZ0e9De30S+d4bnjpR9dgfmLeux1uTbQdsWRevcT+MYgzbiT0daR7rrtKvi9s1g0pHPUFh7jXi4Ploz7ZsuV96ty5qBJMv+GGG55ZuHDhKWpl86kNzYR6Ve62WPg25i5mQl1/TRSDNEpttzy8/ZwxJtUj1V2EdtppUAxKuJiM+W2xWoCsc+z0uZ3fh/GINiKTURsrC+4xH12b7CnbssZFGODnKLcXAUf9LRQKiuSXVVK29zgqqfoI+eVdeZesc76QFix4ifmCteQ2RFqQAbWhndlGNtGY5JlLqWwefP68ec+R3lya2HEFi1aMsiVqMvwTAsUTc6mjI0NxfGZ9aWi9DRXy8C4HkD4+Jc4fm2N7jQ7ljPsKbSbSp6Tvd0exbZl4G647VArZ9zNy+dlB06hi+mwW/DWJq2isfpVq599D+b2r+ag5stbplltuoaFDhyb2bUv+oFUu4L777kMq97vuNoR6QPuc35fPiZx5k7G9/c4tsvAcl+FajDB1XCv4wEHvoQZWXsofS7PCSZtmuz3vXEMm9VtXSc0+5NQBeE1ZL3Ku3QWQOvorps+hDrf9lDKVVZRu2FZ+2V1y7sbq14yu+1RdXU2DBlURE0PxHYThpMmTJ1czz7KCWtharAAAfcxTP00OvQvhP/PMM3AJ9pJSgM+29KhzY3Vnr8RvvdR3LpPmugdjJbww9VvT6Q7IinMAmdjPkx+FaElARlSIT6xVIeMmioWarsIlS9RABZeN+wK1n/lDOlODZQj3r6LGAxsk+snnA9q+fYdYATdE5HuYzqDwmZaCwhZjACB+SlXoQviowE12WlNmHc01kelUqh+FQvq7uNrGs2Y+YXKtWS/R38jXnrM9onkSPluPpYIRNO5l5b3vZZxz2++p8Lo9c3woEJVSbOq9aLtaCrcP7LVg9H+bmtsqZv0HZdp1iu7/6NEj9LvfPevUQkqrhGwAzKkFrUUKgOROGvS98sorUbm1qwASq6dMXyEL6Jh5LxamjmIXXNltRIWj21qBRuf4aUImsJCdYotjASILxMsZP+ubV/c6g4ipc6/bCw2uYIWyPlp/l9f3xjKElC4XIyobdTv5Rcx+U80rr6RM7/HRvcJa7dt3gJYsSSYKIZu6urq51ILWbAUwhZuJYg6kc5cuXUbFR3u6pVG/3WaAVRRmESUBhEvMmD/P+W0R4cS3Zs9phBWmcYErLCLXbHsRgjfRholA7O9D81tsz2SSCu3ZiCD6je9EAj4Lcwy1tJVUTXOuXQfKe++t5L/3Evuxe77byKpZrVkKALOCMi53G/w+snrxRbnhlN3mtrDoe9f/hkb4oec5KuI7oz8wymKTPOnjF/PBbjQQpM7vGcOQp6RS2lFP0egPo232mF70HmlddVGeuQ/9Xn+jQDV0ytgyLRj9tvnlXcimrQMDCHP5RkmwHT9+PLEvZNVcV9AsBUABZ9r0w++fOgX+IS346CLMO9dnF+xFUbqWVNjSYaGGc0r04BunSAPWPAidUM9xI4lzWbPvhH8RhoivS8/hcgIUnb/gUj0X2OXNrnEoKFRu5PdtsspckxcaNNE66iU4VUNJQKvXiKhLeZe4QVYss7ubc9wzXg1QP6XifYz8pN+3rz4VIl93n2TT0ZJxCDwLnHzHC4SUcAPSkWnUHaauIVCELwxdmuK1+xdaKM/SupGFiYGb9HfgKrurdNZVqDuJ8g7RGPCNvFxL07IGXsAOqg4dOpiwUD/v2LFb3HGqzTWyO207owIwsvxH9zNMf3yyYqPPmMxI+4spgY7s0I4Km9zhH3nR79KXaM+RDv2cEU/kULzu9cXfJ0Gnc92eF5n9pD83lsFLWzWfki7F7QerrPZe43qA1jTwAFAAHLu0tCQKt6FojY2Nsn3lyuUiG7eZORenbae9KiZ8Pp+u6sHoV9Mvl0BJIRQbWUSFyJ1E4J4x4xFxI18HZnvaXLo+Pn2e2B9H/ICTOk5aJ3PbYXzdXjTHIGb34rSzWqRknsG97ozZr1G/9cDMaUgZd0umyD00rwU11XRi3l+S4hWl11FZDODZvqIDn0uJr1OnGpkunp/++fQzAcIzXc1c9wMqd1evXu1sUY33klNl9JsCJG/MZwT2LLzyotGvr74z/gNK0rfGjxdMzLACtmaYBNhJeOYFUQIobm6VrmeuIZM4pprrwJzVtR6xa/N8XxI51qLJvjh9mFO+wUsNEM+eu3kNwj/29Ccoz68KTAMZfPD7IITq6k6KQvTt20cUAfLBrOhUm3u6czSpAGb0V7nbbJInvpv4z3PDuGIjNLKeoZ7UB5WaMXhLO/3WWbcKota/vPMXxNtyjbRl8xaKgaEFZKooFr+FECh499CGZDEdrb7e3kNIVBCre2Yf/c2cOXO40xv5r4H/8uZ9jhrqG6mh8aS8zzXG2xsb9e/ggUNUWdmZIsXBCGbSKLd3uQg3qNlmXqsTn2HykQeo+cllvO/K6Prwa7B/9XxesT4G33DoB6Aue7z55puUaqe1AllquiU0B1k+ZfuaarEPFHDnWgZbbSNpW+NPcVOhU1LF+1ZWNo/EqqoaFKFqxRppPbZ+PTCX5HACUnWTrudPW5Qwsh5zZs8VBWhNq6k5Kn9QJj1eTu755OJ/4r9/TpzPdS3aH6EBj1mDPzTK0HUNQspmS0TZsG3Pnn1UUpJlt1BCW7dWyySb1MQTyHJhsWssagHMahxV7rZkzE8UmfQU+Ivz2a7Z8026NmfOqKZezLMcysAvrwUIOcIJLhDLU9JMWz+cZuSckE/2VeHYGgDbLXNmP8jCn0utadXV2+j6669j08yK6gc6GFj4EdD0jNUSV+H0o2cSTZacMtR1GOh1x65Vf5MtUSWCdYR7gHKnw0I6jRVoygUUGf120QU39i7W3NFkGipxxZ3bEMkthzJpXrXb1PyWd3IGtrluIb6OxGFDV0F8owdZGWmhsyNG/Zw5s6k1DfMdIHwoQRBoCZvMMzTuSvIOoa1nyKSuN2uUNZVN5K8zGS0bQ4gLV2QHUiaToT59+lKnjp1kG4ihgwcPpi+rqCYXKICJHae727SUO91iJdBaNgfskPpcHXw2nLIoiWJhO3G11wqEnACHUc7fXlusqDE2IXJTz0kSygLIrBF+68w+hH/ddddzxq5agBny/hHX5HmyTZTAtx3hUtQZAa/k4igv/p2d1IJRn83q5JKQBxVSwwMHVtGgqoGoDWClC+n1119PX9p0LLmT3ljQ42xKCpB/IbJ0zb+2JOpXAeiWlJsINbASmjS0hZCeYxma21wQKldOsXBTkYMtKae45Ct2BR5pQkdH3ew532rDyF/Owr+O/f4RstYILsDPaBgJ8wyLoCbeRBteDGAjFxb6Tn+R9BmAnh31Qd4qNoPjoJEBYC2tW7eaNmzYINeBEHHXrl2CBdxWbKJOsSE33f0A819o7ot9Tm2zo1xeTajlmt3Q/W3QxHFP1xyKN/ptmPrevsaJHSiAxulocTIISjF3ztw2j3yAPnsfLDOZ+pbPqV/3LP6RUjDzXnQxGVHF9+P0mbm3UJQhL1ERjt+xY3txATU1WgZQUlIigxEg8eTJk+nL/Hp6Q0IB7rnnnsS6e2CWVq9eS0nBOIjaXKgt0oiKJx1zVShUV0jpoo6WNJdqdo8bGodjXUqQCE8D43Y0g2fSweyfZ8+ew6P/76g1DcK/4YYbJC6XEW9mL/m+vSIe+VEpe04UTkCdtfDklKFZckkAcobcaiMFlGY/UgII52xsbJDfACMg/JRzhnpdDQ3uamhUmQaDCQXA4ovu53jKtjvKqIltcThHTp2eOw07OfSt0rTE7KdbMUBqzu+75wnN5A+9zsCcMpvV382d20aff/1H6MiRIyKw0tJSEZKfwbGVg8j4WnmklkZrCDxfLaGtbgawg//2hcTU2sEod2CUIgwtJIiVQkFhDBixT6fOneT9tm3bjQWPG9ZadD/7qS8TPkJHfxLcxf7fbYYxi8AdieaGACkeUbJmrvAYcfVwSy1Bmhp2lC2aCKr+H2AJnat/GhmAYIK/b22ot2LFSpox4zo6fqxOLEt9Y61QsnivMXpe7kuF5Jl+8ORa8B0EHkcyecVDgU11Oe4Jd8qhpFgvSTYZF8P7V1RUcHKovVixkycb5PUouyFUC2GNw23btiWuGQttuqniSAGMaYi+gPnX1bjSPtZVhjjUikMuLwZ1YvYC5yehvQjHLKdj+ZY2F+zZqCQwfzp6Muzz/Uwo4VcmkzX+My9mv/UjfyWHejzyubNB/fp8bC0nC6JRKaF/GBhSzBeBt29fTra/IsUgwxGAvpZwUc27hI1eECmJO0awH1yNMJINjXzcdlEkBhyA7zEDe8uWLQWlY1hq136IFIAvJGEakit2FPPT7oh1BQfNzam/g58L0wCn2Ci3WKAlClAIHm2dfQz8MmTTs75XElG1qB+cM3d2G+N8NfuW0BKSizTs86W+0JrsrPQBRi/+nTrFypIJ2P1kjGBNGVmUMbRhoU/KnGZFSYIgMMfMRyEt3E19wylRZhwfSSI0TRercnXv3p3Wr1+f7DldbldapADp6dyo8Y9beuTrtsIkS1op0n9BMs/vWdDjxsLNbW48b4FoxrEuOFcuCvHQSRj96Jg5c/5WKN7WNBX+DCZbTpjYPEM2txDXAeC8jVpWwKbb91W5NQzUOQ0QmCaSsmSQnfIGrByenyNLWaslk4OSDROxrV27dtS5cxcxsI2s2A0Nmn/AvWMiLn6DvEH37j1p06bN6duIsJ6c2ZA/CQVIWoBipp+oUCnS2CA54r2Ez7ZabrTIa6SWYQAbyhl2zZzBCsHjEe9RuQAz62vz+Qbx9633+UD7N7LPPyEC1LAyZzKAvhE0Gp87LI3v11PgBldUWtKONFLh32bMgAjje/G9sui647IGM9dCKp/1XOXl5XwNanXUwuQN4vdMpKNrMlRXb6HDhw8n7gORHpbZ1zNya2xsLBB+nPM3dxCFHu7EDfd73/lzFcTZz3e/dyt1KRoFzW8ubsgYUxRvg18OqcGMolA6bc6cB1pt9leuXMnCn0FHjx2mTDZjogq9b88wdVoJTcbyOFXKQVYsRcDXlGMl1GRZYPx+PLIVMzSyPBs0UvA0eygWRuoKc6a7Qh7lNXTo0BFyB6dGEV6Cle3QAQtglxaQQnjGgvzGfJ7ufok5/eTE+U37btssqnfRvQsO7WkC57AmCA6tG6CWGYCENQkodJhApV7ja0Z/zJ79rTb5/BkzbuCRX0uw4EE+b/rXhmlaFo7OBymjt5c1IZ/6eM/4c3s9FJXAa+2AdIXUMBDZugg5jp8zimDcGwu5pAThZtZkNa3FI0MAZSUysFPKUT0ErLBv3770bY2PehFP23C/0dU5bUv69DhhklYQd1v6vVEKm4jxzNEkxekmcFrStI4/Or8zv19GlXPc2bP/rtVof+VK9vkzZvCIOyQgDqMM9GsY2CqfOHMnZzNWAeUOwBy6lGwoxBOUoKKiTBShoqKDKItOQ8s43ayRhL03VagwETUBx0IMl112GU2aNJmGDx8mx8c2uxS+1gcQW/KTzBZ2LLAA3K7Bf9b5JFwAVuCO/XvSryfr4jyiAo7AbnOXcvWSu5icgB5PI4aQLFPWkuaafO0wsGWK/PXcP/rRj+hv/uZr1JqGkY9FHWvY3EbVPaGCtSB0U8/mXll4+SAv1ifjg5XTfTBiEX0AE8BPw1plsxWyDTn8BoRpAIsmEwgat7ExMIkjU18RaCLZZgSHDB9KBzjjt3/fATl3t+49OP6vIRCB4Dfat+8gioD1lcEFDB8+PHFvlvH1TYYoiv+hQacv/EgcxnSCO53KCtwtuoitR5w5JEpii7CFLsDijlgh7VTr0JjXxx77f60WPnz+9ddfL4kd31MAKzx8qLX+Ga/Euc/UVPGQzPJ3ek0QNIRaUloiI74kWyo8PQo68/lGsd+lJWVUVl4iwhZl8UDytIsigNAUhOC4WHTj2JGjbN5PUN2pWkH/Q4YMpn4D+lN7DgG7d+8q9QFdu3aVy8HxUMqX5gMABH0+aKIMZ//+A1SI7pviAOx2NymT5ujT+zaBJ8K0NTlTS/MGVrn0Gh577DH6i7/4HLWm/fzn/0mXX3655NUxmshl83CeIBRAR/Z+ohVG43uTuD+0/IQi81y+Xr6HlQBLB18NE19WViIRAgQJi6BTx0OpkNL43xcl1ChDOQR8d+jgIUmpY5eDjNvqauuoXVk7CRGxmAR4Dxxn4MCB8qrYLm54shqOmDD/eMRK3NKjLFVcUTQ0lNunpEVoqsW/b7H1Pw17+Nhjj7dJ+F/60hfIpmQFvQcmlg+9mNK1ZWaeSzv7JuTzxP/b2cbYpllBkhGP7+rra8U829DxFLN2ZWUaIvbu3VfM/66de8yx5KCyL0AemMzdu3dTt27dIuRfU3OMcpwUgsJCqfA9MoR4b+ngdNm4PFYvXfqVDP/I6WSvCeLHfp/e7o7oJJCMV+y04SD6NKSWWQA9nmdDU5Ml05H/F9Sa9vOf/xcL/y7SNXssrgjEp5cAdZsJJ4JZPAWhdpUxLfnSmkc11Xa6GKhoktculd1NgQjyBb4oVmNjveAC9AcsAGhcXVHNhpo64gEcUQ+A42LNoN69e0eoH399+vSWVDSmjUPBcEy7VgPcAY5/6lTCBeD3VT4erOhuTJqJpA8vvi1t2p2Qj+KETHJf97eOS2ixGdDEiYa9GRH+5z7XWuE/wcL/osThQd5X9jDUxA4SNPmcgjLL5EVzFsUCGFrX1wUmUaAJ4fkmFEURSDbr0/ETR4zQDbUrrsWPmEtlCkND8Jil6vnYqALGtShPoAwfij+mTr2KzXiZrDW8Zu1aEfThwwcF+UMpevToSWWl8RoCBw8mXQBfQ+dsGgOoBbBCDqhwsYZio9qMZEcwxYs8iilR+rfNa15KVx577D/aMPJ/Tl/84hfZz5YaCllHvy5Jo+AO/jzL4E1HFZRAp5GFIkdN3wo+oLwCwFBnEIeG5/BMLgIWv6SkVFhJO5cw65cqe+dpGTx0Q+kEthJhg5BBGZR68fe5xlAqtLCG8N69u9i/DxB/bzkIxP2I+SE3LMKN46GBpML5k33oVeGqEwqgZcdp4Rbz52GR7T4ll0pNj3prEYiKW5KWgkA9lvr8tglfJ6rkKZ4/kHWuTbNryLppvsHOCiJD9sSA0JI2MPX5oIE8U+cn1LGPqV0VfJxTMoobmb/HPvmgXu4nKwCQeYYcu4NcnTB4OG9gIBXSyMQsIXIIa9asFmuAnASsNuje0JSOgfjp3LmzKIpiBi2AgeK5TVwA/1fEArgtoKQZTwu1KRfgpX5v3zv1f5FxcX/X3KYupm3Cf8KM/BJSps21SA3iq0tKfBmNoanpU4rXEj+hjvxQp4JF8xJ9FjjVky4/oyMQghWbwCCwoqK9jEYtTCmR4ylR5FFDIwu+nETJGhtVFjgk2EcoC4gzKBNC9REjRsqot+EhhK8Cz5tyME+iCq1JCCQaSDe/2Lq+hbV2xSpu0xbBFbSrFH7qs2km5RnahFBB+Hj6Nm7cRBb+T1stfCDkr33tbsqygKW23rJ5YaihmFfO77PS2QjZIEiAMVEWBoLl5e0k/y9UrQhQOTVYhCCnSR2pPTShY0bSFVmJBOrqavlzmWT+ysqyvF+JnAfEURhgH8wfUFyhYSCfw88YejmIlOX997dIOVhdHYd/5WXyXMO+ffuJEiiXgN83MC1cKYoSz+0ge69VBcOuOMp3Rr87yyZSEmveyfkcFnkf4wnxj5JRK4YVztyWLl3SauGjIY6+995vCsgSxs6MfhFyxhfQVVpq436PSZUeYkIzWY2GTnEYp6STLvIoJt5TXwvkHgQNUdUPRrkifFV0WAZYgtLScuNWciaaIUkfI0QU8imafJqRfhoxYphmNvnTJZeMoc4y7YxktIOgwszhffv2CuEDBais7CKWAegfitmVw8Z0K2J343DP8wq/S5Z3pYVuX10rYU8T+/yoeFQQNDa5+56/Nnv2bPrlL3/BoVOVlFxpnJ3RZI8XCh2LEVh36oQ8xKqhnnPuDXkmWjRdKwkoebCUJqI0LewKTgtDPUNz19fnTJLKFyIJPIDFE5pRJPHvwiGY/YAD8D0MAQpA4d8nT7qSNm3eQHv27pWRDdOO3wMAIi+ABgUAU4j9hw4dIqzjnj17CvqgiALY0eyWVQdF9iFnu7uunjvaLZkSOjcaUhIkOkmVD0AJsPDiSy8toE984lPSmQi5IH/fsHda1eNJ/F1eUSIKcvLkCcMRaF/ZhaDjtQm0z4AbQNtqYQjQf5mpFFYuQXGHXWzKmHv4eFNRBWsCDKKKFbDbOiyupw8TRTfN/GjE8IHowahH7I8/cAl2qfkhQ4ZIYSgmj5SWlBTcfxEFcDe5vrkplG4AXfTepDE9Lc2O6/6aAnqBc56WAsGz0zCr5sknn6L77//fci0ww9rimj0dYXVC2SomAMFTYpQ6K9m/0H0iSRQl6HwAySR6vilH15VCdQUTkzaWs2TMubWuEhYJeEHPr1PRoQSHONbHbK3jjNeQVezdu5d52HVPWVcYox24AFYAkQAeYjl06DC68sorC+69oMfBSxeO5LQv8FLvk0DPS2C+tOK4rsDMkjGd/UE3FIlu2rSBiZUqIVhkGlY2nkouRZj5nIwyCFCvGViGR3te+QItETMMocEEoc2GUyCVvEmrGkQjXL4PdE4BStYlcjDWBSP9BJt0rBIKMAcMg+xfbe1xKQfr0KGjXA0SQLhm7I+aAPwWbgRKYmsV3Oab59hGray8nJIj2gVxxbYl30dFmaITnqMERElMYJIlnjJeESb4gNugQYNo6bvv0p13fkk6FaweWDyttPWMkFjsOZ2ZI6PXN6uBmWZXE4vr/FmgqEbmeL9U3ADci+YYLI2MiAHb4WJg5m3srsLXruzbt7+YdtQAQAkBBFH0CcLn0KGDVHvihAhfS+BC4QaQ0IILeOONN+jSSy9N3Ctk34QLSJvsNKnjpd7bUihP2VxfmTBVbfcYLrYwCDskKuQXPtgGEuWHP/wRPfroo2xW+7IpDaPJmWRAns1jCKcRWALMzn7W/tHRb5Q9gFUoESwhJd2ZvEg1yNvJMTpwEMM3NGjlbzZTLtt1wkk5nThxTMJ0CBtz//bt2y/zAtGGDBkmox9K26dvX77uXrJ99OjR0WuRSb414AGq3S2dOrWnwhFfzA1YH2cRvZo8EW+ombMkX2B5gny0TX97JozxwbXPfvaz9IeX5tPIESPMlryMVICp0DyWXlK9lDMmX+nhyJIZC4davsA8iAoJHQHEVKKhnp832MHMIsp6JtWcoVMNtWQXqsDvQP507dpNav3h40+dqhPeH+6qG2+H9RjJ5FBHthI9e/fgBFEfqWuAUsFlIIHkNpZ9Dc68zd1Y2dk+nsSOSgfYSEsrgu8c0DPlWBYYZqKOi/f1Usd2levsgcB/+qdH6aGHHqK2tkFVg2jlqhV0333/S0YVzHZ9vS3iVPOslTz4BxpdkbbkDyg0mKAkUnYdLNjHYiALJDW7KHSwjZikXF6PB1cCkmf58hWC+OEKYA3g50EG7di5TcLANWtWUS2Hl2VsMbBfX7YGo0aNojFjxhSkg7kdhQVIrC4NwBCbapf+JXNjaetgRrCzZk3sMmLTps0u2BR/TnIGZ8cCPPTQg/SNb3yDX78j07WxxHpb27e//W36zW9+wxFDf04Nq3u0fYF5gCJUM/nTLluj96qTPjyj7EInR27PiX5kKZicmTCigwTYQy1EGKV8gQsQnuKx9EhOnThxkrOCV4tFAE+wa/du6typI/XkBBEKThAJfPrTn+btuxhE1ibuSZ5CNnXq1FH8fqbdCPJg8+ZNRAV0rsbD5JSFKwBSNswrsBb2d84TMqLiRxsvJ5nE8ePH0q233kptaRj1ELyNr7dt20rPPfe8gLtRo0ZSWxpG06xZt5pZ06tM2tY3o9c2T2cGyQJYzM1ntMgjNHG90sehvvetEikewD/gDZuIgmuwE0hKeWAePHhIyKNypn2R8Rs0aKBUB6MrEeL16tVbWM0D+/dJadiVV0ySKmIUjmzavJkG9O8v1UJOe6YIBgC9mCZ2DLr30mAtiJC8bAsLQV5cMxfG+yX4Bd3XOwsPMFPhP6TXIELJyTVXV29loufjohhtbVCkf/3Xn9APfvADVWLJF2RMFGBCP1kzsFFuD8hfCZ5A2Ea7MAaaLB4VmqpgabhupIxzUkgqcw59nW6PwpGBA/vJsRCRoPADIBDhKKp+16/fQBs3rmPrpOVieNrYy6+8LM8WeP75551wNtGW+3yw5e4WkAna0rG7IBaKkR5FFy6PZA0tyEtHChlKPv1Djx0vw2ZzAzlqS7MjP3IlIXxvqXMbHj30nQflWXxnwyX81V/9Fa1bt4ExwgC1jKE7OLIGCGuOACM2oLyUfIVmriLIJOT6YxcZXzdKzoNoShgqeupFgHV1WqsB04+JIRD0nXd+mRH+KLrxxhsliVV7oo727T8gNPH4S8fLubdv305vchjoPmVEesTzanzzFMoIB4BRspMM41p0y2QElKzaMcvAJJZCdesBrQWITqmdRVbgLotYLNJoXoPgY+HHFiY0o9CGUrj26u1bZAEn1AG0tcEarF+/lr76ta9SdO2mriCMsnahgEa4hYaGk5KwQX9poYYLim2/BMoqSh0iGbYxa0Z6Z3EdWBUEtZt4cMQqBqgvv/wKLVv2Hu1loZeVlzIALKX27bTgFOBv4mWXSQSQeghlzfe///3lVmrV7jc9e9rHk9lwzca35i9wrINdDyDhz9ECZ5d4/9D+b7FAG5E/AB/+dIauC6yMIphVFaJl4jjdWl29nb74xb+kb36zVc9ZKmjf+9736N/+7d/EJ2vhqCCBaASHJksUmFpBVJXZKew6ayiIqpnRNRUsPJh6GT6BgkD49rVrVwuww6xkfa0R2rehoV4UATyA1h5W0s4du2jNqtVCDp04dpyP2S592WL5fXOBr7rfDBgwwFw4RTSlNhe4GT+WWGrdCtMFgW5z3EQ0Jy7joOSWRQHfeehh4/MN/ojQNVFiybgCskmB6j//8/9llzBElnNra/vsZz/DLmEdK8K/06CBgygePHqfYFhRa4g0MEaz1haqkgpEMBlRuAikmmUqeFQfqIU6w4YNFdmAE8BoBmGFeZwTJkwU1929ew/q3q07s38oFhkq37/++mtUztlL5ALcxtclD5gSCTEaTeGAHiaDhRmsdrEDsyhy6IJAj5KPhnFjfzvFyUvsHy8Gaatq8hQ/+r35LgCj/kGM/ITiZM0hLDNnldCGqV5CMLgnWAMgaCjD2Wif+cyneaSuo6effpruuOMOpmvHiik/yaSNVudYYfuSW4CQ4RayGY1aEPZBwBo14Ii6L64Xq4Ai2YO43mb/QPX+6U9/ktnC+HzTTTdzhvM2cReIFL7ylb/m7GHv9EMncbyFtqfgKxa6XyLGLCvX8EUWeXB8dfxEzDQ55II/e+FeYl91x6njWSsR1do3r2Hke/b4JixVEGX3CBPniS2EvQ8iO32s5uhhuueb99GXv/zlaLWttraPfexj4hYWLXqLefi3JC/foUM7mSGk5lgjAoRpDVIbqNeF4hMtLjFT3k2GUZQkmxXLsXz5e8IXoMEd3H///RKiwr1gfcBf//qXsh0kEfIBWD8Y1sBtdtBLjwMIppNCPbv3oHgGqwrYAsJ45m0auFmq1/IBtrPtkuoOMWRGavQYFnlp/kra+luKfhvPlLWj37aYhtZ1iu00bI23hc0LAqkA+tnPfkYTJ048K1GC27BgdK4xR7V1x6QCKC+jXh/60K4dFKPCuFq7IIRGD5h/mA/yZhnYWklH28JPcBFIBKGId+2a9fT222+Lkvm+Jq5ee+01uTcs9NGvX7/E9fD25fYR9NGQ44M+6+4Ef6P9pSyVnasuPwlsjaAL/JKMX8wm2tW6ddkUffVS/tqCwtOtXV2suSRK1kQjlpnMpM6NE5RoeGhm2SDm1vQ3FlzQEGnXrt107bXT6eGH/w+drQZzjuqihvq8ScnCzNeKMGtrT4kQIVSdT6iLSMENBIHiGISF+bwvmclu7OPtMcHawl0vX7GMR3tXUwdwShaKRsEorMC+vfupqqoqfUmRy48UgDtlnrvHRRddrL7fDyiieD2dsBCHcKQXGLqULhnM4FNUBSRjz/zWs49lS7sKtJZwAdaaZAWpynETRFR8ntCswaMEjY4umcXLyqA1eQjZdHvHjp2EbXv44Yfok5/4s7NiDTTN61F5mZp+CLq8vIOQNWAFwev37Nlbzl9alhWgiFGPOtNOlR00c4g6Y/b7Y8eOEbKuffsKUdZjjPBRb4iFIK648jIJDTdu3MSAdC1dccWVsiCFnSRqGyveE9G12TfMGUMrEnwAsIAuSxKYjksLjih2DTHgksSIsyR76D5nN2IOXYthtoctYQM9IjcqCe0oNxGFgyd0OpdZRSSawZtldFymUSzvi5HYv38/mRl09Ohx8bG/f+E5mjHjesmlt7VpClhJHEzdxnVjAW7wA1jmDWVm8NMnjtdR126dxW2g5uCKKybT8JEjhN+H1Vq4cKHwNPZxMVCirVvfZ5awL23csFmAYGVlJ2EHMc0fAxHvnVbDLOZC+yHqpUceeQTCT7mBIToxMRHyxSGeZx/yxB2MxEVkAUL3UW/p8JAcQTtAMZFMalaXRtSJtrxRiYzU5YfmvBGR5ZkkTFTLl5cRJQszyXdYduWYKAniatTug6vBbOmPfOQjxKQJtbahyLNDR338O4AZQjYIEImarl27yErf8N/9+vUhnR4eyvaLL76YXlv4Kq1bu0GprIwWmmJwgmTCb6BEiC7+9Kc/0s6d23n0bxRA2I//du/ZzQp0ReJa0pY+Dbt/5n646KKL5OKVR06Gg+bWSF1AGDFbkt93ljyL/S+R51DBBeAxbBkPIORUlJsgZeDkCaCBKb7QqV3inkK7iocX3bYqrERA7FvL5ffwsVgACvdjl8dH/h37f+tb32Jkf0srXULI4eAlNJB9cd3JU3T40EFZyg0jH1YVsXxV1WBZmUWmjbECHDp0SD7D6HZmF4Hv+/TuQ8OHD5XKIGT+0LBfz57dZYHKw4drWMG6SSHqYfb/H//4J6KlYuJ+8xKDPKEAxjQk3ABKiuGTbOWO78cLRemadfFqViiYjFO91vxbXxzGiN/BC9GFtTAZpMSkVSxf191PPIEEdGyjKkYKW8A6AfRZM4oRg6OcOHFcOHrcI0qyUJixa9cOEiKHt7+04CVZHxAxfksahLxl81bavXsHHTt+VJZyRU4C+AMjGQqA1xkzbmQr0V2uHcANkz5xP6hDuOii0dSuolwQPZJCUAjU+4OOxvFAAuFe6pluRkYXCowKIFiJ+L69amYtT2sB0B51P6CiVBcr1NBJV99U060LIqoIRCjgsX0tdrCC9hLIPs4RxJZBTXVLk0Huo1hDz6zG6VoVoH0UU4S6t0YGcdYSa+jgVmDqt23byQCtnP2pFleEpmQbqVuMSCg1YuzKLkyx7twj5Mqdd95VbN2dog19tn//btq1YzcNHzpczg3hoEwceXxwBvDvq1atlFBv/PiJUtwBxcCydL169+AwbzFdffU0YRHXrFnL5n2XsIt79+2lSydMEH4AlnrUyFEyX3Do8GGsLH3Tl7IwvaFAAdI+AhqHp1JJl/PIKC+voNKS0mRuwPzpREazdp3dlhj1MfcfWqbOSRF7LVgqznIAvmNxEg7EKIVl/6JHv3nxY2EBymDlZBmXfKOUXEkxZtYC2fgP/vqkmN1ARt6TT/2cGbnR9NWvfvWMigDhwpVccsko8f+IMlDTj0rd2tqTtHjxYnr//a0yvx+AbfPmDTzaKwTQbdq0ier5fAjLUSJ+6NBhGeGNbD0+wtaoavBgYQKHcHoY8oFLmDZtGl17zXTq2iWJ/tndPVhwbekNyBBRSlMmT5kkNw5/BICETvLMA58jU2/SxGGCCCKKwZoXh/8UxpFAGO8bhs3HALGC5ePIwnNAZcT6BUm4Ef1WJ1xi9S+Z9MGKcMnFY7XiJm+vJy9l4JoM0xU5JUnD29uVtRfc89RTT0vB5de//vXUI/WSrQQFHYeOUP+BA+jmW26h1WtWC0q/8sorpMATi0Jgbj+sA8Bip04dpHgD+Xy8Aow+99zvOFLoJH4e2OXlP/6Rtm/bzpbOYyvRk5WmvSjN3r17BGOk2kJL/ritKeYFmjLdfgCx0KlTF0kyyGLGnlsASVLUoBktuyy6WxqGEQffnEuaY7v6BpIeEsNb19Hcpokka0ks4rBNjxtEuCDKBoapqiUP9Uw+g7MTLJQVYkol9PUaOK1aIbN48/kcxeXZvuyDUixM7sRoREz+05/+O82b9xt2Iz0E8IFRvPrqq2VmDubu9e7Vi/pxmImsHR74CJ9dx2Z+7do14u9R4AE2D1hj8OAqWrLkHVEspHOxsANIHfj7kSNH0m9/+1t2E+OZEl5OR5jqhdUABXz7bbezIpeyq+oqxy4i04LmURPt3nvvfYUcJYCZ+8UvfikCB7AA3Qh/FS2l4iWzfDp3LjbvMUMXOIjcZAOt4HgzHgmnKN5dYs5zhKZgsbp6M8X669QlWiXzbO2CSUrZyMRZjAoRjmbl9J66dq2kvXv2SQIMnLwNe2PQS5rJCy3pRBI5qIuop3blHWmohM5ZWs2pWKzahX6CEEE3VzIhs3r1Cnp/y1YW8hA6xIKF34YFAAfQsWMHQfcjR49ixN9fQnCsSbj0naWySOUNN17PVuA5IXjefPMNcUvIEsKdTJ9+Lb216E2pBRjI2UiAzEjIDP7Ysg+mIq1J6D1lypRt/PJ5+xlsFQALMkwwfRgZtlPceDz5uFa7XQWUDAPtXkZwEtKV0BH2w2Cz9Jl7NZyoOSKvR2uO6eeaI+aZPI6gPSvoGBjGzQGk2N0PoqvAJM88YxZk5EqyZQK88kG81LvvU1TgqSSYLhNXIkkZ8wwgFGxmynXlbuYVQOCU8HuUbF/Ccfyhw4eoN2MohGPAR1CIpe8u4/0zsrBDjx69hAFECfe4cZfKVK7u3btxtHBMavzWrVtPM26YIZEYsAcwAYpBgF0mT54s2AEZwd2798hagHhySZHKn2+89dZbiYzvGRWAf1DNSjCd31bZbUg5YmUKrF+XkzVzMoKcYQnihZLTlb4eJUqeiCg54cRJ19r6erJuxI5+cn7vEyWWhDfH9uIFo5NVRvFvtaI23g6ELw92kGVddOEmuQbPCt6uAWyXaNdrk/n8AhazLGxVliDU2b0NjY2kK3bXCCDDusJI3/76V7+SX2NpN9TyAdjBNcCCYF1/vC5a/BZd95HrqGOHjjLIsny9ndiXz+fwc9y4sbRg/gIR8rBhI4T9Q6kXZABM8LGbZzKNXCbWKJZFNPq/QE20M8HuhN9AvAw/VFdXL34PKUpw0RoBWE/smHuyI9MTzKBVwVlKMoPWQpAJ23JR5GBDRVUKO09e6+3iUjW7YnbWIYbSiSid74iCDLcaGQKwC0zJk8zCnMl/6HVLzkBIQnsstTKItzHRo6J9GQurk3DxmFCTy+uTPU+e1PUAR40cTcOHjaQunStpxLBRgvhRxwdhDxjYn0d4T8noYR7/8BEjBBt0ZmEO498MZRcBSwHaeNbNt9LuXXvE72O0Y20EgL1ejCtGjRohFmPJO++KWylS/FnU99t2WvalmBXo16+/WAF0HjoTmqqramgVK268tLSdMoOefeZtHOcj6aKFD9a+xmDM8vgCMD0ntezZRZn1gRAortRl2MzEC0+/00JQU4Tq5cg+zCp+IENo5s3ZpJWxEF7enNdao7yzZEFefLHCAS14AeDt1rWHKD/YPERFyMbheHCNJ0/W8QhvYNCn1K6sBspJn127dkqJFkJphHOIqAQfICt44qQAwHGXjqPf/Po3QgW/9tqrwhd06dKJVq1eIxEDQscNGzaK74c7mTbtGv67ml1BNX/Xg9xACiE9j/6/pdYqABrHlK+y/7vbfobvgZZhTroudhzPi9dHnHji36DlWNEK/QnaNTSzYu36t76nT9fQpVMyZrWQ+NGqYj+iUe6TJX5wHty4hmMUpXbjNX5CsrNpIwFTvOJXXNFkk1zWOtn5DuYKZDe7b9bU8etn4EbBDqL4WQndTjLFW1FRKufEiL711ltkyhZcQT3H7Du272S/flzm9QG5Q5CY3Ll27Vq66JKLqQfTubV1J2QSKVYCWf7eMkb/hyXxc+PMG6XKdycfY8CAQbJi2KBBgwXoLV68SPrDThF3G8vpJk5k1bRJAXAAtgLohel2GwCL+q9SMUUwibIuTV7XuM3JOni5KE2cfjCkzolXAQV5ywxYoKhRAdKnAFa5fFx5BOIGVbIYZeXlpYJD7BM1bEZScxXmAQsRig/VPYQxbxHjEU/2D60bCd1wNcY1qriaE8Hzh7QuDzNz6uTeYRkhgD2cgBk2bLiYdzy+FWb6/S2bWQF2CB6ACYcFQ/iICRs33jhTOIgX2b8PHjxIFASx/7JlyyQZd/nlV1ADu5Wd7AIwHbxq8AC2DK/Lk0mR6LFFonDNqfZgmvYt1ppFvTFQeiRdMYSFlK+5ZppQp/CV8EW2IkWmMxkBYLaqnMg3qeIwjLgELGSIukNJzIRuIkmXVkWplMw5MPPkQlM2hfOh8EFHrBGRzNW2zJ1VhKyxAnbalnPbkcLodepyrOYhDebhkr55wIW1KDZkxPH79RtgFl/yhafvwiHknj17xbxDSRDWATRjIieEjlDtmquvoe5duzM2GCnr/MFVgAd46skn2QUcFwyw5O0lkvQZN2683APcAoSdZYvqs6K9+MJ8Oe5HP/pReuedJcJeghtwG2TFRNAj1IzWrAwMU5Wn+IJRRfp5uw0dght7//33xR8jfIFAO3XuwNtrxUrg+759ewvShpUwF0fWzw4aWCUzXmRxxLwXLYUOYISQTD4bVs+icI3d9fEycEHI20NZtBQ7FqjiBhMXeGHsG+2q5KFZxlXW2dcFmys5VBMAJ493SSqrKku83Brm6aFAExFBx44VUnsHUzx8+Ah69913+LutAg4xKwkovU+ffnxPx8R3I2WLWb2o5kWEMIG5fPTRkiVLxJ3gD2sS2ZU98YSynqwU+9m6YC2AsWPHy7VhsuiECeOpCIF6+9///d+vp7OlAGgAhBx3duFOmGS3gW6EYMENSElTLn5kGkIgdBoWMYYgYf4qGQ3r8+zKmDSpiBYztmjc8z2DvHNmzfsKOQbcCbh0XQP3lOwLdIwRaUGdCkcjEEvdqtD0ad3J0nYyCqSLP8CVQMnq6+tMibbiPcl8RhxjSD2664qcsHSotUNsjwoiZN6Qpt3FZhpkDrJ3qNnDBBQs2oz7QcQEzh/74v4xSJATAAbAvcBigNmDOwEf0KNHVxb02GiGb5bvE64hy8fB+WEZMJO4o1kLyDbuh0c5q/sTamZrfvaFZNHhB9KuAKtOwP+IyWtXJtU0vXv3obh4hGRUIW7WZVAD0Xa8hymzU5a78w1r55aJcGBB7OIHuEyET/K7QMuj9NFogVlo2Ys4erUSmWg+QxCoAsXsY0wUQVnhZnRpuIzhCGyOQZsNSTUBpgswtmvXXq4VnP3YsRcLYq+s7Cazb2C9JLRjZQcGwEAAQOvB26BwF42+2CTXFFBDierZKrz66qsCALENo//NNxdJaRfQPawaBhkszw3XzxDFuPyKiTSMlc5tJua/m1rQWpSEhyvgqOBZ7uzP88dyOQB3HDQUqUvw2UDDSFzgAYn65Iv6SEB2VWwIXytdT5knbIai1RgpwAwAebAOupauTrFCR8Iq+F4M+CQpRSasM6t3qGCdRRrI1ixkom0YZQIwzdq5sCK5XIMQQZ4XRLE0rrFduwpW7koJ82AppnBiDAydXZUb/P24ceNIgWuWCZqt4gaWvbdUrAQmcqxbt1EyjBB2125dRTmRFcREkvZ8v6ij2Fq9jUYyjsLyL68ufJV9/Ex59CsYUTCwI0cOl2XkYTH6c5oXlieVPKvh808+E+pvkwKg4QRTp07FM2Wihw+iM3FDqFcDsAnMdCYUPHhSaVMqqBgs1uHDRwQF6xr8mo6FG4EyQKB2SXQIRaYyG87dMnJ4Vh6mSUFxgMI7d+4oSNz34mhBjY/OqrXhY+QCMOXaLJysxar6OxRjwkrhulF0CVMOgZeXl4gLs4+ew72BuwcdjXw7ANgrr7wsAlm69F1B5cjGDRs6QjJ3GBiorJo16xYZAOANQO/ClyMMvO22WZLShRLs3LVDlAKsH9b0g3W89NLxkofBoELBCqqSQMsHQUH53N8y6p9PLWytmpMNXjmNB3DjUALMtEEVUfv2HaPFJlCzdvnll4nJRApUlkVt0JBR1+JTggajDNQyTClCG3Q4FkUcwMkNEEfHGPFCwaAo8N0w35gBow9Iyhj0z9fSvr0WSDBAg6+0I138pnngAppd6Qu/yxtqG9YKCgmXNmvWLBbmPhEgAN5Hb76ZGbndLPQRMtkDQtm5cwdz8lPY1+/idKw+4gXJGiwkAYoc9zdgQD/h/Q8cOMgKsV3AMlb7AqePSZ2LF79NgzkjCB+PdYBA8Y4efRFnYfuyhVkiABMDZTRHG3gqSHqSB7cH2e9/l1rRWj0pf9GiRfPZEuBpI6PsNsEBfKHIhNVyVguVKVddhYTFZgErBw4cEgGB2YIQoSC2k7B97Jjx0vm4YQgd6WcIEvHyju3b2PeNE0uBjBkwAkYp4un4WTgZsRRQLlucYl2QJnhU0RQvxIAVv+nSpYcoF0qoL710guT2IST4/K3sh+Hnj584ysmdI9F3kyZdIeYZgoPig6cAAARqB6JHMQ2STH379ROkDwUAT4ApXjbiQeX1mDFjZZQPHDBQVvIAYMR1rFrJjCvfL8591VSklodKX7gNbB8L/yvUytamVRmYiFjAHYrVRaLVh3r07CGlU3V1x8VvY8kzMFbr2Hdh5ID+3Lp1m9wUBIPKVwgFggS5g07BcqcwsQBRGF0oxEQNPIgPdBSEAuIFcTjKufS5O7A2ebPEmj7owT6WDaYYEYU+Rate6hvwHiXa2Ac8PoR78cUXaWTDeGDqVVP5mtdJVu6qq6eKOR7KBM/2Hdtk0sUmBmirmZ7FtcOcg7HLoUyb43wQNrBiEBZWWwE/gJk6uF6EgFOmTBUmFVm7o0drZLo3LAPIL4SFcKm43lmzPiY8P+r/0HfpBtDHx7iJXe8pamVrkwIYULjAPHY+WnYeYAfPrIWQlr+3go4eq+EOqmSma7CY2169+si8ehQyQEkwEweEyR7k4g1NAPR76fgJ3OHbBViiaAL7gH/HkzLgZ+ETIVBYExRhAD8gvQr+wU6hQkeCNKpjmnU0I/CRI0fJKMQ+oKFRuLFz527BK3BRAG6Xcnx+iiMXzMGr4pGOCttKjuX3siAlB8KXiGvBSIWVq6zsKqTYDlbOE3xcLNKA6AiKgQQOrAQsHY6Psi4oK5QaeYD+TCht3LhB6iDe5eQPsMDtt98uIeFbby0SMIwHWNkHP9gm6/tkMtc+/PDDe6kNrc3rsgAUIjJIKwFSxqhwnXj5RHYJaxhkZYQ0QpUtrAQZQIVpy7K2LRNI6DygbjxBo7JrZ3EBYORAicIHgnCBuUWnYHThFQQL9gMRhRi7C/PwGDGDBvWTkBQKAN8O4ARghbx5vXlgA5ZZRZwOMgX4BdamC1umyZOm0G9/M8+kumvZIuRo8pQrxCy/YpZdAZaYOPFSwSpwM8jLo1wbgkSYV80jfNKVk+ill16S6VsYzUgDH+PBgEgBCldRUS5P/MC1X3vtdbSeweGNM6+j//7vpzgKuEnYQgDhdHmXFX6xEq+WtrYvzENNKwEAEeLnMWPH0ErGBeC5MyyMO+74c9rELNoJJkPQgeg8+LcjR46K6dy1ewf17d+PlaKSDkhI2Z47ewJt2LieO+ZmidchGLgIAEBgAvhtuI0+zDxu2LBeqmJgbm+88QYZTeAUYEanTJ0sIG0871/OnVvJuAWlW0sYwY8aPYrWrlknnb5p80Y5HmbtIuybNHkyPf3kU8zI9RJA1445DNTtwwogMsGsIlTq9mEl6MHWb9v2rRKvIgdgWT5YMliu/v0HCu7A5337Dkjl9YsvviBl3wgtsawLuBONcpKA72wKH+2sKABaU0oA04Xih6ohVfKETAhvCLuCgYMGsCAuF2wAk75DfOsIuokFjHq23RxqhaY+fu2aNeI/165dLyNn6dJ3qEPH9tETMqBgHZhNg++EYsDHT59+De1l8mQAAyv4bzxmdfiI4dSfSatXOVwdyPE5snAQxNJ3l7LVydC69etp+rXXiqW46aaPygQO7Ne3Xx+q4PDtMKdwAew2sWKpRerEytGDlfRgZNHgw3sxQAXx88ILv5dRDsILrg7XAWuH0BXv9TGv9Zw5nCUCnzhxnLgNRAmwJCWp1b3PtvDRzpoCoDWlBALSmOHrzv4ZIA2uAIUUGBmoLbjmmmsEDPWX5c85t86mFjH5NgaL27ZV0+AhQ9hanBDhYnIkMMZ2jq+nTZ8upMhwBmfwnet55F/JZncPj0SMtNs+/nEmZN4T1I0qW+AHdCwUD+eAoDes38A5+PHCBFbzfnfddaekqjEfEIsvISsHFL+CrQiWXoHLgMkGlwEXh/sAXXWEowOAg/YdKhi9r5QFH2DqEV7GhBJHP3x/XbGKBysAiDLgAuQU4ILWrF3NbmiqKEwR4S/nfrzpbApfZENnuUEJGK0/wReL8HCU+x0KFjG6wX1DKQ7zqIAJBVuGKtnnnn1WKmG3M5iCiYUPRGcM4FG74KUFTKNeJHE/BNyVQdn+/XtpBXd2NXc0iiK2c6iImB0RxSc/+Sl6Z8m7Ysbv+PSnmUM4KiHk9TOup5/85CeCIdD5OAeWU0WkUsHKuYNj8OUrlguuePHF+fTXf/0VYeMWLHhJgJ99CjeEDyuAhA5YxC0c6pax6d7Gv4ebq2WXozmAUrOI4ynZD9aw/wBOHbcrl2sCqMR1IxmFkBDXla7qQajHbvD2tgK+Yu2sKwAaogMmi55J1xGgoWiyjIUOS3CcSQ9MYsBoQ/w/ZNhQOsR+/WIWIpTihz/8oYSJV8nz8UplFPXo2U3YOwDA0Wa/vn10ahdoUqBxuBnE6ldOniScAkJPPAZmGANORBVY4g1hHEYmFmTCiN23d68o2wSOCt5eslh4iI4dOnPyReldxPCYR4DrQI4fwkfJ9x/+8AfhJEDb4h6wEMRJFjisB/AJzt2RQ0TgjTHjxoqPRwg4beo0eTg1mEQoLfh9HCfdkNxBTV9bQr3TtXOiALaxEixkJcAsSzCG5XY7zFsJJ2AQL2tGrSPntt+REYGlXU+crKVFHAK9zyMOCZAaBooLFiyQoserGMQdPsLugn071r5DJ2N5NAgFGGKgcSPyiBQ215OnTJZQDhYG+fT32KSjrgDgCr4cPD7Krr505530X//5nyJUFGnACowaPVJ4eRwLAO0oWwIgcpRgr1jxnvHrx2VuHtwZwCP2BTYAeD3OYSpGfiNTwPX8N278OJnAeYoJp2FDh0uqGCAVSuzO4TOthjHOV3gQtIrha247pwqAxkqwmEf5M2lcgAbhgx/HevYQIJ6uvXbdWprJAkCoBd6gn1ntAqHYjm07ZL3bqkGDhVlEbgGmGKMHfhmxNwQE9wLLMWDQQAFl8377W5o/fz4rz1S6GgQP8+2wPBjxIJbGsmBQkrb0vWVyHEQNt3DYtmzpMtq+TYkfEFEIa1A2jrxGr969xSWgMAb8ALABzo17AmBFuAveAe4ODCNA6nHO8kGZwQ3gyR/4LSj0dAPYQ2KHR/5COsftnCsAGnABK8Kj6fwBGthAmD4oAkI8hEuTmSmDAuzkSADtPRYIKFsIEzH+K6+8IinTsWxS/8gmGIoAkw9kDYF+5jOfoVWrV9GTTz4pIRwSKkjPolBjxowZsi+yeTj+gj+8JBMw2zPKh+8FGHvj9ddlKRZYEwhPRj5q7flaZ978Udq/dz8D2m6yGhiII4SysGpwKSCkECZ27dpdYnyQTmA9oeB7OK8wjM8rU8XZxRRbvhUmn/39n58Lf1+seXSe27333judb/Lx9PMKbUPmEGTNKPaLR44eoW5M7HRi3PCznz1OM66bIf4U4SRCvDEcxoFLQGHkd77zHcEFyEgC2EEYjz/+OH3slo/J6BzMiqPUq044geAwYrNseue/+CLt5Kjix//yL/SrX/6SsixMmY3LVgJu5W1O1owZcwnjh50y0WOvEEqY6TtMLAvoYF1LISOTNaDECEWxOhdAHo4BawOLFi/Fm2wY9dwnX3BX7zgf7bxYALehsghRAncaMjjT098jlgZ7h6pi8AO/+91ztJHDu/vuu0/CrRdeeIF69+kt1gDPzkGHw/RjvhzA5F133SUCgALcdNNNIvCnnnpKWDwIBD7frtSxgSlYRB2gWgE8f/HMM7IaCKICLK+6eNFi8fNDOLsJsgrP6Xuaj5XjY0Oh6li4FskjREU4+/vf/15AHiIOKNwnP/lJKRCBxUnP2HHag6yMX2huGdfZbOddAdBMlLCQ/fATxhKMSu8DgmTr+1vMs/yyOjOWBQBiCAUoYPkQux9nEAgwN/OmmeJKQA7Bj9saBXD+MLn43bx584T7h4mG/x5sFll4e/FiAWJQHGTt4JYOsuCvuvoqsRZQGiw1/9JLf+DoYKD4/EkcYcByYBYPgCgUCqgeCofqJZh8i0nS5dpOW8j3di2qd88Vyj9Ta+m6bGe1GVLjdh7dn+fXucXcQmepeetEvdmXg0zCFG3MgAEjByHDFCPphDAPMTSKNaYzQfTEE0/Iun+PPPKICAbZuH/4h3+gl19+WUalFrH2EAEB/YNogtWAcgAjICcg6Vk+Hkw7Urwj2epgYgdC1XfffVcU0BI2GPFQQJh+KIw7PatIW0iaw19IH3A77xjgdO10iuA2ECtYCh1CBBF07733CIoHozaBRx1WAoc57tCxg1T9bNiwIQq1YN4xWmGyUXiBkYpYHqttIhePMA6WAb7+H1l57rjj04wlHpORP5AJqSVsLcA7zHt2nixOgSdzwLog6jjNSLdtIV0ggrftglIA2wAU+WUuFcEIxVovTtBITWGgU8JBx2IUgjv41J99ijZv2CQuBaN70qRJtGrVKlkfGA99wJNDf/zjH4sCvLzwFbEiSNUCSC5etEhA6R6mlUEbY2burFtukcWlO7Ll0LWFmtUW0gUmeNsuSAWwjYVSxQTLA+yTrzmTVXAbzDL88G4WGniD60EuMRYA3YsoAXPsWclkUQUQTwjjVq7kUDOjE0IQ+yNERAUuavp0QmeJM9WsWQ3FmY+a+XnL6QJtF7QCuO2ee+65jTsTZBIeKlRJF2arYUWdx9f5xIU42ou1D40CuM24iM/zH+qxx9MH2Mw8CWRA5zGgXP7AAw+cneXGz1P7UCqA2+AmGLiNZ9Q9nYVgFeJcWQgIt5qF/iq/LufzLuQoo5o+xO1DrwDFGkcT41kZoATjWVhV/DrIfK7kz5VN4Qk76wlPUsMfK9VR+55DxOUfdmEXa/8fQ79G5HHSfbcAAAAASUVORK5CYII=)
