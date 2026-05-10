---
title: 揭秘 Claude Code 前沿技巧与 Qoder CLI 日常开发实战
source_url: https://mp.weixin.qq.com/s/bM3iAIJocpmwjphgX8Wo1A
saved: 2026-05-09
tags: [ai]
---
文泊 *2026年2月28日 08:31*

![图片](https://mmbiz.qpic.cn/mmbiz_png/j7RlD5l5q1x7ga33jCOiaKFSh4licXveytXnxJjtDjSxVx1YqMzicIKicZ6oK4O3o8PqvjUZjhUCMiaIwy73DkaPgDCU1n3pJ9iaWQfHYfRw0Jpu8/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0)

还在纠结 Claude Code 的各种“黑魔法”怎么玩？

- Command、Subagent、Skills 到底有什么区别，各自适合什么场景？
- 新出来的 Programmatic Tool Calling 又是啥，真的能提升「代码质量 + 开发效率」吗？
- 因为一个工具不得不搭梯子，有没有体验接近、甚至更灵活的「平替」方案？

本次分享将带你彻底搞懂～

- Claude Code 核心能力拆解，剖析 Subagent / Skills / PTC 技术的底层原理；
- 聊聊 Agent 的设计哲学，探讨 CLI 背后关于“效率”提升的深度思考；
- 介绍 Qoder CLI 如何接管你的日常开发工作流（含真实项目示例）。

建议点赞收藏慢慢读～

一、Claude Code 核心能力剖析

Claude Code 的诞生引领了 AI Coding 领域技术范式，先后定义了 Skills、PTC 等前沿技术理念，推动了 AI Coding 技术的加速发展，包括 Curosr 在内的众多产品都有 Follow 其定义的产品设计，所以非常值得我们借鉴和学习。

**1.1 它是什么？**

Claude Code 产品初衷——更好地发挥出模型的能力！

Claude Code 具有多种产品形态：

- TUI：交互式终端，低开销占用系统资源使用 Agent；
- Headless Mode：非交互式，便于批处理脚本集成；
- Agent SDK：基于 Claude Code 进行二次开发，快速享受最前沿的 Agent 能力设计；

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

**1.2 核心能力剖析**

Claude Code 定义了非常多的概念，很多人都或多或少听过、用过，但这些概念背后蕴藏着大量技术细节，且不同的技术概念之间交错关联，常常会让用户在使用过程中产生困惑，不知道如果进行技术决策。下面将对这些概念进行依次介绍和剖析，让大家在做技术决策时更加有据可依。

> Claude Code 不光是对 AI Coding 领域，对其他任何 AI Agent 类产品开发都有非常重要的借鉴意义，这些核心能力具有普适性。

### 1.2.1 Command

通俗的理解：Command 就是一个快捷方式，将预置的一段提示词发送至对话中。

#### 1.2.1.1 如何使用

在 Cursor 产品中也支持自定义 Command，概念、使用逻辑与 Claude Code 没有任何区别，只是 GUI 和 TUI 的区别。Cursor Command 列表截图：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

Claude Code Command 列表截图：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

#### 1.2.1.2 自定义扩展

Command 本质上就是一个存放在指定位置的 Markdown 文件，文件定义了一些配置属性（FrontMatter）形式，具体包括这个 Command 的名称以及描述等。

```markdown
---name: api-documentdescription: A command that generates and maintains API documentation, OpenAPI specifications, SDKs, interactive docs, authentication guides, and versioning materials. It is invoked whenever API documentation or client library generation is needed.---

When invoked, this command performs the following tasks:
* Generate complete OpenAPI 3.0 / Swagger specifications* Create multi-language SDK client libraries with usage documentation* Build interactive API documentation with testing capabilities* Design API versioning strategies and migration guides* Write authentication setup guides covering multiple auth methods* Document error codes with explanations and troubleshooting steps* Produce code examples and common integration scenarios* Validate documentation accuracy through actual API call testing
Process guidelines:
* Document APIs during development, not afterward* Prioritize real request/response examples* Include both success and error cases* Version all documentation to maintain consistency* Test API behavior and ensure documentation correctness* Focus on developer experience with copy‑ready examples* Include curl examples and common integration workflows* Provide Postman/Insomnia collections for interactive testing
This command returns:
* Complete OpenAPI 3.0specification with all field types and validation rules* Request and response examples(success & error)* Authentication documentation(Token, OAuth, API Key, etc.)* Error code reference with troubleshooting guidance* SDK usage examples in multiple languages* Postman/Insomnia API testing collections* Versioning strategy documentation and migration guides* Integration tutorials covering common developer use cases
```

上述示例定义了一个 API 文档生成的 Command，以下表格列出各字段的作用。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

配置文件存储可以存储在如下位置，并具有不同的生效范围

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

⚠️ 请根据需要合理配置 User 级别 Command，如果配置则会对所有打开项目生效

你需要根据你日常任务，从中提取一些共性的规则，并将其整理为规范的 SOP 流程，并使用自然语言进行描述。当然你也可以借助大模型来帮助你整理，一个示例的 git-commit Command 定义如下：

整理前，经常通过手工输入发送提示词给对话：

```bash
# 提示词 1检查当前分支是否为 bugfix/xxx 格式，如果不是请从当前分支 checkout 一个新分支，然后再 commit 和 push，要求分支名称、Commit Message根据修改内容进行设置
# 提示词 2检查当前分支是否为 feature/xxx 格式，如果不是请从当前分支 checkout 一个新分支，然后再 commit 和 push，要求分支名称根据修改内容进行设置，Commit Message 中列举当前代码修改点，分点列出
```

整理后，直接使用 /git-commit 指令发起任务：

```markdown
---name: git-commitdescription: 智能 Git 工作流自动化工具，用于分支管理、提交和推送。根据变更上下文自动处理分支命名规范、提交信息格式化和推送操作。主动用于任何 Git 提交操作。tools: Bash, Read, Glob, Grep, Edit---
你是一个 Git 工作流自动化专家，专注于一致的分支策略和有意义的提交信息。
当被调用时：1. 分析当前 Git 分支和工作目录的变更2. 强制执行分支命名规范（bugfix/*、feature/*、hotfix/* 等）3. 需要时自动创建适当命名的分支4. 根据变更内容生成上下文感知的提交信息5. 安全地执行提交和推送操作6. 验证分支保护规则和命名模式
工作流程：- 始终先检查当前分支名称和状态- 分析暂存/未暂存的变更以确定变更类型（修复 bug、新功能、重构等）- 如果分支命名规范与变更类型不匹配，从当前分支创建新分支- 根据实际代码变更生成描述性分支名称- 按照约定式提交标准格式化提交信息- 执行破坏性命令前进行确认- 处理合并冲突并提供清晰的解决指导- 验证推送成功并提供远程分支信息
分支命名规则：- \`bugfix/[问题编号]-[简短描述]\` - 用于 bug 修复- \`feature/[功能名称]\` - 用于新功能- \`hotfix/[紧急问题]\` - 用于生产环境紧急修复- \`refactor/[重构范围]\` - 用于代码重构- \`docs/[文档区域]\` - 用于文档更新- \`test/[测试范围]\` - 用于测试添加/更新- \`chore/[任务描述]\` - 用于维护任务
提交信息格式：- **Bug 修复**：\`fix: [bug 修复的简洁描述]\`- **新功能**：\`feat: [功能名称]\` 后跟变更点列表
```

### 1.2.2 Subagent

Subagent 是 Claude Code 中专门用于处理特定任务的 AI Agent（有的 Subagent 也定义为处理通用任务），每个 Subagent 有自己独立的上下文窗口、系统提示词和工具权限，通过合理使用可以显著改善复杂任务的处理能力。简单理解，Claude Code 为自己的主 Agent 配置了多个“工具人”，从关注过程转变为关注结果。

#### 1.2.2.1 Subagent 作用

1）处理更长程任务

在大模型存在上下文长度限制的前提下，Claude Code 尝试将 “大任务拆小任务，把原本只能塞进一个模型上下文里的信息，拆分到多个子上下文中分别处理”，从而在整体上突破单一上下文的实际可用上限，提升能够处理任务的复杂度。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

2）提升处理效率

Claude Code 可能会同时唤起多个 Subagent 并行处理同一任务，并且将不同的 Subagent 处理结果进行汇聚和总结，从而并发提升任务处理效率。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

一个基本的示例是代码库搜索，主 Agent 自动唤起多个 Subagent 分别对不同的代码目录进行并行搜索，最后聚合不同目录里面的代码检索结果，聚合后形成最终的任务结果。

3）专业领域定制

Subagent 支持配置自己独立的提示词、工具清单，可以实现不同专业领域的定制

- 自定义提示词，通过 Markdown 描述领域提示词；
- 自定义工具清单，限制 Subagent 可以使用的工具清单。

#### 1.2.2.2 Subagent 配置定义

Claude Code 中的 Subagent 由一个配置文件定义，配置文件格式为 Markdown，定义 Subagent 的 Metadata 以及系统提示词。

以下是一个功能定位为 RESTful API 审查的 Subagent 示例，frontmatter 部分包含三个字段作用如下：

- `name` ：唯一的名称；
- `description` ：定义 Subagent 的作用，模型根据该内容选择具体的 Subagent 执行任务；
- `tools` ：定义 Subagent 可以使用的工具清单；

```diff
---name: api-reviewerdescription: Review API designs for RESTful compliance and best practices, including endpoint structures, HTTP methods, status codes, and resource naming. Evaluates REST principles and suggests improvements.tools: Read,Grep,Glob---
You are an expert API design reviewer specializing in RESTful architecture principles and best practices. Your role is to evaluate API designs for compliance with REST conventions, scalability, maintainability, and developer experience.
When reviewing APIs, you will focus on:
1. Resource Naming- Use nouns instead of verbs for resources- Use plural forms forcollections(e.g., /users not /user)- Use kebab-caseor snake_case consistently(prefer kebab-case)- Avoid CRUD verbs in URLs
2. HTTP Methods Compliance- GET: Retrieve resources(safe, idempotent)- POST: Create resources or actions- PUT: Update entire resources(idempotent)- PATCH: Partial updates(idempotent)- DELETE: Remove resources(idempotent)
3. Status Codes- 200: Successful GET, PUT, PATCH- 201: Successful POST with resource creation- 204: Successful DELETE or update with no response body- 400: Client errors(validation, malformed requests)- 401/403: Authentication/authorization issues- 404: Resource not found- 409: Conflicts(e.g., duplicate resources)- 500: Server errors
4. URL Structure- Use hierarchical URLs forrelationships(/users/123/orders)- Keep URLs short but meaningful- Use query parameters for filtering, sorting, pagination- Version APIs in URL path(/api/v1/)or headers
5. Response Format- Consistent JSON structure- Proper error message formats- Include HATEOAS links where appropriate- Standardized timestamp formats
When providing feedback:1. First identify any RESTful violations or anti-patterns2. Explain why the current design is problematic3. Provide specific recommendations for improvement4. Reference relevant REST constraints or best practices5. Consider scalability andfuture extensibility
Be thorough but constructive in your reviews. Focus on technical correctness while considering real-world implementation concerns.
```

#### 1.2.2.3 如何唤起 Subagent

Subagent 只可以通过主 Agent 进行唤起工作，也就是说通过自然语言给主 Agent 发送相关任务，具体可以有如下方式：

1）显示唤起：使用自然语言直接指定 Subagent 进行对话，示例对话内容如下：

```js
帮我使用 general-purpose subagent 进行代码审查
```

2）隐式唤起：使用自然语言直接输入任务内容，让 CLI 帮助你选择合适的 Subagent 处理任务：

```js
帮我进行代码审查
```

3）串联唤起：使用自然语言描述 Subagent 的先后执行顺序，按照编排的流程顺序进行任务处理：

```css
先使用 subagent 完成系统开发和设计，最后使用 code-review subagent 对生成代码进行审查
```

> 💡 Command 试将一段预置的提示词发送到对话当中，所以也可以在 Command 中显示指定 Subagent 之间如何工作

#### 1.2.2.4 共享上下文实现逻辑

Claude Code 主 Agent 与 Subagent 采用相互隔离的上下文，目的是避免上下文污染。

1）发起任务

主 Agent 通过一个名为 Task 的工具来调用 Subagent，调用时自动生成以下三个参数，其作用如下：

- description：描述这次调用的具体任务，主要是给 UI 上展示使用；
- prompt：给 Subagent 指派的具体任务；
- subagent\_type: 具体的 Subagent 标识；

```json
{    "description": {        "type":        "string",        "description": "A short (3-5 word) description of the task"    },    "prompt": {        "type":        "string",        "description": "The task for the agent to perform"    },    "subagent_type": {        "type":        "string",        "description": "The type of specialized agent to use for this task"    }}
```

2）返回任务结果

Subagent 执行完成后，以 prompt 参数作为第一条输入消息，并将模型生成的最后一条消息作为 Task 工具的返回结果给到主 Agent，主 Agent 并不关心 Subagent 的执行过程。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

3）共享更多的上下文

与分布式系统的设计思路一致，如果不同的 Subagent 之间需要共享更多的数据，可以使用书写文件、并传递文件路径的方式，如此可以避免模型处理过多的上下文内容，提升任务执行效果。

> 这一思路也是目前 Spec Driven 编程的主要思想

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

### 1.2.3 Skills

Skill 是 Claude Code 中将专业知识打包成可复用功能的机制，每个 Skill 包含一个 SKILL.md 文件，其中包含 Claude Code 在对应场景时读取的指令。

#### 1.2.3.1 渐进式披露

每个 Skill 本质上是一个文件夹，核心是 skill.md，里面用结构化方式描述：这个技能叫什么、解决什么任务、需要哪些步骤/脚本/资源，以及调用时应遵循的规则。

```bash
{skill-name}/├── SKILL.md          # 必需：主文件，包含 Skill 定义├── reference.md      # 可选：详细参考文档├── examples.md       # 可选：使用示例├── scripts/          # 可选：辅助脚本│   └── helper.py└── templates/        # 可选：模板文件    └── template.txt
```

Claude Code 在对话前会先读取所有 Skill 的名字和简短描述，匹配当前任务是否适合用某个 Skill；只有匹配成功时，才按需加载该 Skill 的详细说明和脚本，这就是所谓“渐进式披露”（Progressive Disclosure）。

```cs
---name: skill-namedescription: Brief description of what this Skill does and when to use itallowed-tools: Read, Write, Bash    # Optional: available tools---
# Skill Name
## InstructionsProvide clear, step-by-step guidance for Claude.
## ExamplesShow concrete examples of usingthis Skill.
```

#### 1.2.3.2 Skill 是一种 Command

Claude Code 实际上是将每个 Skill 定义为 Command，因此实际上可以在 TUI 输入框中输入对应的 Slash Command 来实现 Skill 的手工加载。从下面的截图中可以看出，Skill 的加载过程与 Command 的执行效果类似，本质上都是把 Skill 预设的提示词放到上下文中。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

> Claude Code 并没有以 Command 形式展示 Skill，但是支持在输入框中直接唤起，这说明 Claude Code 期望 Skill 的加载不需要用户关注，而是自动加载方式。

#### 1.2.3.3 Agent 可以自动加载 Skill

Claude Code 实现了一个 Skill 工具，工具的描述定义如下，其中 <available\_skills/> 标签包含了可以被加载的 Skill 名称和描述信息，模型会根据这些信息判断何时以及如何调用该工具进行 Skill 加载。

```sql
Execute a skill within the main conversation
<skills_instructions>When users ask you to perform tasks, check if any of the available skills below can help complete the task more effectively. Skills provide specialized capabilities and domain knowledge.
How to use skills:- Invoke skills usingthis tool with the skill name only(no arguments)- When you invoke a skill, you will see <command-message>The "{name}" skill is running</command-message>- The skill's prompt will expand and provide detailed instructions on how to complete the task- Examples:  - skill: "pdf" - invoke the pdf skill  - skill: "xlsx" - invoke the xlsx skill  - skill: "ms-office-suite:pdf" - invoke using fully qualified name
Important:- Only use skills listed in <available_skills> below- Do not invoke a skill that is already running- Do not use this tool for built-in CLI commands (like /help, /clear, etc.)</skills_instructions>
<available_skills>%s</available_skills>
```

#### 1.2.3.4 Skill 如何节省 Token 消耗

下面通过一个 Skill 示例来介绍这个特性是如何节省 Token 消耗的。

示例地址：https://github.com/kevinsuperme/claude-Skills/tree/main/skills/document-skills/pdf

```objectivec
pdf/├── SKILL.md         ├── forms.md     ├── reference.md       └── scripts/        ├── check_bounding_boxes.py    ├── check_fillable_fields.py    ├── convert_pdf_to_images.py    ├── create_validation_image.py    ├── extract_form_field_info.py    ├── fill_fillable_fields.py    └── fill_pdf_form_with_annotation.py
```

SKILL.md 描述文件会在 Skill 加载时学习到上下文中，从下面的文件片段第 11 行中可以看出，如果需要填写 PDF 表单，可以继续参照 forms.md 文件。

```markdown
---name: pdfdescription: Comprehensive PDF manipulation toolkit for extracting text and tables, creating new PDFs, merging/splitting documents, and handling forms. When Claude needs to fill in a PDF form or programmatically process, generate, or analyze PDF documents at scale.license: Proprietary. LICENSE.txt has complete terms---
# PDF Processing Guide
## Overview
This guide covers essential PDF processing operations using Python libraries and command-line tools. For advanced features, JavaScript libraries, and detailed examples, see reference.md. If you need to fill out a PDF form, read forms.md and follow its instructions.
……
```

继续查看 forms.md 文件，其中第 4 行给出调用 scripts/check\_fillable\_fields 脚本的说明，而这个脚本已经存在于 Skill 目录中，这样 Agent 无需思考如何完成任务，而只需根据文件内容寻找解决方案，节省了大量代码生成的过程，提升执行效果的同时降低 Token 的消耗。

```sql
**CRITICAL: You MUST complete these steps in order. Do not skip ahead to writing code.**
If you need to fill out a PDF form, first check to see if the PDF has fillable form fields. Run this script from this file's directory: \`python scripts/check_fillable_fields <file.pdf>\`, and depending on the result go to either the "Fillable fields"or"Non-fillable fields"and follow those instructions.
# Fillable fieldsIf the PDF has fillable form fields:- Run this script from this file's directory: \`python scripts/extract_form_field_info.py <input.pdf> <field_info.json>\`. It will create a JSON file with a list of fields in this format:……
```

### 1.2.4 Hooks

Hooks 是 Claude Code 提供的接入 Agent 推理循环过程的一种能力，可以支持在 Claude Code 生命周期中的不同阶段执行用户配置的脚本。Hooks 为 Claude Code 的行为提供了可预测的确定性，确保某些操作一定会发生，而不是依赖于让大模型自行决定是否运行这些操作。一些示例用法如下：

- Agent 操作审计
- 用户输入改写
- 工具执行权限确认
- Agent 任务完成通知
- ……

#### 1.2.4.1 Hook 示例

1）读取 Agent 执行信息

以下是一个 PostToolUse 类型 Hook，并设置在 Edit 或者 Write 工具调用后，执行一段 Shell 命令。

```powershell
{  "hooks": {    "PostToolUse": [      {        "matcher": "Edit|Write",        "hooks": [          {            "type": "command",            "command": "jq -r '.tool_input.file_path' | { read file_path; if echo \"$file_path\" | grep -q '\\.ts$'; then npx prettier --write \"$file_path\"; fi; }"          }        ]      }    ]  }}
```

Shell 命令从标准输入中获取信息，并利用 jq 命令完成工具调用参数 filePath 的读取。Claude Code 会将对标准输入中发送一个 JSON 字符串，如下是一个示例的 Hook 输入信息。

```json
{  "session_id": "abc123",  "transcript_path": "/Users/.../.claude/projects/.../00893aaf-19fa-41d2-8238-13269b9b3ca0.jsonl",  "cwd": "/Users/...",  "permission_mode": "default",  "hook_event_name": "PostToolUse",  "tool_name": "Write",  "tool_input": {    "file_path": "/path/to/file.txt",    "content": "file content"  },  "tool_response": {    "filePath": "/path/to/file.txt",    "success": true  },  "tool_use_id": "toolu_01ABC123..."}
```

2）干预 Agent 执行结果

以上示例并没有对 Agent 执行过程产生影响，而实际上可以通过 Hook 干预 Agent 执行过程，只需要在标准输出中打印一个结果字符串即可，如下是一个改写用户输入内容 Hook 结果。

- decision 字段 block、undefined 表示拒绝本次用户请求
- reason：用于 TUI 展示
- hookSpecificOutput：设置输出，不同的 Hook 不一样

```javascript
{  "decision": "block" | undefined,  "reason": "Explanation for decision",  "hookSpecificOutput": {    "hookEventName": "UserPromptSubmit",    "additionalContext": "My additional context here"  }}
```

#### 1.2.4.2 内置 Hook 清单

Claude Code 为多个“生命周期事件”提供 Hook 入口，用户可以在这些事件上添加脚本调用设置。核心是围绕「用户输入 → 工具调用 → Claude 输出与结束」这条链路提供若干标准事件点，目前主要包括如下 8 个关键 Hook 事件点。

| 事件名 | 触发阶段 | 能力重点 | 常见用途示例 |
| --- | --- | --- | --- |
| UserPromptSubmit | 用户输入 → Claude 接收之前 | 过滤/增强用户提示 | 注入上下文、屏蔽敏感 prompt、打日志 |
| SessionStart | 新会话/agent 创建时 | 会话级初始化，追加上下文 | 初始化环境、加载配置、为会话追加说明性 context |
| PreToolUse | 工具执行之前 | 能阻止工具执行、控制权限 | 安全闸门、参数校验、预备环境 |
| PostToolUse | 工具执行之后 | 评估结果、向 Claude 反馈 | 自动格式化/测试、对坏结果打回 |
| Notification | 发送系统通知时 | 改写/转发通知 | 推送到 Slack/邮件/系统通知 |
| PreCompact | 会话历史即将被 compact 之前 | 影响 compact 策略、追加 compact 指令 | 手动 /compact 或自动 compact 前，告诉 Claude 要优先保留哪些内容 |
| Stop | Claude 准备结束本轮工作时 | 可阻止停止、要求继续 | 检查任务是否完成，智能“别急着停” |
| SubagentStop | 子代理准备结束时 | 精细控制单个 subagent 的停止 | 多代理协作中的子任务验收 |
| SessionEnd | 整个会话结束、进程退出前（在 Stop 之后） | 无法阻止结束，但可做清理和收尾 | 持久化会话统计、写审计日志、清理临时资源 |

更详细的配置可以参考官方文档：https://code.claude.com/docs/en/hooks-guide

#### 1.2.4.3 外部集成场景实例

Hooks 的外部集成价值，核心在于“把 Claude Code 变成你现有工程体系的一等公民”，是 Claude Code 从单机走向分布式的关键，让 AI 编码不再是一个孤立的聊天工具，而是可以被监控、审计、触发和编排的自动化节点。

1\. 接入现有工程流水线

- 在工具执行后自动跑格式化、lint、测试，把本地“工程规范”变成 Claude 必须遵守的步骤。
- 在阶段结束时触发 CI、生成变更报告、记录审计日志，让 Claude 的修改直接进入团队已有流程。

2\. 打通外部服务与协作平台

- 将 Claude 的关键事件（通知、任务完成等）推送到 Slack、飞书、邮件或告警系统。
- 在 Issue/PR 系统中自动创建或更新任务，把 AI 输出直接串入代码评审和项目管理体系。

3\. 统一安全与合规控制层

- 在执行命令或访问外部服务前集中做安全/权限校验，阻止高危操作、控制数据流向。
- 对即将流出到日志、监控、业务系统的数据做脱敏和过滤，把 Claude 纳入既有安全与合规框架，而不是旁路系统。

在实际工程团队里，Claude Code 很少是“单机玩具”，而是需要嵌入到已有的开发流程、协作平台和安全体系中运转，Hooks 正是承担这个“粘合层”的关键机制：通过在关键生命周期节点自动调用外部脚本或服务，把 AI 的每一步行为暴露给你的 CI/CD、通知系统、安全网关和度量平台，从而让 AI 编码既能被自动驱动，也能被严格约束和观测，而不是游离在工程体系之外。

**1.3 技术对比与选型决策**

Hooks 能力相对容易理解，对于外部系统集成是第一选择。

但是，很多人在使用 Command、Subagent、Skills 这些新特性存在使用困惑，比如：

- 代码提交功能既可以用 git-commter Subagent，也可以定义 git-commit Command
- 节省上下文占用 Subagent 和 Skills 都能达到效果
- ……

因为都是对上下文中的提示词进行管理，所以容易产生技术决策问题。

### 1.3.1 通俗理解

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

Command：

- 是 “人” 给 Agent 下达指令
- 指令通常是 “任务描述” 和 “任务要求”

Subagent：

- 主 Agent 通过 Task 工具唤起 Subagent “工具人” 工作
- Subagent 提示词配置的是 “人设描述”、“价值观”

Skills：

- 主子 Agent 进行工作的 “指导方针”

CLAUDE.md（AGENTS.md）：

- 长期记忆文件
- “价值观”、“红线”、“员工手册”

> 一些 Story 来辅助理解相互之间的关系：
> 
> 用户通过 Command 让 Subagent 加载 Skill 完成某个任务
> 
> Subagent 自动判断一个任务需要加载特定 Skill 来完成
> 
> 多个 Subagent 都可以加载 Skill，如果 Skill 不够通用可以直接设置给 Subagent常用的 Agent 执行准则可以放到 AGENTS.md 当中

### 1.3.2 特性对比

对 Command、Subagent 以及 Skill 三种特性的功能项对比

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

**1.4 对比 Cursor Rules**

Cursor 提供了 Command、Subagent 相关能力，这部分能力类似。

Curosr 还提供了 Rules 配置能力，并且支持设置 Rule 的 4 种使用方法，4 种方法分别对应不同的加载方式。从功能上来看与 Claude Code 有所差异，但本质上只是概念设计上的差异。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

- Always Apply：CLAUDE.md 记忆文件
- Apply to Specific Files：CLAUDE.md 记忆文件
- Apply Intelligently：模型动态加载，Skills 能力
- Apply Manually：@文件引用

从这个设计来看，Claude Code 相比 Cursor 的设计概念上更加清晰，更容易被理解。

**1.5 高度可扩展的产品架构**

Claude Code 通过定义各种抽象资源，并定义其形态为配置文件，通过三级配置设计，实现一个高度可扩展、可管控的的产品架构

- 行为说明：用 CLAUDE.md 之类的文档定义项目级、用户级规则和上下文，相当于给模型一个统一的「说明书接口」。
- 配置层级：支持用户级、项目级等配置层次，让不同作用域的配置文件叠加生效，有点像「点文件 + 项目配置」的组合。
- 命令与工具：甚至 Slash 命令、工具行为等也可以通过文件/文档约定，而不是硬编码在 UI 里。

这种做法的本质是：尽量让「如何工作」都体现在可读、可编辑的文本/配置中，用文档作为统一的契约与控制面。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

二、Claude 前沿工具调用技术

Anthropic 公司在 11.24 发布了一篇博文，一口气发布了三个 beta 特性，重点介绍更高效的工具调用方法。

https://www.anthropic.com/engineering/advanced-tool-use

三个特性分别是：

- Tool Search Tool
- Programatic Tool Calling
- Tool use Examples

**2.1 技术背景介绍**

大模型上下文窗口大小有限，即使是超大的窗口也很难满足日益增长的工具调用诉求，数十个 MCP 工具即有可能把常见的 200K 上下文占满，同时带来其他各方面的技术问题。

- 用户可用上下文减少，对话轮次骤减
- 工具过多，模型识别工具调用的准确性降低了，参数出错概率高
- 上下文占用过大，对话效果降低

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

导致上下文快速膨胀的的两种原因：

- 预加载的工具数量过多
- 工具调用结果过大

因此，Anthropic 针对这两种情况对工具调用逻辑进行优化，提出了三种特性能力：

- Tool Search Tool
- Programatic Tool Calling
- Tool Use Examples

**2.2 Tool Search Tool**

传统模型调用时，需要传递所有的工具清单，直接占用大量的上下文空间，Tool Search Tool 和 Skill Tool 动态加载 Skill 的逻辑类似，工具也可以使用该工具按需动态加载。

> Tool Search Tool, which allows Claude to use search tools to access thousands of tools without consuming its context window

```javascript
{  "tools": [    // Include a tool search tool (regex, BM25, or custom)    {"type": "tool_search_tool_regex_20251119", "name": "tool_search_tool_regex"},
    // Mark tools for on-demand discovery    {      "name": "github.createPullRequest",      "description": "Create a pull request",      "input_schema": {...},      "defer_loading": true    }    // ... hundreds more deferred tools with defer_loading: true  ]}
```

上下文窗口占用情况示例：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

**2.3 Programmatic Tool Calling**

工具调用，尤其是用户自行接入的 MCP 工具，可能会返回大量的工具结果数据，造成上下文被快速填充，导致后续对话无法进行。实际上很多工具的输出结果最好进行清洗后再给模型，比如：

- 网页爬取工具返回的网页内容，大量 CSS 等无用代码需要去除
- 统计数据时，先进行全量数据查询，然后需要对内容进行聚合计算给出结果
- ……

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

用户侧初始化的工具清单包含 code\_execution 工具（注意这是一个服务端工具），以及设定 get\_team\_members 工具可以被 code\_execution 工具调用。

```makefile
{  "tools": [    {      "type": "code_execution_20250825",      "name": "code_execution"    },    {      "name": "get_team_members",      "description": "Get all members of a department...",      "input_schema": {...},      "allowed_callers": ["code_execution_20250825"] # opt-in to programmatic tool calling    },    {      "name": "get_expenses",   ...    },    {      "name": "get_budget_by_level",  ...    }  ]}
```

第一次模型请求时，模型结合“用户需求”+“工具清单”，生成一段调用工具、处理工具结果的代码，这段代码执行的结果可以直接满足原始“用户需求”。这里需要注意的是，get\_team\_members 工具的调用是在 python 代码当中，它会要求用户侧发起工具调用、返回工具执行结果。第二次给到模型的数据已经是代码处理过后的数据了，如此避免了模型直接处理 get\_team\_members 工具返回的巨大结果。

```makefile
{  "type": "server_tool_use",  "id": "srvtoolu_abc",  "name": "code_execution",  "input": {    "code": "team = get_team_members('engineering')\n..."# the code example above  }}
```

**2.4 Tool Use Examples**

目前工具调用时，需要传递输入参数的 schema 信息，用来让模型了解每个参数的具体类型，保证模型生成的准确性。然而只提供这些信息还不足以更精确地指导模型生成调用参数，比如一个字符串可能有具体的格式要求、一个数字有数值区间要求等。

```json
{  "name": "create_ticket",  "input_schema": {    "properties": {      "title": {"type": "string"},      "priority": {"enum": ["low", "medium", "high", "critical"]},      "labels": {"type": "array", "items": {"type": "string"}},      "reporter": {        "type": "object",        "properties": {          "id": {"type": "string"},          "name": {"type": "string"},          "contact": {            "type": "object",            "properties": {              "email": {"type": "string"},              "phone": {"type": "string"}            }          }        }      },      "due_date": {"type": "string"},      "escalation": {        "type": "object",        "properties": {          "level": {"type": "integer"},          "notify_manager": {"type": "boolean"},          "sla_hours": {"type": "integer"}        }      }    },    "required": ["title"]  }}
```

Anthropic 的思路是在工具描述中添加 input\_examples 字段，用来向模型提供调用参数示例，从而指导模型给出正确的输入参数信息。

```json
{    "name": "create_ticket",    "input_schema": { /* same schema as above */ },    "input_examples": [      {        "title": "Login page returns 500 error",        "priority": "critical",        "labels": ["bug", "authentication", "production"],        "reporter": {          "id": "USR-12345",          "name": "Jane Smith",          "contact": {            "email": "jane@acme.com",            "phone": "+1-555-0123"          }        },        "due_date": "2024-11-06",        "escalation": {          "level": 2,          "notify_manager": true,          "sla_hours": 4        }      },      {        "title": "Add dark mode support",        "labels": ["feature-request", "ui"],        "reporter": {          "id": "USR-67890",          "name": "Alex Chen"      },      {        "title": "Update API documentation"      }    ]  }
```

三、一场关于“效率”提升的深度思考

Claude Code 的出现不仅仅是新产品、新技术的诞生，而是一场关于“效率”提升的深度思考，它将大家的视野从眼花缭乱的 AI 新产品拉回到 AI 应用的核心，重新思考 Agent 实现的本质是什么。

**3.1 回归 Agent 本质**

如何才能像传统 IT 基础设施一样，让 AI 应用也能够进行分层设计、通过复用让上层应用只关注业务实现，因此需要回归 Agent 本质，从 Agent 最核心的能力出发，从而能够辐射更多的 AI 应用场景。

### 3.1.1 Agent 的本质是什么

一句话概括 Agent 是什么：“能感知环境、根据目标自己做决策并采取行动的智能实体”。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

近年来市面上 AI 产品花样很多，但底层 Agent 技术的“配方”这两年变化不算颠覆，而是在同一套核心框架里不断打磨和工程化。绝大多数所谓 Agent，本质都是在大模型外面包一层相似的架构：

- 核心是大模型（LLM/多模态模型），负责理解、推理和生成。
- 再加上三件“标配”：规划（Planning）、记忆（Memory）、工具调用（Tool Use）。
- 外面再接一个编排框架/工作流（如各种 Agentic Workflow），把单次调用变成多轮闭环执行。

换句话说，现在大部分 Agent 只是在“意图理解 → 拆解任务 → 选工具 → 执行 → 再判断要不要继续”这个套路里做不同包装和场景定制，并没有跳出这个范式，这也就是所谓的 Agent Loop。

#### 3.1.1.1 主子 Agent 架构（规划）

通过主子 Agent 架构，将模型的推理过程具象化到具体的上下文空间，构建了现实世界分析任务、拆解任务以及实现任务的核心框架。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

> 当前的 Subagent 更多是工程上的权宜之计，以人工梳理 Subagent 为主；从更长远的视角看，智能系统大概率会从“人工设计 Subagent”，演化到“AI 自主按需生成 Subagent”，再进一步收敛为对外只有一个统一的通用Agent、对内却是自组织的多子系统结构。

#### 3.1.1.2 上下文管理（记忆）

Claude Code 提供多种上下文管理的机制和手段。

- CLAUDE.md

通过 Markdown 文件进行记忆设置，用于指导 AI 编码代理如何与项目交互。本质上是参考 README.md，不过 README.md 文件是为人类准备的，而 CLAUDE.md 是给 AI 看的。

- /compact

上下文压缩指令，内部通过多种压缩机制来精简超长的上下文，从而避免对话无法继续。

- System-reminder

Claude Code 内部隐形提醒机制，避免长期对话产生的记忆遗忘等问题。

#### 3.1.1.3 持续的工具优化（工具调用）

Claude Code 内置的工具一直在持续演进，包括新增的 Skill 等工具来提供全新的 Agent 能力，也有被下架的 LS 等内置工具，工具是 AI 连接外部世界的桥梁，合理的工具设计、完善的工具监测体系，是 Agent 能力提升的必要条件。

### 3.1.2 一切旁路皆 Hook

从 Claude Code 研发方的视角看，Hooks 其实更像是“把内部旁路能力产品化”的机制，而不仅仅是为了对外集成好用。在真实的服务端实现里，很多绕不开的需求本来就需要旁路逻辑：

- 各种 Workaround：针对模型 Bug、特定命令的危险边界、少数用户环境的兼容补丁，都不适合直接硬写进主流程逻辑，而是挂在某些关键点“拦一手”再决定怎么修。
- 监控与审计：细粒度记录“模型调用了什么工具、改了哪些文件、执行结果怎样”，以及在异常路径（工具失败、长时间无响应）时打点、告警，这些也天然是旁路链路，不应该污染主业务代码。
- 针对特定租户/场景的行为调整：例如某些大客户需要额外的安全检查、额外的日志字段或特定的 stop 策略，这类客制逻辑如果直接写在内核里，会让主代码极其复杂，而通过 hook 则可以“外挂”进去。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

1）内部实现依赖

随着模型能力的不断提升，很多以前因为模型能力不足而添加的 Workaround 逻辑都会随之而消失，保证主链路的清晰才能紧跟模型迭代的步伐，Agent 在未来的发展中更具竞争力。

2）AI 演进的牺牲品

现在 Hardcode 的代码未来都可能被 AI 替代掉，它们都是固定的 SOP，只要 AI 能够生成代码，即可完成能力的自我实现。

**3.2 加速迭代效率**

Claude Code 这一产品形态的诞生，加速了 AI Agent 类产品的开发效率，包括它自身。

### 3.2.1 更容易被集成

除了 TUI 的交互式使用模式，Claude Code 还提供了 Headless Mode、Agent SDK 两种集成方式，个人或者企业能够基于其进行二次开发，同时享受最先进的 Agent 能力演进。

1）Headless Mode

```nginx
claude --output-format=stream-json -p 'your_task_description'
```

Headless Mode 是一种无界面、非交互式的运行模式，适用于自动化脚本、持续集成/持续部署（CI/CD）流水线或后台批处理任务等场景。用户可以通过命令行参数、配置文件或环境变量传入输入内容，如代码片段或问题描述，系统会直接返回处理结果，无需人工干预，从而便于集成到开发流程中实现高效、自动化的代码辅助。

2）Agent SDK

Agent SDK 则为开发者提供了更深层次的集成能力。通过提供多种编程语言的软件开发工具包，Claude Code 的核心功能可以被嵌入到自定义应用、IDE 插件、企业内部工具或智能体（Agent）系统中。该 SDK 支持上下文管理、多轮对话、自定义提示模板以及对输出结果的进一步处理，使开发者能够根据具体业务需求灵活调用和扩展其能力。这三种使用模式——TUI、Headless Mode 和 Agent SDK——共同构成了从终端交互到自动化执行再到深度系统集成的完整能力矩阵，满足不同用户和场景下的多样化需求。

### 3.2.2 更容易做评估

Headless Mode 也方便了 Claude Code 自身的评估工作，避免因为复杂的 GUI 交互，导致评估工作量的增加。

1）避免 GUI 的复杂性

首先，Headless Mode 极大简化了 Claude Code 自身的评估流程。通过摒弃图形或交互式终端界面，评估脚本能够以程序化方式直接调用模型，输入结构化测试数据并获取可解析的输出结果。这不仅避免了因GUI交互引入的操作复杂性和不确定性，还显著降低了人工干预成本，使得大规模、高频率的自动化测试成为可能。该模式支持在持续集成（CI/CD）环境中无缝运行，确保评估结果具备高度的可重复性与可靠性，从而加速模型迭代与功能验证。

2）Headless 模式催生生态建设

其次，Terminal Bench 作为专为终端形态AI工具设计的测评系统，进一步完善了这一评估生态。它提供标准化的输入输出协议、隔离且可复现的测试环境，以及涵盖任务完成率、代码正确性、响应时间等维度的结构化指标体系。这种基础设施天然适配 Headless Mode，使开发者无需人工介入即可完成端到端的基准测试。Terminal Bench 不仅降低了评估门槛，还实现了不同模型版本或竞品之间的公平比较，真正构建起“开发—集成—评估”一体化的自动化闭环。

### 3.2.3 重新定义生产关系

Claude Code 以平均每周 5 个版本的计划在持续迭代中。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

1）技术选型简单

Claude Code 采用「命令行 + TUI」的技术选型，避免了现代 IT 企业里多种岗位之间的复杂生产关系，让 Agent 在一个简单的命令行中充分演进，这能明显提升个人与团队的产出效率。

> TUI 基于文本实现，界面由字符、符号、菜单等文本元素组成，不依赖图形，极大地降低开发难度，也避免了因为 GUI 带来的各种技术实现成本，让开发的核心更加聚焦在 Agent 能力上。

2）利用 AI 迭代 AI

Anthropic 的 Claude Code 团队用 Claude Code 开发 Claude Code，本质上是一种非常激进、系统化的「dogfooding」（自己吃自己的狗粮），目的是在真实高强度场景下打磨这个代理式编码工具的能力与可靠性，同时建立了 AI、人以及系统之间的全新生产关系，加速产品开发的过程。

四、Qoder CLI 的最佳实践

Qoder CLI 是 Qoder 产品家族中的一员，与 IDE 类产品相同，它使用全球顶级模型，基础模型免费使用！

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

如何安装，一条命令即可！

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

Qoder CLI 从发布到现在，积累了众多实践场景，下面概要地进行介绍。

**4.1 Vibe Coding**

Qoder CLI 在快速原型开发场景中展现独特优势，通过特定的代码开发框架、标准 MCP，我们可以将 AI Coding 扩展到各类办公场景。

<video src="https://mpvideo.qpic.cn/0bc35ia74aab5qabh3m6nfuvd2wd73vad7qa.f10002.mp4?dis_k=83e49babc0fcc0c731eb1a8baa6b06ce&amp;dis_t=1778307139&amp;play_scene=10120&amp;auth_info=C7ipynkfaAa7nKGiCCNtOD5hSzcwFzJMBGw8EAxTdSIzTWcYEiVjUzRyFkZoYFQ3Kjc=&amp;auth_key=dac71f5f142943967e47fa2e42d4de86&amp;vid=wxv_4403957319403831298&amp;format_id=10002&amp;support_redirect=0&amp;mmversion=false" controls="">您的浏览器不支持 video 标签</video>

```nginx
qodercli mcp add chrome-devtools -- npx chrome-devtools-mcp@latest
```

通过配置如下的 Chrome MCP 工具，让 CLI 能够自动生成代码、启动并测试应用，用户无需关注代码，即可实时关注产品的开发过程。

**4.2 Quest Mode — Spec Coding**

开发者通过自然语言描述意图，CLI 理解用户意图并生成 Spec 结构化地拆解任务，然后通过 Spec 将任务委派至 CLI 进行执行。

- 充分澄清设计：Specification 对于开发者来说是最熟悉的意图表达方式，让设计文档成为人与 AI 之间的沟通媒介；
- 异步委派任务：开发者的工作变成明确任务意图、写作生成设计文档，工作模式从实时伴随进化到异步委派；

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

Qoder CLI 已支持 OpenSpec、spec-kit 开源实现

https://github.com/github/spec-kit

https://github.com/Fission-AI/OpenSpec

Qoder IDE 中的 Quest Mode 即将迎来全新版本升级！

**4.3 Code Review**

Qoder CLI 提供 Code Review 能力，支持在本地和远端运行，可以根据业务场景进行选择。

### 4.3.1 本地 Code Review

Qoder CLI 在本地提供了基于 / 命令的 Code Review 功能，只需在修改完成的代码仓库下执行 /review 命令，就会唤起 CLI 执行代码审查任务。/review 命令面向专业开发场景，使用最顶级的模型+专用 Subagent，对当前仓库未提交的代码修改进行审查。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

### 4.3.2 基于 Github、Gitlab 进行 Code Review

近期，我们发布的 Qoder CLI GitHub Action，为 Pull Request 环节引入了自动化的智能评审工作流，切实帮助开发者提升了代码质量与合并效率。与此同时，我们也收到了大量企业用户的反馈——希望在 GitLab 的工作流中也获得一致的智能评审体验，将 Qoder CLI 平稳地集成到现有的 Merge Request 流程中。

基于这些需求，我们整理并推出了一套可直接落地的官方最佳实践指南，帮助你快速在 GitLab CI 中构建自动化、深度智能的代码审查流程。

只需少量 CI 配置，你的 GitLab MR 就能具备自动触发、深度理解代码上下文并输出高质量审查意见的智能评审能力，享受 Qoder CLI 带来的智能化效率提升。

效果展示：

<video src="https://mpvideo.qpic.cn/0bc3nib4yaad6maccg44pjuvg2wdzrvahtaa.f10002.mp4?dis_k=5df0b9006ef1c67b30fcbab480e23959&amp;dis_t=1778307139&amp;play_scene=10120&amp;auth_info=T+/njZdzGT5Ru8mmowwgOj4+MUg7akY2SFVlP0pcDyB3ZU01SUYjNQQ0JxFHbGMDMSpn&amp;auth_key=4b33e185ea98323504f6006df78860d4&amp;vid=wxv_4403959785537224716&amp;format_id=10002&amp;support_redirect=0&amp;mmversion=false" controls="">您的浏览器不支持 video 标签</video>

**4.4 标准的 ACP 协议实现**

ACP 协议是一种 CLI 与 IDE 集成的协议，详见：

Agent Client Protocol（https://agentclientprotocol.com/overview/introduction）

Qoder CLI 实现了该协议标准, 通过该特性 Qoder CLI 可以被集成到任何一种实现了 ACP 协议的客户端中。

Zed IDE 适配效果：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

支持 ACP 协议的一系列编辑器（https://zed.dev/acp）：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

**4.5 环境部署、云上运维**

Qoder CLI 支持三大主流操作系统，并且可以在 容器、K8s 等沙箱环境中运行。通过一行命令即可完成快速安装，即使是在远端 ECS 环境中，我们可以直接在 WebTerminal 中完成 CLI 的安装，同时利用它进行运维操作。一些实践案例：

1）ECS 运维

一般用户：通过 CLI 进行开源环境的搭建

```javascript
参照 https://github.com/nextcloud/docker 帮我在这台机器上安装
```

开发用户：利用 CLI 在线上 ECS 上进行自然语言调试应用 BUG

```markdown
帮我看下如下错误信息，分析错误原因
**错误堆栈信息**
```

运维用户：通过 CLI 进行网络抓包分析

```js
监听8004端口的http请求，来源地址为4.4.4.4
```

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

2）Kuberntes 运维（ACK）

通过与 Kubernetes MCP 工具的集成，可以直接将 CLI 运行在远端 Pod 中，实现 Kubernetes 集群的连接，用户只需要通过自然语言进行需求描述，如“分析xx Pod为什么没有起来”，CLI 则会自动完成相关日志查询和分析，最后给出问题分析的结果。

- 诊断集群问题

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

- 修复集群问题

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

**4.6 基于 CLI 进行日志分析**

Qoder CLI 适合各类被集成场景，使用代码或者脚本都可，内部我们通过日志分析落地了一些最佳实践。

Qoder 产品的 Feedback 日志会通过 Qoder CLI 自动进行分析处理，结合用户上报的环境和配置信息，实现反馈问题的根因自动定位，极大地提升了问题处理效率。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

继续滑动看下一个

阿里云开发者

向上滑动看下一个

![kimi](data:image/png;base64,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)
