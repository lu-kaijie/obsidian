---
title: 深入理解OpenClaw技术架构与实现原理（下）
source_url: https://mp.weixin.qq.com/s/FUJEofqbK7vX-J64UX8Nkg
saved: 2026-05-09
tags: [ai]
---
踏天 *2026年3月26日 08:32*

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naJHXTfualp74OLgwX608ia0wRiaI2UibSKtCWZKXF22DAZzUA327XqWiaYSKtibTvaQ4NQKcqfn8uY9lmA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0)

阿里妹导读

本文是《 [深入理解OpenClaw技术架构与实现原理（上）](https://mp.weixin.qq.com/s?__biz=MzIzOTU0NTQ0MA==&mid=2247558933&idx=1&sn=926e244c855c44c818d83c6c7d57d4d3&scene=21#wechat_redirect) 》的续篇，主要讲述从沙箱隔离到企业级智能体演进。

三、各系统模块讲解

**3.8 SandBox 沙箱系统**

Sandbox 是 OpenClaw 的 Docker 隔离层，用于在容器中执行 AI Agent 的工具操作，而非直接在主机上运行。

### 3.8.1 核心目的

- 限制工具执行（exec、read、write、edit 等）的安全边界
- 减少模型执行意外操作时的"爆炸半径"
- 提供可配置的隔离级别

### 3.8.2 关键文件结构

```bash
src/agents/sandbox/├── types.ts          # 核心类型定义├── config.ts         # 配置合并逻辑├── context.ts        # 入口点 - 解析沙箱上下文├── docker.ts         # Docker 容器管理├── browser.ts        # 隔离浏览器容器├── tool-policy.ts    # 工具允许/拒绝策略├── validate-sandbox-security.ts  # 安全验证├── fs-bridge.ts      # 文件系统操作桥接└── prune.ts          # 容器自动清理
```

### 3.8.3 沙箱模式

| 模式 | 行为 |
| --- | --- |
| "off" | 不隔离，所有工具直接在主机运行 |
| "non-main" | 仅隔离非主会话（默认） |
| "all" | 所有会话都隔离 |

### 3.8.4 容器作用域

| 作用域 | 容器数量 |
| --- | --- |
| "session" | 每个会话一个容器（默认） |
| "agent" | 每个 Agent 一个容器 |
| "shared" | 所有会话共享一个容器 |

### 3.8.5 工作区访问权限

| 权限 | 挂载行为 |
| --- | --- |
| "none" | 完全隔离的工作区 ~/.openclaw/sandboxes |
| "ro" | 只读挂载 Agent 工作区到 /agent |
| "rw" | 读写挂载到 /workspace |

### 3.8.6 安全限制

禁止的绑定挂载：

- 系统路径：/etc、/proc、/sys、/dev、/root、/boot、/run
- Docker socket：/var/run/docker.sock
- 根文件系统：/

禁止的网络模式：

- host（绕过网络隔离）
- container:<id> （命名空间加入）

默认安全配置：

- 只读根文件系统 (readOnlyRoot: true)
- 无网络 (network: "none")
- 丢弃所有能力 (capDrop: \["ALL"\])

### 3.8.7 工具策略层级

隔离时工具过滤顺序：

1. 全局工具策略
2. Agent 特定策略
3. Sandbox 工具策略（只能进一步限制）
4. 子 Agent 策略

默认允许的工具：exec、read、write、edit、apply\_patch、image 等  
默认禁止的工具：browser、canvas、nodes、cron、gateway 及所有消息通道

### 3.8.8 配置示例

```css
{  "agents": {    "defaults": {      "sandbox": {        "mode": "non-main",        "scope": "session",        "workspaceAccess": "none",        "docker": {          "image": "openclaw-sandbox:bookworm-slim",          "network": "none",          "memory": "512m",          "cpus": 1        },        "prune": {          "idleHours": 24,          "maxAgeDays": 7        }      }    }  }}
```

### 3.8.9 CLI 命令

- openclaw sandbox list - 列出沙箱容器
- openclaw sandbox recreate - 强制重建容器
- openclaw sandbox explain - 调试当前配置

**3.9 记忆管理**

### 3.9.1 核心设计理念

OpenClaw 的记忆系统采用 "文件即真相" 的设计哲学：

- 存储介质：纯 Markdown 文件（人类可读可编辑）
- 索引机制：SQLite + 向量嵌入（机器可搜索）
- 工作模式：文件优先，索引辅助

### 3.9.2 记忆文件布局

```bash
~/.openclaw/workspace/├── MEMORY.md           # 长期记忆（精选、持久化）└── memory/    └── YYYY-MM-DD.md   # 每日记忆日志（append-only）
```

分层设计：

- MEMORY.md：长期记忆，存储决策、偏好、重要事实
- memory/YYYY-MM-DD.md：短期记忆，存储日常笔记、临时上下文

### 3.9.3 核心类型定义

```typescript
type MemorySource = "memory" | "sessions";type MemorySearchResult = {  path: string;           // 文件路径  startLine: number;      // 起始行号  endLine: number;        // 结束行号  score: number;          // 相关性得分  snippet: string;        // 文本片段（~700 字符）  source: MemorySource;   // 来源类型  citation?: string;      // 引用标注};
```

### 3.9.4 核心功能模块

**记忆索引管理器 (MemoryIndexManager)**

核心职责：

- 管理 SQLite 索引数据库
- 协调向量化搜索和全文搜索
- 监视文件变化并自动同步
- 处理 embedding 提供商

关键特性：

- 单例模式：通过缓存避免重复实例
- 混合搜索：向量 + BM25 关键词
- 自动同步：文件监视 + 定时刷新
- 会话集成：delta 更新机制

**记忆搜索工具**

- memory\_search - 语义搜索
- 基于向量嵌入的语义检索
	- 返回相关片段（路径 + 行号 + 得分）
	- 支持 MMR 去重和时间衰减
- memory\_get - 定向读取
- 按路径读取特定记忆文件
	- 支持行范围查询
	- 安全检查（防止路径穿越）

### 3.9.5 混合搜索机制

**搜索流程**

```css
用户查询  ↓关键词提取 (FTS)向量嵌入  ↓并行搜索  ├─ 向量搜索（语义相似）  └─ BM25 搜索（关键词匹配）  ↓加权融合  ↓后处理  ├─ 时间衰减  └─ MMR 去重  ↓Top-K 结果
```

**为什么需要混合搜索？**

向量搜索优势：

- 理解语义："Mac Studio gateway host" ≈ "运行 gateway 的机器"
- 容错性强：拼写错误、同义词

向量搜索劣势：

- 对精确 token 弱：ID、代码符号、错误字符串

BM25 优势：

- 精确匹配：a828e60、memorySearch.query.hybrid
- 高信号 token

**分数融合算法**

```ini
finalScore = vectorWeight * vectorScore + textWeight * textScore
```

权重归一化为 1.0，默认 vectorWeight=0.7，textWeight=0.3

### 3.9.6 后处理机制

**MMR（最大边际相关性）去重**

目的：避免返回重复或高度相似的片段

算法：

```ini
score = λ × relevance - (1-λ) × max_similarity_to_selected
```

λ 参数：

- 1.0：纯相关性（不去重）
- 0.0：最大多样性（忽略相关性）
- 默认 0.7：平衡

```markdown
查询："家庭网络设置"
无 MMR：1. memory/2026-02-10.md (0.92) ← 路由器 + VLAN2. memory/2026-02-08.md (0.89) ← 路由器 + VLAN（重复！）3. memory/network.md (0.85)
有 MMR (λ=0.7)：1. memory/2026-02-10.md (0.92) ← 路由器 + VLAN2. memory/network.md (0.85) ← 参考文档（多样）3. memory/2026-02-05.md (0.78) ← AdGuard DNS（多样）
```

**时间衰减**

目的：让近期记忆排名更高

公式：

```ini
decayedScore = score × e^(-λ × ageInDays)
```

半衰期：默认 30 天

- 今天：100%
- 7 天：84%
- 30 天：50%
- 90 天：12.5%

常青文件（不衰减）：

- MEMORY.md
- 非日期命名的文件（如 memory/projects.md）

### 3.9.7 Embedding 提供商

**支持的提供商**

1. OpenAI - text-embedding-3-small（默认）
2. Gemini - gemini-embedding-001
3. Voyage - voyage-4-large
4. Mistral - mistral-embed
5. Local - 本地模型

**自动选择逻辑**

```kotlin
if (local.modelPath 存在) return "local";if (OpenAI key 可用) return "openai";if (Gemini key 可用) return "gemini";if (Voyage key 可用) return "voyage";if (Mistral key 可用) return "mistral";return disabled;
```

**批量索引**

- OpenAI Batch API：异步处理，折扣价格
- Gemini Batch：异步 embeddings 批量端点
- 并发控制：默认 2 个并发批处理任务

### 3.9.8 索引管理

**SQLite 数据库结构**

```css
-- 文件表CREATE TABLE files (  path TEXT PRIMARY KEY,  source TEXT,  mtime INTEGER,  hash TEXT);-- 分块表CREATE TABLE chunks (  id TEXT PRIMARY KEY,  path TEXT,  startLine INTEGER,  endLine INTEGER,  text TEXT,  embedding BLOB,  source TEXT);-- 向量表CREATE VIRTUAL TABLE chunks_vec USING vec0(...);-- 全文索引CREATE VIRTUAL TABLE chunks_fts USING fts5(...);-- Embedding 缓存CREATE TABLE embedding_cache (  hash TEXT PRIMARY KEY,  embedding BLOB);
```

**索引更新策略**

触发条件：

1. 会话启动时（sync.onSessionStart）
2. 搜索前（sync.onSearch）
3. 定时刷新（可配置间隔）
4. 文件变化（watcher，防抖 1.5s）

会话索引：

- 基于 delta 阈值：100KB 字节 或 50 条消息
- 异步更新，不阻塞搜索

### 3.9.9 自动记忆刷新

**预压缩提示**

触发时机：会话接近自动压缩时

机制：

```nginx
tokenEstimate > contextWindow - reserveTokensFloor - softThresholdTokens
```

行为：

- 发送静默提示："Session nearing compaction. Store durable memories now."
- 模型可能回复（但通常 NO\_REPLY）
- 每个压缩周期仅触发一次
- 只在可写工作空间执行

**配置示例**

```css
{  agents: {    defaults: {      compaction: {        reserveTokensFloor: 20000,        memoryFlush: {          enabled: true,          softThresholdTokens: 4000,          systemPrompt: "Session nearing compaction...",          prompt: "Write any lasting notes to memory/YYYY-MM-DD.md..."        }      }    }  }}
```

### 3.9.10 最佳实践

**何时写入记忆**

- 长期记忆：决策、偏好、重要事实 → MEMORY.md
- 短期记忆：日常笔记、运行上下文 → memory/YYYY-MM-DD.md
- 用户明确要求："记住这个" → 立即写入

**配置建议**

小型语料库：

```json
{  "provider": "openai",  "query": { "hybrid": { "enabled": false } }}
```

```
大型语料库 + 每日笔记：
```

```json
{  "provider": "openai",  "remote": { "batch": { "enabled": true } },  "query": {    "hybrid": {      "enabled": true,      "mmr": { "enabled": true, "lambda": 0.7 },      "temporalDecay": { "enabled": true, "halfLifeDays": 30 }    }  }}
```

```
完全本地：
```

```json
{  "provider": "local",  "fallback": "none",  "local": { "modelPath": "hf:ggml-org/embeddinggemma-300m-qat-q8_0-GGUF" }}
```

### 3.9.11 性能优化

**Embedding 缓存**

```json
{  "cache": {    "enabled": true,    "maxEntries": 50000  }}
```

```
好处：
```
- 避免重复嵌入相同文本
- 加速增量更新
- 降低 API 成本

**sqlite-vec 加速**

```json
{  "store": {    "vector": {      "enabled": true,      "extensionPath": "/path/to/sqlite-vec"    }  }}
```

好处：

- 数据库内向量计算
- 避免加载所有 embedding 到 JS
- 查询速度提升显著

### 3.9.12 CLI 命令

```bash
# 查看状态openclaw memory status# 强制同步openclaw memory sync --force# 检查配置openclaw config get agents.defaults.memorySearch
```

OpenClaw 的记忆管理系统是一个生产级的混合记忆解决方案，核心优势：

1. 简洁直观：Markdown 文件，人类可读可编辑
2. 强大灵活：语义搜索、混合检索、多种提供商
3. 自动化：自动索引、自动刷新、预压缩保存
4. 可扩展：插件架构、实验性后端
5. 隐私优先：本地存储、支持完全本地运行

这个设计体现了现代 AI Agent 记忆管理的最佳实践：让文件成为真相，让索引成为加速器。

**3.10 Skills 模块详解**

Skills 模块位于 src/agents/skills/，是 OpenClaw Agent 能力扩展的核心模块。

### 3.10.1 核心概念

Skill = 一个封装了特定能力的 Markdown 文件 (SKILL.md)，包含：

- YAML frontmatter（元数据）
- 使用指南/命令模板

### 3.10.2 目录结构

```python
src/agents/skills/├── types.ts           # 类型定义├── config.ts          # 配置解析与过滤├── workspace.ts       # 核心加载逻辑├── frontmatter.ts     # SKILL.md 解析├── filter.ts          # 技能过滤器├── bundled-dir.ts     # 内置技能目录解析├── bundled-context.ts # 内置技能缓存├── plugin-skills.ts   # 插件技能集成├── refresh.ts         # 文件监听与版本刷新├── env-overrides.ts   # 环境变量注入├── serialize.ts       # 并发控制锁├── tools-dir.ts       # 工具目录路径└── skills-install.ts  # 技能安装器
```

### 3.10.3 Skill 加载优先级（从低到高）

```javascript
// workspace.ts:369-388extra → bundled → managed → agents-skills-personal → agents-skills-project → workspace
```

| 来源 | 路径 | 说明 |
| --- | --- | --- |
| extra | config.skills.load.extraDirs | 用户自定义目录 |
| bundled | skills/ (包内) | OpenClaw 内置技能 |
| managed | ~/.openclaw/skills | 全局管理技能 |
| agents-skills-personal | ~/.agents/skills | 个人 agents 目录 |
| agents-skills-project | /.agents/skills | 项目 agents 目录 |
| workspace | /skills | 项目技能 (最高优先) |

### 3.10.4 核心类型

```typescript
type SkillEntry = {  skill: Skill;  frontmatter: ParsedSkillFrontmatter;  metadata?: OpenClawSkillMetadata;  invocation?: SkillInvocationPolicy;};type OpenClawSkillMetadata = {  always?: boolean;  skillKey?: string;  primaryEnv?: string;  emoji?: string;  os?: string[];  requires?: {    bins?: string[];    env?: string[];    config?: string[];  };  install?: SkillInstallSpec[];};type SkillSnapshot = {  prompt: string;  skills: Array<{ name, primaryEnv, requiredEnv }>;  skillFilter?: string[];  resolvedSkills?: Skill[];  version?: number;};
```

### 3.10.5 技能过滤逻辑

```markdown
// config.ts:70-101shouldIncludeSkill() 检查:1. config.skills.entries[skillKey].enabled !== false2. bundled allowlist 检查3. 运行时资格评估:   - OS 兼容性   - 二进制依赖   - 环境变量依赖   - 配置路径检查
```

### 3.10.6 SKILL.md 结构示例

```makefile
---name: githubdescription: "GitHub operations via \`gh\` CLI..."metadata:  openclaw:    emoji: "🐙"    requires:      bins: ["gh"]    install:      - id: brew        kind: brew        formula: gh        bins: ["gh"]---# GitHub Skill...使用说明...
```

### 3.10.7 关键导出 API

```javascript
// skills.ts 导出loadWorkspaceSkillEntries()     // 加载技能条目buildWorkspaceSkillSnapshot()   // 构建快照（含 prompt）buildWorkspaceSkillsPrompt()    // 生成 Agent promptfilterWorkspaceSkillEntries()   // 过滤技能buildWorkspaceSkillCommandSpecs() // 生成命令规格syncSkillsToWorkspace()         // 同步到沙箱applySkillEnvOverrides()        // 注入环境变量
```

### 3.10.8 安装支持类型

```go
// types.ts:3-17type SkillInstallSpec = {  kind: "brew" | "node" | "go" | "uv" | "download";  formula?: string;   // brew formula  package?: string;   // npm/go/uv 包名  module?: string;    // go module  url?: string;       // download URL  // ...};
```

### 3.10.9 文件监听机制

```javascript
// refresh.tsensureSkillsWatcher() // 监听 SKILL.md 变更  → bumpSkillsSnapshotVersion() // 版本号递增  → 事件通知 listeners
```

### 3.10.10 安全机制

1. 路径安全：resolveSandboxPath() 防止路径穿越
2. 环境变量安全：sanitizeSkillEnvOverrides() 过滤危险变量
3. 代码扫描：安装前 scanDirectoryWithSummary() 检查危险模式

**3.11 Session 管理**

### 3.11.1 整体架构设计

```java
┌─────────────────────────────────────────────────────────────────┐│                        会话管理层级                              │├─────────────────────────────────────────────────────────────────┤│  Session Key(身份标识)                                          ││       ↓                                                         ││  Session Entry(会话元数据)                                      ││       ↓                                                         ││  Session Store(持久化存储)                                      ││       ↓                                                         ││  Transcript File(对话历史)                                      │└─────────────────────────────────────────────────────────────────┘
```

### 3.11.2 Session Key 设计

**1\. 会话键格式规范**

```ruby
基础格式: agent:<agentId>:<rest>
示例:- agent:main:main                    # 默认 agent 的主会话- agent:ops:work                     # ops agent 的主会话- agent:main:telegram:direct:user123 # Telegram DM 会话- agent:main:discord:group:guild789  # Discord 群组会话- agent:main:cron:daily-backup:run:uuid  # Cron 任务运行会话- agent:main:subagent:child-session  # 子代理会话
```

**2\. 会话键解析 **（session-key-utils.ts:12-32）****

```javascript
function parseAgentSessionKey(sessionKey: string | undefined | null): ParsedAgentSessionKey | null {  // 1. 规范化：小写、去空格  const raw = (sessionKey ?? "").trim().toLowerCase();  // 2. 验证最小结构 (agent:id:rest)  const parts = raw.split(":").filter(Boolean);  if (parts.length < 3 || parts[0] !== "agent") return null;  // 3. 提取 agentId 和 rest  return { agentId: parts[1], rest: parts.slice(2).join(":") };}
```

**3\. 会话类型判断**

| 类型 | 识别模式 | 示例 |
| --- | --- | --- |
| direct | 包含:direct: 或:dm: | agent:main:telegram:direct:user123 |
| group | 包含:group: | agent:main:discord:group:guild789 |
| channel | 包含:channel: | agent:main:slack:channel:C123 |
| cron | rest 以 cron: 开头 | agent:main:cron:backup |
| subagent | 包含:subagent: | agent:main:subagent:child |
| thread | 包含:thread: 或:topic: | agent:main:discord:group:123:thread:456 |

### 3.11.3 Session Entry 数据结构

核心字段(`config/sessions/types.ts` 相关)

```typescript
type SessionEntry = {  // === 身份标识 ===  sessionId: string;        // UUID，关联对话历史文件  sessionFile?: string;     // 自定义会话文件路径    // === 模型配置 ===  model?: string;           // 当前运行时模型  modelProvider?: string;   // 当前运行时提供商  modelOverride?: string;   // 用户指定的模型覆盖  providerOverride?: string;// 用户指定的提供商覆盖  contextTokens?: number;   // 上下文窗口大小    // === Token 统计 ===  totalTokens?: number;     // 总 token 数  inputTokens?: number;     // 输入 token  outputTokens?: number;    // 输出 token  totalTokensFresh?: boolean; // token 数据是否新鲜    // === 投递路由 ===  lastChannel?: string;     // 最后使用的通道  lastTo?: string;          // 最后发送目标  lastAccountId?: string;   // 最后账号 ID  lastThreadId?: string;    // 最后线程 ID  deliveryContext?: DeliveryContext; // 投递上下文    // === 会话状态 ===  updatedAt?: number;       // 最后更新时间戳  systemSent?: boolean;     // 系统消息是否已发送  abortedLastRun?: boolean; // 上次运行是否中断    // === 行为配置 ===  thinkingLevel?: string;   // 思考级别  verboseLevel?: string;    // 详细级别  reasoningLevel?: string;  // 推理级别  sendPolicy?: string;      // 发送策略    // === 群组/频道元数据 ===  chatType?: "direct" | "group" | "channel";  channel?: string;         // 来源通道  subject?: string;         // 主题/名称  groupId?: string;         // 群组 ID  groupChannel?: string;    // 频道名称  space?: string;           // 空间/工作区    // === 显示 ===  displayName?: string;     // 显示名称  label?: string;           // 用户标签    // === 线程/派生 ===  forkedFromParent?: boolean; // 是否从父会话派生  spawnedBy?: string;       // 创建来源    // === 压缩状态 ===  compactionCount?: number; // 压缩次数  memoryFlushAt?: number;   // 内存刷新时间};
```

### 3.11.4 会话生命周期管理

**1\. 会话初始化流程(**`auto-reply/reply/session.ts:165-579`)

```bash
┌─────────────────────────────────────────────────────────────────┐│                     会话初始化流程                               │├─────────────────────────────────────────────────────────────────┤│  1. 解析 Agent ID                                               ││     resolveSessionAgentId({ sessionKey, config })               ││                                                                 ││  2. 加载会话存储                                                 ││     loadSessionStore(storePath, { skipCache: true })            ││                                                                 ││  3. 检查重置触发器                                               ││     匹配 /new, /reset 等命令                                    ││                                                                 ││  4. 评估会话新鲜度                                               ││     evaluateSessionFreshness({ updatedAt, now, policy })        ││                                                                 ││  5. 处理线程派生 (可选)                                          ││     forkSessionFromParent()                                     ││                                                                 ││  6. 持久化会话文件                                               ││     resolveAndPersistSessionFile()                              ││                                                                 ││  7. 归档旧会话 (重置时)                                          ││     archiveSessionTranscripts()                                 ││                                                                 ││  8. 触发插件钩子                                                 ││     hookRunner.runSessionStart() / runSessionEnd()              │└─────────────────────────────────────────────────────────────────┘
```

**2\. 重置触发器机制** (`auto-reply/reply/session.ts:249-273`)

```javascript
// 默认重置触发器const DEFAULT_RESET_TRIGGERS = ["/new", "/reset"];
// 匹配逻辑for (const trigger of resetTriggers) {  if (trimmedBodyLower === triggerLower) {    isNewSession = true;    bodyStripped = "";  // 清空消息体    resetTriggered = true;  }  // 支持带参数的重置: "/new 你好"  if (strippedForResetLower.startsWith(triggerPrefixLower)) {    isNewSession = true;    bodyStripped = strippedForReset.slice(trigger.length).trimStart();    resetTriggered = true;  }}
```

**3\. 会话新鲜度策略** (`config/sessions/reset.ts` 相关)

```typescript
type ResetPolicy = {  direct: number;   // DM 会话过期时间 (小时)  group: number;    // 群组会话过期时间  thread: number;   // 线程会话过期时间};
// 默认策略const DEFAULT_RESET_HOURS = { direct: 24, group: 4, thread: 24 };
// 评估函数function evaluateSessionFreshness(params: {  updatedAt: number;  now: number;  policy: ResetPolicy;}): { fresh: boolean; reason?: string };
```

**4\. 线程派生机制 (**`auto-reply/reply/session.ts:123-163`)

```cs
// 当父会话 token 过多时跳过派生，避免上下文溢出const DEFAULT_PARENT_FORK_MAX_TOKENS = 100_000;
function forkSessionFromParent(params: {  parentEntry: SessionEntry;  agentId: string;  sessionsDir: string;}): { sessionId: string; sessionFile: string } | null {  // 1. 打开父会话管理器  const manager = SessionManager.open(parentSessionFile);    // 2. 创建分支会话  const sessionFile = manager.createBranchedSession(leafId);    // 3. 或创建新的派生会话文件  const header = {    type: "session",    version: CURRENT_SESSION_VERSION,    id: sessionId,    parentSession: parentSessionFile,  // 链接到父会话  };}
```

### 3.11.5 会话存储机制

**1\. 存储结构** **(**`gateway/session-utils.ts:575-618`)

```bash
~/.openclaw/├── agents/│   ├── main/sessions.json      # main agent 的会话存储│   ├── ops/sessions.json       # ops agent 的会话存储│   └── ...└── sessions.json               # 全局存储 (旧版兼容)
```

**2\. 多 Agent 存储合并(**`gateway/session-utils.ts:575-618`)

```php
function loadCombinedSessionStoreForGateway(cfg: OpenClawConfig): {  storePath: string;  store: Record<string, SessionEntry>;} {  // 模板路径: 支持每个 agent 独立存储  if (isStorePathTemplate(storeConfig)) {    for (const agentId of agentIds) {      const storePath = resolveStorePath(storeConfig, { agentId });      const store = loadSessionStore(storePath);      // 合并到 combined，以 canonicalKey 为键    }  }  // 单一文件: 所有 agent 共享存储  else {    // 直接加载，规范化键名  }}
```

**3\. 键名规范化与遗留键清理（gateway/session-utils.ts:237-260）**

```cs
// 清理大小写不一致的遗留键function pruneLegacyStoreKeys(params: {  store: Record<string, unknown>;  canonicalKey: string;  candidates: Iterable<string>;}) {  // 1. 收集所有需要删除的键  for (const candidate of candidates) {    if (candidate !== canonicalKey) {      keysToDelete.add(candidate);    }    // 2. 查找大小写变体    for (const legacyKey of findStoreKeysIgnoreCase(store, candidate)) {      if (legacyKey !== canonicalKey) {        keysToDelete.add(legacyKey);      }    }  }  // 3. 批量删除  for (const key of keysToDelete) {    delete store[key];  }}
```

### 3.11.6 会话清理机制

Cron 会话清理器策略（cron/session-reaper.ts:57-156）

```javascript
// 清理策略const DEFAULT_RETENTION_MS = 24 * 3_600_000; // 24 小时const MIN_SWEEP_INTERVAL_MS = 5 * 60_000;    // 最小清理间隔 5 分钟
async function sweepCronRunSessions(params: {  cronConfig?: CronConfig;  sessionStorePath: string;}): Promise<ReaperResult> {  // 1. 节流检查  if (now - lastSweepAtMs < MIN_SWEEP_INTERVAL_MS) {    return { swept: false, pruned: 0 };  }    // 2. 遍历存储，删除过期 cron 运行会话  for (const key of Object.keys(store)) {    if (isCronRunSessionKey(key) && entry.updatedAt < cutoff) {      delete store[key];      pruned++;    }  }    // 3. 归档关联的对话文件  archiveSessionTranscripts({ sessionId, reason: "deleted" });}
```

### 3.11.7 会话投递路由

**1\. 投递信息持久化（channels/session.ts:21-58）**

```php
async function recordInboundSession(params: {  storePath: string;  sessionKey: string;  ctx: MsgContext;  updateLastRoute?: InboundLastRouteUpdate;}): Promise<void> {  // 1. 记录入站消息元数据  await recordSessionMetaFromInbound({    storePath,    sessionKey: canonicalSessionKey,    ctx,    groupResolution,  });    // 2. 更新投递路由  await updateLastRoute({    storePath,    sessionKey: targetSessionKey,    deliveryContext: {      channel: update.channel,      to: update.to,      accountId: update.accountId,      threadId: update.threadId,    },  });}
```

**2\. 投递路由解析（auto-reply/reply/session.ts:60-87）**

```typescript
function resolveLastChannelRaw(params: {  originatingChannelRaw?: string;  persistedLastChannel?: string;  sessionKey?: string;}): string | undefined {  // 内部 webchat/系统消息不应覆盖已知的外部投递路由  if (originatingChannel === INTERNAL_MESSAGE_CHANNEL) {    // 优先使用已持久化的外部通道    if (persistedChannel && isDeliverableMessageChannel(persistedChannel)) {      return persistedChannel;    }    // 回退到 sessionKey 中编码的通道提示    if (sessionKeyChannelHint && isDeliverableMessageChannel(sessionKeyChannelHint)) {      return sessionKeyChannelHint;    }  }}
```

### 3.11.8 关键设计决策

### 1\. 大小写不敏感的键匹配所有 sessionKey 在存储和查询时统一转为小写遗留的大小写变体通过后台清理机制移除2. 跳过缓存的会话加载loadSessionStore(storePath, { skipCache: true }) 避免 Windows mtime 粒度问题导致的数据不一致3. 线程会话的父会话派生父会话 token 超过阈值时跳过派生，避免上下文溢出派生会话记录 parentSession 链接，支持追溯4. 重置时的行为继承/new 或 /reset 后保留用户设置的 thinkingLevel、verboseLevel 等清空 token 统计和压缩计数，确保干净的会话状态5. 会话归档机制重置或删除会话时，对话文件移动到.archived/ 目录避免对话历史无限累积占用磁盘空间

**3.12 自进化机制**

OpenClaw 通过以下几个核心机制实现自我更新与进化：

### 3.12.1 可修改的核心文件

工作区包含可编辑的文件，agent 可在对话中修改：

- AGENTS.md - 操作指南和行为规则
- SOUL.md - 人设、语气和边界
- MEMORY.md - 长期记忆 (经验教训、重要决定)
- memory/YYYY-MM-DD.md - 短期日常记忆

### 3.12.2 动态系统提示

每次运行时动态构建系统提示，包括：

1. 注入的工作区文件内容
2. 可用的工具列表
3. Skills 列表 (可扩展)
4. 运行时环境信息

当文件被修改后，下次会话立即反映变化。

### 3.12.3 Skills 扩展系统

- 通过 ClawHub 发现和安装新 skills
- Skills 可以教 agent 使用新工具
- 支持 workspace、managed、bundled 三种位置
- 按需加载 SKILL.md 指令

### 3.12.4 自我修改能力

Agent 内置 read/write/edit 工具，可以：

1. 更新自己的行为 - 修改 AGENTS.md 添加新规则
2. 学习经验 - 将教训写入 MEMORY.md
3. 扩展能力 - 安装新的 skills
4. 调整人设 - 更新 SOUL.md 改变语气和边界

### 3.12.5 自我更新指令

系统提示包含 OpenClaw Self-Update 部分，agent 可以运行：

- config.apply - 应用配置变更
- update.run - 执行更新流程

### 3.12.6 进化循环

```js
用户指令/反馈 → Agent 修改文件 → 下次会话加载新内容 → 行为改变 → 持续迭代
```

这种设计让 agent 能够像人类一样"学习"和"成长"，通过文件系统持久化进化成果。

**3.13 工作区与 Agent 路由**

### 3.13.1 工作区

工作区是代理的文件系统级隔离环境，包含代理的引导文件、记忆、身份和工具配置。

**核心特性**

1\. 目录结构

- 默认路径：~/.openclaw/workspace（可通过 OPENCLAW\_PROFILE 环境变量切换到 workspace-{profile}）
- 每个工作区包含以下引导文件：
- AGENTS.md - 代理行为指南
- SOUL.md - 代理核心人格
- TOOLS.md - 工具定义
- IDENTITY.md - 身份配置
- USER.md - 用户偏好
- HEARTBEAT.md - 心跳任务
- BOOTSTRAP.md - 初始引导
- MEMORY.md / memory.md - 持久记忆

2\. 安全边界

- 所有文件读取通过 openBoundaryFile 进行边界检查
- 防止路径遍历攻击
- 文件大小限制：2MB
- 缓存机制：基于 inode/dev/size/mtime 的身份验证

3\. 初始化流程

- 自动创建目录结构
- Git 仓库初始化
- 引导文件模板加载
- 入职状态跟踪（`.openclaw/workspace-state.json` ）

4\. 子代理过滤

- 子代理和定时任务会过滤掉非核心引导文件
- 只保留：AGENTS.md, TOOLS.md, SOUL.md, IDENTITY.md, USER.md

### 3.13.2 多代理路由策略

路由系统决定哪个代理处理哪个消息，是 OpenClaw 多代理架构的核心。

**路由层次结构（从高到低优先级）**

```javascript
const tiers = [  { matchedBy: "binding.peer", ... },           // 1. 精确对等匹配  { matchedBy: "binding.peer.parent", ... },    // 2. 父对等匹配（线程）  { matchedBy: "binding.guild+roles", ... },    // 3. 公会+角色匹配  { matchedBy: "binding.guild", ... },          // 4. 公会匹配  { matchedBy: "binding.team", ... },           // 5. 团队匹配  { matchedBy: "binding.account", ... },        // 6. 账户匹配  { matchedBy: "binding.channel", ... },        // 7. 频道匹配  { matchedBy: "default", ... }                 // 8. 默认代理];
```

**路由匹配规则详解**

1\. 对等绑定（binding.peer）：

最高优先级

- 匹配特定聊天对象（用户/频道/群组）
- 示例：Discord 频道 c1 绑定到代理 chan

2\. 父对等绑定（binding.peer.parent）：

用于线程继承

- 当消息来自线程但线程本身无绑定时，检查父频道绑定
- 确保 Discord 论坛主题能继承父频道路由

3\. 公会 + 角色绑定（binding.guild+roles）：

- Discord 特有
- 匹配特定公会中的特定角色成员
- 需要同时满足 `guildId` 和 `roles` 约束

4\. 公会绑定（binding.guild）：

- Discord 公会级匹配（无角色要求）
- 示例：公会 `g1` 的所有消息路由到代理 `guild`

5\. 团队绑定（binding.team）：

- Slack 特有
- 匹配 Slack 工作区（团队）

6\. 账户绑定（binding.account）：

- 匹配特定账户（非通配符 \*）
- 示例：WhatsApp Business 账户 biz 绑定到代理 b

7\. 频道绑定（binding.channel）：

- 跨账户通配符匹配（accountId: "\*"）
- 匹配所有使用该频道的账户

8\. 默认路由：

- 无任何绑定匹配时使用
- 路由到配置中的 `defaultAgentId` （默认为 `main` ）

绑定配置示例

```javascript
const bindings = [  // 精确对等绑定  {    agentId: "support",    match: {      channel: "whatsapp",      accountId: "biz",      peer: { kind: "direct", id: "+1234567890" }    }  },  // 公会绑定  {    agentId: "community",    match: {      channel: "discord",      guildId: "123456789"    }  },  // 角色绑定  {    agentId: "admin",    match: {      channel: "discord",      guildId: "123456789",      roles: ["admin", "moderator"]    }  }];
```

### 3.13.3 会话键策略

会话键决定对话历史如何隔离和持久化。

**DM 作用域类型**

```typescript
type DmScope =   | "main"                      // 所有 DM 共享一个会话  | "per-peer"                  // 每个对等有独立会话  | "per-channel-peer"          // 每个频道+对等组合独立  | "per-account-channel-peer"; // 每个账户+频道+对等组合独立
```

**会话键格式**

```css
agent:{agentId}:{mainKey}                           // 主会话agent:{agentId}:direct:{peerId}                     // per-peer DMagent:{agentId}:{channel}:direct:{peerId}           // per-channel-peer DMagent:{agentId}:{channel}:{accountId}:direct:{peerId}  // per-account-channel-peeragent:{agentId}:{channel}:{peerKind}:{peerId}       // 群组/频道agent:{agentId}:{channel}:{peerKind}:{peerId}:thread:{threadId}  // 线程
```

**身份链接**

- 允许跨平台合并会话。
- 示例：Telegram 用户 111111111 和 Discord 用户 222222222222222222 共享身份 alice。

```css
identityLinks: {  alice: ["telegram:111111111", "discord:222222222222222222"]}
```

### 3.13.4 性能优化

1\. 绑定缓存

- evaluatedBindingsCacheByCfg - WeakMap 缓存
- 按 channel + accountId 双键索引
- 最大 2000 个缓存条目

2\. 文件缓存

- 基于 inode + dev + size + mtime 的缓存键
- 避免重复读取相同文件

3\. 模板缓存

- 工作区模板预加载
- Promise 缓存防止重复编译

### 3.13.5 实际应用场景

场景 1：多租户 SaaS

```apache
binding 1: WhatsApp Business "company-a" -> agent "support-a"binding 2: WhatsApp Business "company-b" -> agent "support-b"
```

场景 2：Discord 社区管理

```apache
binding 1: Guild "123" + Role "admin" -> agent "admin-bot"binding 2: Guild "123" (no role) -> agent "community-bot"binding 3: Channel "welcome" -> agent "greeter"
```

场景 3：Slack 企业部署

```apache
binding 1: Team "T-sales" -> agent "sales-assistant"binding 2: Team "T-engineering" -> agent "tech-support"
```

场景 4：线程隔离

```nginx
Parent Channel "general" -> agent "general-bot"Thread in "general" inherits parent binding
```

### 3.13.6 关键设计决策

1. 确定性路由：相同输入总是产生相同输出，无随机性。
2. 显式优先级：层级清晰，避免隐式行为。
3. 安全边界：所有路径操作都经过验证和边界检查。
4. 性能优先：多级缓存，最小化 I/O。
5. 可扩展性：支持插件扩展路由逻辑。

这套系统实现了灵活的多代理路由，同时保证了安全性、性能和可维护性。

**3.14 Nodes**

Nodes 是 OpenClaw 的分布式设备/客户端管理架构，让 Gateway 可以远程控制和协调多个设备上的操作。

### 3.14.1 核心概念

Node = 一个可执行命令的远程客户端，例如：

- iOS/Android 手机
- macOS/Linux/Windows 主机
- 任何运行 openclaw node 的设备

### 3.14.2 核心数据结构

| 类型 | 位置 | 用途 |
| --- | --- | --- |
| NodeListNode | shared/node-list-types.ts | 节点列表展示（含 caps/commands/permissions） |
| NodeSession | gateway/node-registry.ts | 运行时 WebSocket 会话 |
| NodePairingPairedNode | infra/node-pairing.ts | 持久化配对记录（含认证 token） |

### 3.14.3 配对流程

```sql
Node                              Gateway  |-- node.pair.request -------------->|  (发起请求)  |                                    |  (加入 pending 列表)  |                                    |  |<-------- 广播 node.pair.requested --|  (通知管理员)  |                                    |  |          [管理员审批]              |  |                                    |  |<------- node.pair.approve ---------|  (生成 token)  |                                    |  |-- node.pair.verify --------------->|  (验证 token)  |                                    |  |<----------- 连接建立 --------------|  (认证成功)
```

### 3.14.4 Node Host 架构

src/node-host/ 中的代码运行在远程设备上：

```bash
┌─────────────────────────────────────────────────┐│                   Node Host                      │├─────────────────────────────────────────────────┤│  runner.ts                                      ││    - 启动 WebSocket 客户端                       ││    - 连接 Gateway                               ││    - 声明 caps/commands                         │├─────────────────────────────────────────────────┤│  invoke.ts                                      ││    - 处理 system.run/which/browser.proxy        ││    - 执行本地命令                                ││    - 管理执行审批                   │├─────────────────────────────────────────────────┤│  config.ts                                      ││    - 存储 nodeId/token/gateway 配置             ││    - ~/.openclaw/node.json                      │└─────────────────────────────────────────────────┘
```

### 3.14.5 命令系统

**命令分类（gateway/node-command-policy.ts:56）**

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

**安全策略**

1. 命令必须在 allowlist 中
2. 必须在节点声明的 commands 列表中
3. 敏感命令（camera.snap, screen.record, sms.send）需额外配置

### 3.14.6 唤醒机制

当 Node 离线时自动唤醒 (gateway/server-methods/nodes.ts:89):

```markdown
1. maybeWakeNodeWithApns()   ├── 检查 APNS 注册   ├── 发送后台唤醒通知   └── 等待重连 (3s)
2. 如果仍离线，重试一次   └── 发送后台唤醒 + 等待 (12s)
3. 如果仍离线   └── maybeSendNodeWakeNudge()       └── 发送可见提醒："OpenClaw needs a quick reopen"
```

### 3.14.7 事件处理

Node 可向 Gateway 发送事件 (gateway/server-node-events.ts:247):

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

### 3.14.8 安全机制

1. 配对认证：token + 节点 ID 双重验证
2. 命令白名单：按平台限制可用命令
3. Exec 审批：敏感命令需用户确认
4. 环境隔离：sanitizeHostExecEnv 阻止 PATH 污染
5. 输出截断：200KB 上限防止内存溢出

### 3.14.9 典型调用流程

```cpp
客户端 → Gateway.node.invoke({nodeId, command, params})          │          ├── 检查节点是否连接          │     └── 未连接 → APNS 唤醒          │          ├── 命令策略检查          │     └── node-command-policy.ts          │          ├── NodeRegistry.invoke()          │     └── WebSocket 发送 node.invoke.request          │          └── Node Host 处理                ├── invoke.ts 路由到具体 handler                ├── 执行命令 (spawn + 安全检查)                └── 返回 node.invoke.result
```

**3.15 安全策略**

### 3.15.1 核心信任模型

核心原则：OpenClaw 采用"个人助手"模型，不是多租户共享总线。

```js
单用户信任模型：┌─────────────────────────────────────┐│  一个可信操作者                      ││  ├── 一个 Gateway                   ││  │   └── 多个 Agent                 ││  └── 多个会话（sessionKey 是路由控制，不是授权边界）└─────────────────────────────────────┘
```

关键点：

- 通过 Gateway 认证的调用者 = 可信操作者
- sessionKey/session ID 只是路由控制，不是用户授权边界
- 同一 Gateway 上的操作者可以互相看到数据 → 预期行为
- 多用户场景：每个信任边界使用独立的 OS 用户/主机/Gateway

### 3.15.2 插件信任边界

插件以进程内方式加载，与 Gateway 同等权限：

```js
插件 = 可信代码├── 可以读取环境变量/文件├── 可以执行主机命令└── 与 Gateway 进程权限相同
```

安全报告必须证明边界绕过（如未认证加载、策略绕过），而不是"恶意插件执行了特权操作"。

### 3.15.3 执行沙箱默认行为

```python
agents.defaults.sandbox.mode: off  # 默认关闭沙箱tools.exec.host: sandbox          # 路由偏好，但沙箱未激活时在主机执行
```

含义：默认情况下，命令执行在主机上进行，因为操作者已受信任。

### 3.15.4 范围外事项（常见误报）

| 类型 | 说明 |
| --- | --- |
| Prompt 注入 | 除非绕过策略/认证/沙箱边界 |
| 操作者预期的本地功能 | 如 TUI 本地! shell |
| 已授权用户触发的本地操作 | 如 allowlisted 发送者运行 /export-session |
| 恶意插件行为 | 安装/启用插件 = 授予信任 |
| 多租户隔离期望 | 同一 Gateway 不提供用户间隔离 |
| ReDoS/DoS 需要可信配置输入 | 如自定义正则表达式 |
| 公网暴露 | 不建议但不是漏洞 |
| 暴露第三方凭证 | 除非能访问 OpenClaw 基础设施 |

### 3.15.5 部署假设

```bash
信任边界：├── 主机 OS/管理员边界 = 信任边界├── 能修改 ~/.openclaw 的人 = 可信操作者└── 推荐模式：一用户一主机/VPS一Gateway
```

### 3.15.6 Web 接口安全

```bash
# 推荐：仅绑定回环地址gateway.bind="loopback"  # 默认值openclaw gateway run --bind loopback
# 远程访问方案：# 1. SSH 隧道# 2. Tailscale serve/funnel# 3. 不要直接暴露到公网
```

### 3.15.7 运行时要求

- Node.js ≥ 22.12.0（包含安全补丁）
- Docker：非 root 用户、只读文件系统、最小权限

### 3.15.8 工具文件系统加固

```bash
# 推荐tools.exec.applyPatch.workspaceOnly: true  # 限制写入到工作区
# 可选tools.fs.workspaceOnly: true  # 限制所有文件操作到工作区
```

总结：OpenClaw 的安全模型基于"可信操作者"假设，核心是主机信任 + Gateway 认证，不提供多租户隔离。安全报告需要证明边界绕过，而非展示已授权/可信场景下的行为。

**3.16 配置管理**

### 3.16.1 架构概览

配置管理采用分层架构，核心模块位于 src/config/:

```bash
src/config/├── io.ts              # 配置I/O核心引擎├── paths.ts           # 路径解析策略├── validation.ts      # 多层验证框架├── zod-schema.ts      # Zod模式定义├── defaults.ts        # 运行时默认值应用├── includes.ts        # $include模块化支持├── env-substitution.ts # 环境变量替换└── types.ts           # TypeScript类型聚合
```

### 3.16.2 核心设计原则

**配置文件格式与位置**

JSON5 格式，支持注释和尾随逗号：

```kotlin
// 支持环境变量{  "models": {    "providers": {      "anthropic": {        "apiKey": "${ANTHROPIC_API_KEY}"  // 运行时替换      }    }  }}
```

路径解析优先级（paths.ts:130-183）：

1. OPENCLAW\_CONFIG\_PATH 环境变量
2. OPENCLAW\_STATE\_DIR/openclaw.json
3. ~/.openclaw/openclaw.json (新规范)
4. ~/.clawdbot/clawdbot.json (历史兼容)

**配置生命周期**

```js
加载 → 解析 → 合并 → 验证 → 应用默认值 → 缓存
```

关键实现（io.ts:682-818）：

```javascript
function loadConfig(): OpenClawConfig {  // 1. 读取原始JSON5文件  const raw = deps.fs.readFileSync(configPath, "utf-8");  const parsed = deps.json5.parse(raw);    // 2. 解析$include指令  const resolved = resolveConfigIncludesForRead(parsed, configPath, deps);    // 3. 环境变量替换  const { resolvedConfigRaw } = resolveConfigForRead(resolved, deps.env);    // 4. 验证与默认值  const validated = validateConfigObjectWithPlugins(resolvedConfigRaw);  const cfg = applyModelDefaults(applyAgentDefaults(validated.config));    // 5. 路径规范化  normalizeConfigPaths(cfg);    return cfg;}
```

### 3.16.3 模块化配置 ($include)

**设计目标**

支持大型部署的配置拆分，实现关注点分离。

```php
// openclaw.json{  "$include": [    "./models/anthropic.json5",    "./channels/telegram.json5"  ],  "gateway": {    "mode": "remote"  }}
```

**安全实现**

路径遍历防护（includes.ts:198-222）：

```javascript
// 拒绝访问配置目录外的文件if (!isPathInside(this.rootDir, normalized)) {  thrownew ConfigIncludeError(    \`Include path escapes config directory: ${includePath}\`,    includePath  );}
// 解析符号链接并重新验证const real = fs.realpathSync(normalized);if (!isPathInside(this.rootRealDir, real)) {  thrownew ConfigIncludeError(    \`Include path resolves outside config directory (symlink)\`,    includePath  );}
```

深度限制：最大嵌套层级 10 层，防止循环引用

**合并策略**

深度合并算法（includes.ts:70-85）：

```typescript
export function deepMerge(target: unknown, source: unknown): unknown {  if (Array.isArray(target) && Array.isArray(source)) {    return [...target, ...source]; // 数组连接  }  if (isPlainObject(target) && isPlainObject(source)) {    const result = { ...target };    for (const key of Object.keys(source)) {      result[key] = key in result         ? deepMerge(result[key], source[key]) // 递归合并        : source[key];    }    return result;  }  return source; // 原始类型: 后值覆盖前值}
```

### 3.16.4 验证框架

**多层验证架构**

```javascript
// 验证流水线validateConfigObjectWithPlugins(raw)  → validateConfigObjectRaw(raw)        // 1. 基础Zod验证  → findLegacyConfigIssues(raw)         // 2. 历史配置检查  → findDuplicateAgentDirs(config)      // 3. Agent目录冲突检测  → validateIdentityAvatar(config)      // 4. Avatar路径安全验证  → validatePluginSchemas(config)       // 5. 插件配置验证
```

**Zod Schema 设计**

严格模式 + 细粒度校验 (zod-schema.ts):

```php
exportconst OpenClawSchema = z.object({  // 元数据自动时间戳处理  meta: z.object({    lastTouchedAt: z.union([      z.string(),      z.number().transform((n, ctx) => {        const d = new Date(n);        if (Number.isNaN(d.getTime())) {          ctx.addIssue({ code: z.ZodIssueCode.custom, message: "Invalid timestamp" });          return z.NEVER;        }        return d.toISOString();      })    ]).optional()  }).strict().optional(),    // Gateway配置  gateway: z.object({    mode: z.union([z.literal("local"), z.literal("remote")]).optional(),    auth: z.object({      mode: z.union([        z.literal("none"),        z.literal("token"),        z.literal("password"),        z.literal("trusted-proxy")      ]).optional(),      token: z.string().optional().register(sensitive) // 敏感值标记    }).strict().optional()  }).strict().optional(),    // 跨字段验证}).superRefine((cfg, ctx) => {  // 验证broadcast中的agentId是否存在于agents.list  const agentIds = new Set(cfg.agents?.list?.map(a => a.id) ?? []);  for (const [peerId, ids] of Object.entries(cfg.broadcast ?? {})) {    for (const agentId of ids) {      if (!agentIds.has(agentId)) {        ctx.addIssue({          code: z.ZodIssueCode.custom,          path: ["broadcast", peerId],          message: \`Unknown agent id "${agentId}"\`        });      }    }  }});
```

**插件配置验证**

动态Schema加载 (validation.ts:418-438):

```php
for (const record of registry.plugins) {  const pluginId = record.id;  const entry = normalizedPlugins.entries[pluginId];    if (record.configSchema) {    const res = validateJsonSchemaValue({      schema: record.configSchema,      cacheKey: record.schemaCacheKey ?? pluginId,      value: entry?.config ?? {}    });        if (!res.ok) {      issues.push({        path: \`plugins.entries.${pluginId}.config\`,        message: \`invalid config: ${res.errors.join(", ")}\`      });    }  }}
```

### 3.16.5 环境变量处理

**双阶段替换**

读取时替换（io.ts:652-666）：

```bash
function resolveConfigForRead(resolvedIncludes, env){  // 1. 应用config.env到process.env  applyConfigEnvVars(resolvedIncludes, env);    // 2. 替换${VAR}引用  return {    resolvedConfigRaw: resolveConfigEnvVars(resolvedIncludes, env),    envSnapshotForRestore: { ...env } // 保存快照用于回写  };}
```

写入时恢复：

保持配置文件中的环境变量引用 (io.ts:1082-1109):

```javascript
// 写入前恢复原始的${VAR}引用cfgToWrite = restoreEnvVarRefs(  cfgToWrite,  parsedRes.parsed,     // 原始文件内容  envForRestore          // 读取时的环境变量快照);
```

### 3.16.6 运行时默认值

**设计原则**

不持久化默认值 - 只在加载时应用，写入时剥离。

```javascript
// 加载时应用默认值const cfg = applyModelDefaults(  applyAgentDefaults(    applySessionDefaults(validated.config)  ));
// 写入时不包含运行时默认值const stampedOutputConfig = stampConfigVersion(outputConfig);// 不调用 applyModelDefaults
```

**关键默认值**

Model 默认值（defaults.ts:213-347）：

- contextWindow: 200K tokens
- maxTokens: min(8192, contextWindow)
- cost: {input: 0, output: 0, cacheRead: 0, cacheWrite: 0}
- api: "anthropic-messages" (Anthropic provider)

Agent 默认值（defaults.ts:349-388）：

- maxConcurrent: 10
- subagents.maxConcurrent: 5

Session 默认值（defaults.ts:144-168）：

- mainKey: "main" (忽略用户配置)
- maintenance.mode: "warn"
- maintenance.pruneAfter: "30d"

### 3.16.7 缓存与性能优化

**配置缓存**

短期缓存策略（io.ts:1292-1374）：

```javascript
const DEFAULT_CONFIG_CACHE_MS = 200; // 200ms TTL
export function loadConfig(): OpenClawConfig {  // 检查运行时快照  if (runtimeConfigSnapshot) {    return runtimeConfigSnapshot;  }    // 检查缓存  const cached = configCache;  if (cached && cached.configPath === configPath && cached.expiresAt > now) {    return cached.config;  }    // 加载并更新缓存  const config = io.loadConfig();  configCache = {    configPath,    expiresAt: now + cacheMs,    config  };    return config;}
```

**禁用缓存: OPENCLAW\_DISABLE\_CONFIG\_CACHE=1**

**写入审计**

**安全审计日志 (io.ts:1178-1228):**

```javascript
const auditRecord = {  ts: new Date().toISOString(),  event: "config.write",  result: "rename" | "copy-fallback" | "failed",  configPath,  previousHash,  nextHash,  previousBytes,  nextBytes,  changedPathCount,  suspicious: [    "size-drop",           // 大小骤降50%+    "missing-meta-before-write",    "gateway-mode-removed"  ],  pid, ppid, cwd, argv};
await appendConfigWriteAuditRecord(deps, auditRecord);
```

### 3.16.8 错误处理与诊断

**友好错误消息**

DM策略错误提示 (io.ts:148-167):

```bash
function formatConfigValidationFailure(pathLabel, issueMessage){  const match = issueMessage.match(OPEN_DM_POLICY_ALLOW_FROM_RE);  if (!match) return \`Config validation failed: ${pathLabel}: ${issueMessage}\`;    return [    \`Configuration mismatch: ${policyPath} is "open", but ${allowPath} does not include "*".\`,    "",    "Fix with:",    \`  openclaw config set ${allowPath} '["*"]'\`,    "",    "Or switch policy:",    \`  openclaw config set ${policyPath} "pairing"\`  ].join("\n");}
```

**历史配置检测**

**迁移警告 (legacy.ts):**

```javascript
export function findLegacyConfigIssues(raw: unknown): LegacyConfigIssue[] {  const issues: LegacyConfigIssue[] = [];    // 检测已移除的配置项  if (raw.routing?.allowFrom) {    issues.push({      path: "routing.allowFrom",      message: "routing.allowFrom is removed. Use routing.channels.*.dm.policy instead."    });  }    // 检测废弃的命名  if (raw.gateway?.token) {    issues.push({      path: "gateway.token",      message: 'Use "gateway.auth.token" instead.'    });  }    return issues;}
```

### 3.16.9 安全特性

**原型污染防护**

Blocked Keys（ **prototype-keys.ts）：**

```typescript
exportconst BLOCKED_OBJECT_KEYS = new Set([  "__proto__",  "constructor",  "prototype"]);
export function isBlockedObjectKey(key: string): boolean {  return BLOCKED_OBJECT_KEYS.has(key);}
```

**敏感值处理**

Zod 敏感值标记（zod-schema.sensitive.ts）：

```typescript
exportconst sensitive = Symbol("sensitive");
export function redactSensitiveValues(config: unknown): unknown {  // 递归遍历配置对象  // 替换标记为sensitive的值为"[REDACTED]"}
```

**文件权限**

安全默认值（io.ts:1236-1240）：

```php
// 目录权限: 0o700 (仅所有者可访问)await fs.promises.mkdir(dir, { recursive: true, mode: 0o700 });
// 文件权限: 0o600 (仅所有者可读写)await fs.promises.writeFile(tmp, json, { encoding: "utf-8", mode: 0o600 });
```

### 3.16.10 扩展机制

**插件配置 Schema**

动态加载（plugins/config-schema.ts）

```typescript
export interface PluginConfigSchema {  id: string;  kind: "channel" | "memory" | "tool" | "skill";  configSchema?: JSONSchema;  channels?: string[];}
```

**运行时覆盖**

测试与诊断场景（runtime-overrides.ts）：

```javascript
export function applyConfigOverrides(config: OpenClawConfig): OpenClawConfig {  // 允许通过环境变量覆盖特定配置项  // 例如: OPENCLAW_OVERRIDE_gateway_mode=local  return config;}
```

### 3.16.11 关键文件索引

| 文件 | 核心职责 |
| --- | --- |
| io.ts:682-818 | 配置加载主流程 |
| includes.ts:91-279 | $include 解析器实现 |
| validation.ts:173-453 | 多层验证框架 |
| zod-schema.ts:131-813 | 完整 Schema 定义 |
| defaults.ts:129-532 | 默认值应用逻辑 |
| paths.ts:115-183 | 路径解析策略 |

四、总结展望

2025 年个人效率工具在编程、办公领域得到了极大的发展与提升，2026 年作为企业智能元年，可以预见会出来更多的企业级智能体，这些智能体将朝着更加分布式、安全可控且深度集成业务流程的方向演进。

**4.1 架构方向演进**

1\. 控制平面与执行节点的解耦 (Decoupling Control & Execution)  
企业架构将采用类似的“中枢管理 + 边缘执行”模式。总部部署中央控制平面负责权限审核、配置同步和任务路由，而执行节点则部署在员工终端、内部服务器或特定硬件设备上，确保数据在企业内网中闭环处理。

2\. 多智能体协作网络 (Multi-Agent Networking)  
企业内部将不再是单一的“万能助手”，而是由多个专业化智能体（如 HR 助手、财务助手、IT 运维助手）构成的网络。它们通过标准化的 RPC 协议或内部会话协议进行信息共享和任务接力，形成复杂的自动化流水线。

3\. 软硬件深层权限管理 (Hardware & Permission Integration)  
企业级 Agent 将深入集成到办公硬件（如会议室系统、扫码枪、PDA）。架构设计上将包含更精细的 TCC（透明度、同意和控制）策略，确保 Agent 在执行敏感操作（如调用公司摄像头或执行系统脚本）时受到严格的合规性审计。

**4.2 应用方向演进**

1\. 全渠道业务渗透 (Omnichannel Business Integration)  
企业 Agent 将无缝嵌入到所有内部协作工具中，打破“信息孤岛”。Agent 不仅能回答问题，还能直接在通讯软件中通过 Discord/Slack 动作或自动触发工作流来处理审批、请假或订单更新，实现“对话即办公”。

2\. 动态可视化协同工作区 (Live Visual Collaboration)  
在复杂的企业决策场景（如供应链管理或数据分析）中，Agent 将利用类似 Canvas 的技术提供动态可视化面板。团队成员可以与 Agent 在同一画布上进行交互式修改，Agent 实时生成数据图表或拓扑图，大幅提升决策效率。

3\. 基于沙箱的安全生产环境 (Sandboxed Production Environment)  
企业将普遍采用“隔离运行环境”。当 Agent 处理外部客户询价或不受信任的输入时，系统会自动将其放入临时的、受限的容器中执行，防止恶意脚本通过 Agent 渗透进企业核心数据库或内部网络。

4\. 企业级“技能中心” (Enterprise Skill Hubs)  
企业将建立私有的技能注册表。员工可以为自己的 Agent 订购经 IT 部门认证的“技能包”（如 ERP 插件、专业财务计算器等）。Agent 会根据当前任务上下文，自动从企业私有云拉取并挂载这些技能，实现功能的动态扩展。

未来的企业级智能体将借鉴 OpenClaw 的 Local-first（数据私有）和 Gateway/Node（控制与执行分离）理念，在保证数据所有权（Own your data）的前提下，通过多智能体协作和深度的软硬件集成，成为企业数字化转型的核心基础设施。

## 附录

- https://derisk.alipay.com/
- https://notebooklm.google.com/
- https://openclaw.ai/
- https://github.com/openclaw/openclaw
- https://github.com/anomalyco/opencode
- https://github.com/derisk-ai/OpenDerisk

大模型 · 目录

继续滑动看下一个

阿里云开发者

向上滑动看下一个

![kimi](data:image/png;base64,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)
