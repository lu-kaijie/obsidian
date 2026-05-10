---
title: 深入理解OpenClaw技术架构与实现原理（上）
source_url: https://mp.weixin.qq.com/s/wVcItgqsCiwl9-PZ56z27w
saved: 2026-05-09
tags: [ai]
---
踏天 *2026年3月19日 08:32*

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naKrH5XVibdiasmwsEydzQa9smdjpr7LE2icZmGRKUkqrztI5Eep2jfibEuKwKuqFiac1Wo4gjdtrAopxyw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0)

一、背景

最近OpenClaw如日中天，俨然已经是当下最热门并实用的个人助理。OpenClaw已经是我每日深度使用的效率工具，作为技术人，忍不住想系统性扒一下其技术架构与实现细节。当然了，本文也是通过与一堆Agent协作完成，包括OpenClaw、OpenCode、ClaudeCode、NotebookLLM、 DeRisk等。

OpenClaw 在面向个人助手方向上，不仅仅体现在其灵活先进的智能体架构，还有其围绕个人助手方向的各种工具与生态的完整实现，是各类技术与工具的集大成者。 最让人惊讶的是，这些能力的基本全部通过AI-Coding实现，可以说彻底改变了软件开发的范式，而且清晰简洁的架构设计与表达，比传统人类编程的系统具有更高的标准，可以说是开启新的软件构建范式的开山之作，非常值得深入的研究。

由于OpenClaw涉及的技术点非常多，所以文章篇幅会显得很长。 这里建议大家按照感兴趣的模块，分模块阅读了解：

1.统一控制平面Gateway网关

2.Agentic Loop/Pi Loop

3.定时任务系统

4.工具系统

5.Channels

6.上下文管理

7.SubAgent子智能体

8.SandBox沙箱系统

9.记忆管理

10.Skills模块

11.Session管理

12.自进化机制

13.工作区与Agent路由

14.Nodes

15.安全策略

16.配置管理

二、OpenClaw总体架构

如下图所示为OpenClaw的技术架构图，其架构设计上是以本地优先(Local-First)多端联动为核心，建立一个高度灵活且可拓展的个人AI助手系统。其架构可以概括为一个以Gateway(网关)为核心的控制平面的分布式系统。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

以下是对OpenClaw技术架构设计的详细解读:

1.核心控制平面: Gateway(网关)

Gateway是OpenClaw的心脏，充当系统的单一控制平面

- 功能职责: 负责管理会话(Sessions)、状态感知(Presence)、配置、定时任务(Cron)、网络钩子(Webhooks)以及控制界面(Control UI)和Canvas宿主
- 通信协议: 基于WebSocket(WS) 网络构建，为所有客户端、工具和事件提供统一的连接通道。
- 运行环境: 推荐在 Node ≥22 环境下运行，通常作为守护进程（Daemon）常驻后台。

2.智能体运行时: Pi Agent

Pi Agent是处理逻辑和生成回复的核心引擎:

- RPC模型: Pi Agent以RPC(远程过程调用)模式运行，支持工具流(Tool Streaming)和块流(Block Streaming)，确保响应的高效与实时性。
- 多智能体路由: 系统能够将来自不同频道、账户或同伴的输入路由到相互隔离的智能体(拥有独立的Workspace和会话)
- 会话模型: 提供 `main` 模式用户直接对话，并支持群组隔离、激活模式切换和队列管理。

3.连接生态: Channels(频道)

OpenClaw的一大特色是其极强的连接性，它将AI能力注入到用户已有的社交生态中。

- 多频道集成：原生支持包括 WhatsApp、Telegram、Slack、Discord、Google Chat、Signal、iMessage、Microsoft Teams、Matrix 以及 Zalo 等多种通讯平台
- 路由规则: 具备复杂的群组路由逻辑，包括提及门控（Mention gating）、回复标签处理以及针对不同频道的自动消息分块。

4.设备节点与伴侣应用: Nodes & Apps

通过将不同设备定义为“节点”，OpenClaw 实现了跨设备的硬件控制：

- 跨平台支持：包括 macOS 菜单栏应用、iOS 节点和 Android 节点。
- 硬件能力调用：通过 `node.invoke` 协议，智能体可以远程调用各节点上的硬件功能，如摄像头拍照/录码、屏幕录制、地理位置获取以及 macOS 特有的系统命令执行（ `system.run` ）。
- Voice Wake & Talk Mode：利用 ElevenLabs 等技术，在 macOS/iOS/Android 上提供始终在线的语音唤醒和连续对话能力。

5.工具与自动化：Tools & Skills

架构中集成了丰富的生产力工具：

- 浏览器控制：内置托管的 Chrome/Chromium 实例，支持快照、动作执行和文件上传。
- Live Canvas：基于 A2UI 构建的实时交互画布，允许智能体驱动视觉化的工作空间。
- 技能平台 (ClawHub)：提供技能注册表，支持捆绑技能、托管技能和工作区技能的自动搜索与安装。

6.安全与沙箱机制 (Security & Sandboxing)

由于 OpenClaw 会连接到真实的社交媒体和本地文件系统，安全性被置于重要位置：

- DM 配对策略：默认情况下，未知发送者必须通过配对码验证，bot 才会处理其消息，以防止不受信任的输入。
- Docker 沙箱：支持将 非主会话（如群组或外部频道）放入独立的 Docker 容器中运行，限制其对主机的访问权限，并对敏感工具（如浏览器、系统命令）进行黑白名单管理。

7.部署与远程访问

- 本地/远程灵活部署：Gateway 可以运行在本地或小型 Linux 实例上。
- 内网穿透：集成 Tailscale Serve/Funnel 或 SSH 隧道，使用户能够安全地从远程访问 Gateway 面板和 WebSocket 服务。

三、各系统模块详解

**3.1 统一控制平面Gateway网关**

### 3.1.1、核心定位

Gateway 是 OpenClaw 的统一控制平面，是一个 WebSocket 服务器，负责：

1.消息路由 - 所有频道（Telegram、Discord、Slack 等）的消息路由

2.会话管理 - Agent 会话的生命周期管理

3.工具调用 - Agent 工具的执行协调

4.节点通信 - iOS/Android 等移动节点的通信桥接

5.HTTP API - 提供 OpenAI 兼容的 REST API

### 3.1.2、架构模型

```js
┌─────────────────────────────────────────────────────┐│                   Gateway 进程                       ││  ┌──────────┐  ┌──────────┐  ┌──────────┐          ││  │WebSocket │  │ HTTP API │  │ Control  │          ││  │  Server  │  │ (OpenAI) │  │   UI     │          ││  └────┬─────┘  └────┬─────┘  └────┬─────┘          ││       │             │             │                 ││       └─────────────┴─────────────┘                 ││                     │                               ││              ┌──────┴──────┐                        ││              │  RPC Router │                        ││              └──────┬──────┘                        ││       ┌─────────────┼─────────────┐                 ││  ┌────┴────┐  ┌─────┴─────┐  ┌────┴────┐          ││  │Channels │  │  Agents   │  │  Nodes  │          ││  │(消息路由)│  │ (会话管理) │  │(设备节点)│          ││  └─────────┘  └───────────┘  └─────────┘          │└─────────────────────────────────────────────────────┘
```

### 3.1.3、关键特性

| 特性 | 说明 |
| --- | --- |
| 单端口复用 | WebSocket RPC + HTTP API + Control UI 共用一个端口（默认 18789） |
| 协议版本化 | 客户端声明 `minProtocol/maxProtocol` ，服务端拒绝不匹配的连接 |
| 角色分离 | `operator` （控制面）和 `node` （能力节点）两种角色 |
| 作用域控制 | 细粒度的 scopes 控制（ `operator.read` 、 `operator.write` 、 `operator.admin` 等） |
| 设备认证 | 支持设备身份验证和配对机制 |
| 热重载 | 支持 `hot` / `restart` / `hybrid` 三种配置重载模式 |

### 3.1.4、协议机制

连接握手流程：

```nginx
Gateway                          Client  │                                │  │◄──── connect.challenge ────────│  (可选：带 nonce 的挑战)  │                                │  │─────── connect (req) ─────────►│  携带 auth + role + scopes  │                                │  │◄────── hello-ok (res) ─────────│  返回 policy + 设备令牌  │                                │  │◄─────── events ────────────────│  持续推送状态变更
```

帧类型：

- Request: `{type:"req", id, method, params}`
- Response: `{type:"res", id, ok, payload|error}`
- Event: `{type:"event", event, payload, seq?, stateVersion?}`

### 3.1.5、认证模式

| 模式 | 使用场景 |
| --- | --- |
| `token` | 共享令牌认证（默认） |
| `password` | 共享密码认证 |
| `trusted-proxy` | 反向代理认证（如 Pomerium） |
| `device-token` | 设备身份认证（配对后自动获取） |

安全强制：

- 非环回地址绑定必须启用认证
- 明文 `ws://` 禁止连接非本机地址（CWE-319）

### 3.1.6、绑定模式

| 模式 | 地址 | 用途 |
| --- | --- | --- |
| `loopback` | 127.0.0.1 | 默认，仅本机访问 |
| `lan` | 0.0.0.0 | 局域网访问 |
| `tailnet` | Tailscale IP | Tailscale 网络 |
| `auto` | 自动选择 | 根据环境自动判断 |
| `custom` | 自定义地址 | 特定绑定需求 |

### 3.1.7、服务生命周期

macOS (launchd)：

```nginx
openclaw gateway install   # 安装 LaunchAgentopenclaw gateway start     # 启动服务openclaw gateway stop      # 停止服务openclaw gateway restart   # 重启服务
```

Linux (systemd)：

```css
openclaw gateway installsystemctl --user enable --now openclaw-gateway.service
```

### 3.1.8、配置热重载

| 模式 | 行为 |
| --- | --- |
| `off` | 不重载 |
| `hot` | 仅应用安全热更新 |
| `restart` | 需要重启时自动重启 |
| `hybrid` | 安全时热更新，必要时重启（默认） |

### 3.1.9、关键配置项

```json
{  "gateway": {    "port": 18789,    "bind": "loopback",    "mode": "local",    "auth": {      "mode": "token",      "token": "your-token"    },    "tls": {      "enabled": true,      "certPath": "/path/to/cert.pem",      "keyPath": "/path/to/key.pem"    },    "reload": {      "mode": "hybrid",      "debounceMs": 300    }  }}
```

### 3.1.10、常用命令

```nginx
# 启动网关openclaw gateway --port 18789
# 查看状态openclaw gateway statusopenclaw gateway status --deep  # 深度检查
# 健康检查openclaw gateway healthopenclaw channels status --probe
# 发现局域网网关openclaw gateway discover
# 查看日志openclaw logs --follow
```

### 3.1.11、核心源码位置

| 模块 | 路径 |
| --- | --- |
| CLI 入口 | `src/cli/gateway-cli/` |
| 客户端 | `src/gateway/client.ts` |
| 协议定义 | `src/gateway/protocol/` |
| 服务端 HTTP | `src/gateway/server-http.ts` |
| 配置类型 | `src/config/types.gateway.ts` |

**3.2 Agentic Loop / Pi Loop**

如下图所示为OpenClaw的整个推理循环架构，也是构成整个系统执行的大脑思考核心。系统中所有的运行逻辑都由推理循环架构来控制，也就是AgenticLoop，OpenClaw的推理循环是一个事件驱动的架构：

1.主循环 (`run.ts`) 负责错误处理、重试、profile轮换

2.尝试层 (`attempt.ts`) 负责单次LLM调用的完整生命周期

3.事件订阅 (`subscribe.ts`) 处理流式响应和工具调用

4.工具循环 由底层SDK自动管理，当模型返回 `tool_use` 时自动执行工具并继续调用。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

### 3.2.1 核心推理循环

#### 主循环架构 (runEmbeddedPiAgent in run.ts:192)

```javascript
runEmbeddedPiAgent()  └── while (true) {  // 行538 - 主重试循环        ├── 检查重试次数限制 (MAX_RUN_LOOP_ITERATIONS)        ├── 调用 runEmbeddedAttempt()  // 单次推理尝试        ├── 处理 context overflow → 自动压缩        ├── 处理 auth failure → profile轮换        ├── 处理 timeout → 重试或报错        └── 成功则返回 payloads      }
```

#### 单次推理尝试 (runEmbeddedAttempt in run/attempt.ts:306)

```swift
runEmbeddedAttempt()  ├── 1. 准备阶段  │     ├── 创建 workspace 和 session  │     ├── 解析 tools (createOpenClawCodingTools)  │     ├── 构建 system prompt  │     └── 创建 session manager  │  ├── 2. 会话初始化  │     ├── createAgentSession()  // 行688  │     ├── 设置 streamFn (LLM调用函数)  │     └── 安装事件订阅器 subscribeEmbeddedPiSession()  // 行921  │  ├── 3. 执行推理  │     ├── await activeSession.prompt(effectivePrompt)  // 行1180-1182  │     │   └── 调用 LLM API(streamSimple/streamFn)  │     │  │     └── 事件流处理:  │           ├── message_start/message_update/message_end  → handleMessageStart/Update/End  │           ├── tool_execution_start/update/end          → handleToolExecutionStart/Update/End  │           └── agent_start/agent_end                   → handleAgentStart/End  │  └── 4. 返回结果        ├── assistantTexts(生成的文本)        ├── toolMetas(工具调用元数据)        └── usage(token使用统计)
```

#### 工具调用循环

工具调用由底层 SDK (`@mariozechner/pi-coding-agent`) 的 `createAgentSession` 自动管理。当模型返回 `tool_use` 时：

```php
LLM Response(tool_use)  └── SDK 自动执行:        ├── handleToolExecutionStart()   // 记录工具开始        │     └── emitAgentEvent({stream: "tool", data: {phase: "start", name, toolCallId, args}})        │        ├── 执行工具函数        │        ├── handleToolExecutionUpdate()  // 流式更新        │     └── emitAgentEvent({stream: "tool", data: {phase: "update", ...}})        │        └── handleToolExecutionEnd()      // 工具完成              ├── emitAgentEvent({stream: "tool", data: {phase: "result", ...}})              ├── 调用 after_tool_call hook              └── SDK 自动将 tool_result 添加到消息历史                    └── 继续调用 LLM(下一轮推理)
```

#### 消息处理流程 (subscribeEmbeddedPiSession in pi-embedded-subscribe.ts:34)

```css
事件分发 (createEmbeddedPiSessionEventHandler):  ├── message_start    → handleMessageStart()  │     └── 重置状态，准备新消息  │  ├── message_update   → handleMessageUpdate()  │     ├── 处理 text_delta  │     ├── 处理 thinking 块  │     └── 调用 onPartialReply / onBlockReply  │  ├── message_end      → handleMessageEnd()  │     ├── 提取最终文本  │     ├── 处理 reasoning  │     └── 推送最终回复  │  ├── tool_execution_* → handleToolExecution*()  │     └── 跟踪工具状态，发送工具事件  │  └── agent_start/end  → handleAgentStart/End()        └── 生命周期事件广播
```

---

### 3.2.2 关键调用链

```css
用户消息  ↓runAgentTurnWithFallback() (agent-runner-execution.ts:72)  ↓runEmbeddedPiAgent() (pi-embedded-runner/run.ts:192)  ↓ [while循环 - 重试]runEmbeddedAttempt() (pi-embedded-runner/run/attempt.ts:306)  ↓createAgentSession() + activeSession.prompt()  ↓ [LLM调用 + 工具循环]subscribeEmbeddedPiSession() → 事件处理器  ↓onPartialReply / onBlockReply / onToolResult  ↓回复消息发送
```

### 3.2.3 LLM调用函数

实际的LLM API调用通过 `streamFn` 完成：

- 默认: `streamSimple` (来自 `@mariozechner/pi-ai`)
- Ollama: `createOllamaStreamFn()`
- 可通过 `applyExtraParamsToAgent()` 包装添加额外参数

**3.3 定时任务系统**

在OpenClaw中，定时任务是非常重要的一个基础设施，OpenClaw非常多的工作都是长任务，定时任务可以很好的满足这些长任务在后台单次或者周期性运行的诉求，同时与Heatbeat交互使得整个系统的交互更加拟人化，有些定时回复与交流，往往给人意想不到的拟人化体验。

### 3.3.1 、核心架构

```bash
┌─────────────────────────────────────────────────────────────────────┐│                         CronService                                  ││  (src/cron/service.ts)                                              │├─────────────────────────────────────────────────────────────────────┤│  ┌───────────────┐  ┌──────────────┐  ┌────────────────────┐       ││  │   Timer       │  │    Store     │  │   State            │       ││  │  (timer.ts)   │  │  (store.ts)  │  │  (state.ts)        │       ││  └───────┬───────┘  └──────┬───────┘  └────────────────────┘       ││          │                 │                                        ││          ▼                 ▼                                        ││  ┌─────────────────────────────────────────────────────────┐       ││  │               Jobs Collection(jobs.json)               │       ││  └─────────────────────────────────────────────────────────┘       │└─────────────────────────────────────────────────────────────────────┘
```

---

### 3.3.2、调度类型

```typescript
type CronSchedule =  | { kind: "at"; at: string }           // 一次性任务，指定时间  | { kind: "every"; everyMs: number; anchorMs?: number }  // 周期性任务  | { kind: "cron"; expr: string; tz?: string; staggerMs?: number }  // Cron表达式
```

支持的调度模式：

1.at - 一次性任务，执行后自动禁用

2.every - 固定间隔执行

3.cron - 标准cron表达式，支持时区和stagger

---

### 3.3.3、定时器机制

核心实现在 `src/cron/service/timer.ts`:

```javascript
const MAX_TIMER_DELAY_MS = 60_000;  // 最大延迟60秒const MIN_REFIRE_GAP_MS = 2_000;    // 最小重触发间隔2秒
// 定时器armed函数export function armTimer(state: CronServiceState){  const nextAt = nextWakeAtMs(state);  // 计算下次唤醒时间  const delay = Math.max(nextAt - now, 0);  const clampedDelay = Math.min(delay, MAX_TIMER_DELAY_MS);    state.timer = setTimeout(() => {    void onTimer(state).catch(...);  }, clampedDelay);}
```

关键特性：

- 定时器最大延迟60秒，防止时钟漂移
- 支持并发运行控制 (`maxConcurrentRuns`)
- 错误指数退避 (30s → 1min → 5min → 15min → 60min)
- 自动清理卡住的任务 (2小时超时)

---

### 3.3.4、任务持久化机制

存储位置：

- 默认路径: `~/.openclaw/cron/jobs.json`
- 可通过配置 `cron.store` 自定义

存储格式：

```typescript
type CronStoreFile = {  version: 1;  jobs: CronJob[];};
type CronJob = {  id: string;  name: string;  enabled: boolean;  schedule: CronSchedule;  sessionTarget: "main" | "isolated";  payload: CronPayload;  state: CronJobState;  // 运行时状态  // ...};
```

持久化流程：

1.原子写入（临时文件 + rename）

2.自动备份

3.支持热重载（文件修改时间检测）

运行日志：

- 路径: `~/.openclaw/cron/runs/<jobId>.jsonl`
- 自动裁剪（默认2MB，保留2000行）

---

### 3.3.5、任务恢复机制

启动恢复流程 (src/cron/service/ops.ts):

```javascript
export async function start(state: CronServiceState){// 1. 加载存储await ensureLoaded(state, { skipRecompute: true });
// 2. 清理卡住的任务for (const job of jobs) {if (job.state.runningAtMs) {      job.state.runningAtMs = undefined;  // 清除过期标记    }  }
// 3. 运行错过的任务await runMissedJobs(state);
// 4. 重新计算下次运行时间  recomputeNextRuns(state);
// 5. 启动定时器  armTimer(state);}
```

3.3.6、任务执行类型

两种执行模式：

1.Main Session (sessionTarget: "main")

- 注入系统事件到主会话
- payload必须为 `{ kind: "systemEvent", text: string }`

2.Isolated Agent (sessionTarget: "isolated")

- 独立agent会话执行
- payload必须为 `{ kind: "agentTurn", message: string, ... }`
- 支持模型覆盖、thinking模式、超时设置

超时控制：

```javascript
export async function executeJobCoreWithTimeout(state, job){  const jobTimeoutMs = resolveCronJobTimeoutMs(job);  return await Promise.race([    executeJobCore(state, job, abortSignal),    new Promise((_, reject) => {      timeoutId = setTimeout(() => {        abortController.abort();        reject(new Error("cron: job execution timed out"));      }, jobTimeoutMs);    }),  ]);}
```

### 3.3.7、与Heartbeat的集成

定时任务通过Heartbeat机制唤醒agent：

```javascript
// src/gateway/server-cron.tsconst cron = new CronService({  enqueueSystemEvent: (text, opts) => {    enqueueSystemEvent(text, { sessionKey, contextKey });  },  requestHeartbeatNow: (opts) => {    requestHeartbeatNow({ reason, agentId, sessionKey });  },  runHeartbeatOnce: async (opts) => {    return await runHeartbeatOnce({ cfg, reason, agentId, sessionKey });  },  // ...});
```

Wake模式：

- next-heartbeat - 等待下次心跳执行
- now - 立即触发心跳

### 3.3.8、Webhook通知

支持任务完成后的Webhook回调：

```perl
if (webhookTarget && evt.summary) {  await fetch(webhookTarget.url, {    method: "POST",    headers: {      "Content-Type": "application/json",      "Authorization": \`Bearer ${webhookToken}\`,    },    body: JSON.stringify(evt),  });}
```

### 3.3.9、CLI命令

```bash
openclaw cron status           # 查看调度器状态openclaw cron list             # 列出任务openclaw cron add              # 添加任务openclaw cron edit             # 编辑任务openclaw cron remove <id>      # 删除任务openclaw cron run <id>         # 手动触发任务
```

### 3.3.10、关键设计特点

1.单一定时器设计 - 只维护一个定时器，基于最近任务的nextRunAtMs

2.文件持久化 - JSON存储，支持跨进程共享

3.错误隔离 - 单个任务失败不影响其他任务

4.自动恢复 - 启动时检测并运行错过的任务

5.并发控制 - 可配置最大并发任务数

6.进度追踪 - 完整的运行日志和状态跟踪

7.Agent集成 - 可通过cron工具在agent中管理任务

**3.4 工具系统**

### 3.4.1 总体架构图

```cs
┌─────────────────────────────────────────────────────────────────────────────────┐│                              OpenClaw Tool System                               │├─────────────────────────────────────────────────────────────────────────────────┤│                                                                                 ││  ┌─────────────────────────────────────────────────────────────────────────┐   ││  │                        Tool Creation Layer                               │   ││  │  ┌──────────────────────┐  ┌──────────────────────┐                    │   ││  │  │createOpenClawCoding- │  │ createOpenClawTools  │                    │   ││  │  │Tools() (主入口)       │  │ (OpenClaw 特定工具)  │                    │   ││  │  │ pi-tools.ts:182      │  │ openclaw-tools.ts    │                    │   ││  │  └──────────┬───────────┘  └──────────┬───────────┘                    │   ││  │             │                         │                                 │   ││  │             ▼                         ▼                                 │   ││  │  ┌──────────────────────────────────────────────────────────────────┐  │   ││  │  │                    Coding Tools(pi-coding-agent)                │  │   ││  │  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌───────────┐  │  │   ││  │  │  │  read   │ │  write  │ │  edit   │ │  bash   │ │ (其他...)  │  │  │   ││  │  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘ └───────────┘  │  │   ││  │  └──────────────────────────────────────────────────────────────────┘  │   ││  └─────────────────────────────────────────────────────────────────────────┘   ││                                      │                                          ││                                      ▼                                          ││  ┌─────────────────────────────────────────────────────────────────────────┐   ││  │                       Tool Definition Layer                              │   ││  │                                                                          │   ││  │  ┌────────────────────────────────────────────────────────────────────┐ │   ││  │  │                    AnyAgentTool(核心类型)                          │ │   ││  │  │  tools/common.ts:8                                                  │ │   ││  │  │  ┌─────────────────────────────────────────────────────────────┐   │ │   ││  │  │  │ {                                                           │   │ │   ││  │  │  │   name: string;         // 工具名称 (小写唯一)               │   │ │   ││  │  │  │   label?: string;       // 显示标签                         │   │ │   ││  │  │  │   description: string;  // 工具描述 (给 AI 看)              │   │ │   ││  │  │  │   parameters?: TSchema; // JSON Schema / TypeBox schema     │   │ │   ││  │  │  │   execute?: (id, args, signal) => Promise<TResult>;         │   │ │   ││  │  │  │   ownerOnly?: boolean;  // 仅所有者可用                     │   │ │   ││  │  │  │ }                                                           │   │ │   ││  │  │  └─────────────────────────────────────────────────────────────┘   │ │   ││  │  └────────────────────────────────────────────────────────────────────┘ │   ││  │                                                                          │   ││  │  ┌────────────────────────────────────────────────────────────────────┐ │   ││  │  │              内置工具实现 (src/agents/tools/)                       │ │   ││  │  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐  │ │   ││  │  │  │browser-tool │ │memory-tool  │ │message-tool │ │exec-tool    │  │ │   ││  │  │  │浏览器控制    │ │记忆搜索     │ │消息发送     │ │命令执行     │  │ │   ││  │  │  └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘  │ │   ││  │  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐  │ │   ││  │  │  │canvas-tool  │ │gateway-tool │ │tts-tool     │ │process-tool │  │ │   ││  │  │  │画布操作      │ │网关管理     │ │语音合成     │ │进程管理     │  │ │   ││  │  │  └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘  │ │   ││  │  └────────────────────────────────────────────────────────────────────┘ │   ││  └─────────────────────────────────────────────────────────────────────────┘   ││                                      │                                          ││                                      ▼                                          ││  ┌─────────────────────────────────────────────────────────────────────────┐   ││  │                       Schema Normalization Layer                         │   ││  │                                                                          │   ││  │  ┌───────────────────────────────────────────────────────────────────┐  │   ││  │  │              normalizeToolParameters()                             │  │   ││  │  │              pi-tools.schema.ts                                    │  │   ││  │  │                                                                    │  │   ││  │  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐   │  │   ││  │  │  │ Anthropic       │  │ OpenAI          │  │ Google/Gemini   │   │  │   ││  │  │  │ 保持完整 JSON   │  │ 确保顶层有      │  │ 清理不支持的    │   │  │   ││  │  │  │ Schema draft    │  │ type:"object"   │  │ constraint 关键字│   │  │   ││  │  │  │ 2020-12 兼容   │  │                 │  │                 │   │  │   ││  │  │  └─────────────────┘  └─────────────────┘  └─────────────────┘   │  │   ││  │  └───────────────────────────────────────────────────────────────────┘  │   ││  └─────────────────────────────────────────────────────────────────────────┘   ││                                      │                                          ││                                      ▼                                          ││  ┌─────────────────────────────────────────────────────────────────────────┐   ││  │                       Policy Pipeline Layer                              │   ││  │                                                                          │   ││  │  ┌───────────────────────────────────────────────────────────────────┐  │   ││  │  │              applyToolPolicyPipeline()                             │  │   ││  │  │              tool-policy-pipeline.ts:65                            │  │   ││  │  │                                                                    │  │   ││  │  │   工具列表 ──▶ [Step 1: Profile Policy] ──▶ 过滤                  │  │   ││  │  │                     │                                              │  │   ││  │  │                     ▼                                              │  │   ││  │  │            [Step 2: Provider Profile Policy] ──▶ 过滤              │  │   ││  │  │                     │                                              │  │   ││  │  │                     ▼                                              │  │   ││  │  │            [Step 3: Global Policy] ──▶ 过滤                       │  │   ││  │  │                     │                                              │  │   ││  │  │                     ▼                                              │  │   ││  │  │            [Step 4: Agent Policy] ──▶ 过滤                        │  │   ││  │  │                     │                                              │  │   ││  │  │                     ▼                                              │  │   ││  │  │            [Step 5: Group Policy] ──▶ 过滤                        │  │   ││  │  │                     │                                              │  │   ││  │  │                     ▼                                              │  │   ││  │  │            [Step 6: Sandbox Policy] ──▶ 过滤                      │  │   ││  │  │                     │                                              │  │   ││  │  │                     ▼                                              │  │   ││  │  │            [Step 7: Subagent Policy] ──▶ 最终工具列表             │  │   ││  │  └───────────────────────────────────────────────────────────────────┘  │   ││  │                                                                          │   ││  │  ┌───────────────────────────────────────────────────────────────────┐  │   ││  │  │  ToolPolicyConfig (配置结构)                                       │  │   ││  │  │  ┌─────────────────────────────────────────────────────────────┐  │  │   ││  │  │  │ {                                                           │  │  │   ││  │  │  │   allow?: string[];      // 允许的工具列表                  │  │  │   ││  │  │  │   alsoAllow?: string[];  // 额外允许 (合并到 allow)         │  │  │   ││  │  │  │   deny?: string[];       // 拒绝的工具列表                  │  │  │   ││  │  │  │   profile?: "minimal" | "coding" | "messaging" | "full";    │  │  │   ││  │  │  │ }                                                           │  │  │   ││  │  │  └─────────────────────────────────────────────────────────────┘  │  │   ││  │  └───────────────────────────────────────────────────────────────────┘  │   ││  └─────────────────────────────────────────────────────────────────────────┘   ││                                      │                                          ││                                      ▼                                          ││  ┌─────────────────────────────────────────────────────────────────────────┐   ││  │                       Execution Layer                                    │   ││  │                                                                          │   ││  │  ┌───────────────────────────────────────────────────────────────────┐  │   ││  │  │                  AI Model Interaction                              │  │   ││  │  │                                                                    │  │   ││  │  │  ┌─────────────┐     tool_use      ┌─────────────────────────┐   │  │   ││  │  │  │   AI Model  │ ───────────────▶  │ Tool Execution Handler  │   │  │   ││  │  │  │ (Claude等)  │                   │ handleToolExecutionStart│   │  │   ││  │  │  └─────────────┘                   └───────────┬─────────────┘   │  │   ││  │  │         ▲                                       │                  │  │   ││  │  │         │                                       ▼                  │  │   ││  │  │         │                          ┌─────────────────────────┐   │  │   ││  │  │         │      tool_result         │  tool.execute()         │   │  │   ││  │  │         └──────────────────────────│  (实际执行)             │   │  │   ││  │  │                                    └───────────┬─────────────┘   │  │   ││  │  │                                                │                  │  │   ││  │  │                                                ▼                  │  │   ││  │  │                                    ┌─────────────────────────┐   │  │   ││  │  │                                    │handleToolExecutionEnd   │   │  │   ││  │  │                                    │(处理结果 + after hook)  │   │  │   ││  │  │                                    └─────────────────────────┘   │  │   ││  │  └───────────────────────────────────────────────────────────────────┘  │   ││  │                                                                          │   ││  │  ┌───────────────────────────────────────────────────────────────────┐  │   ││  │  │  Hook System(pi-tools.before-tool-call.ts)                       │  │   ││  │  │                                                                    │  │   ││  │  │  ┌─────────────────────┐  ┌─────────────────────┐                │  │   ││  │  │  │ before_tool_call    │  │ after_tool_call     │                │  │   ││  │  │  │ - 修改参数          │  │ - 记录结果          │                │  │   ││  │  │  │ - 阻止调用          │  │ - 循环检测          │                │  │   ││  │  │  │ - 记录日志          │  │ - 统计耗时          │                │  │   ││  │  │  └─────────────────────┘  └─────────────────────┘                │  │   ││  │  └───────────────────────────────────────────────────────────────────┘  │   ││  └─────────────────────────────────────────────────────────────────────────┘   ││                                                                                 ││  ┌─────────────────────────────────────────────────────────────────────────┐   ││  │                       Plugin System                                      │   ││  │                                                                          │   ││  │  ┌───────────────────────────────────────────────────────────────────┐  │   ││  │  │  resolvePluginTools()                                              │  │   ││  │  │  plugins/tools.ts                                                  │  │   ││  │  │                                                                    │  │   ││  │  │  ┌─────────────────────────────────────────────────────────────┐  │  │   ││  │  │  │  Plugin Registry(plugins/registry.ts)                       │  │  │   ││  │  │  │                                                              │  │  │   ││  │  │  │  extensions/                                                 │  │  │   ││  │  │  │  ├── msteams/     → msteams-send, msteams-react tools      │  │  │   ││  │  │  │  ├── matrix/      → matrix-send, matrix-react tools        │  │  │   ││  │  │  │  ├── zalo/        → zalo-send tool                         │  │  │   ││  │  │  │  └── voice-call/  → voice-call tool                        │  │  │   ││  │  │  └─────────────────────────────────────────────────────────────┘  │  │   ││  │  └───────────────────────────────────────────────────────────────────┘  │   ││  └─────────────────────────────────────────────────────────────────────────┘   ││                                                                                 ││  ┌─────────────────────────────────────────────────────────────────────────┐   ││  │                       HTTP Invocation API                                │   ││  │                                                                          │   ││  │  ┌───────────────────────────────────────────────────────────────────┐  │   ││  │  │  handleToolsInvokeHttpRequest()                                    │  │   ││  │  │  gateway/tools-invoke-http.ts                                      │  │   ││  │  │                                                                    │  │   ││  │  │  POST /tools/invoke                                               │  │   ││  │  │  {                                                                 │  │   ││  │  │    "tool": "browser",                                              │  │   ││  │  │    "action": "screenshot",                                         │  │   ││  │  │    "args": { ... },                                                │  │   ││  │  │    "sessionKey": "..."                                             │  │   ││  │  │  }                                                                 │  │   ││  │  └───────────────────────────────────────────────────────────────────┘  │   ││  └─────────────────────────────────────────────────────────────────────────┘   │└─────────────────────────────────────────────────────────────────────────────────┘
```

---

### 3.4.2 核心组件详解

#### 工具创建层

入口函数: `createOpenClawCodingTools()` (`pi-tools.ts:182`)

这是工具系统的主入口，负责：

- 解析工具策略配置
- 创建编码工具
- 创建 OpenClaw 特定工具
- 加载插件工具
- 应用策略过滤
- 规范化 Schema

```javascript
// 工具创建流程createOpenClawCodingTools() {  1. resolveEffectiveToolPolicy() // 解析策略  2. codingTools.flatMap()        // 处理基础编码工具  3. createExecTool()             // 创建执行工具  4. createOpenClawTools()        // 创建 OpenClaw 工具  5. applyToolPolicyPipeline()    // 应用策略管道  6. normalizeToolParameters()    // 规范化 Schema  7. wrapToolWithBeforeToolCallHook() // 添加钩子}
```

#### 工具定义层

核心类型: `AnyAgentTool` (`tools/common.ts:8`)

```typescript
type AnyAgentTool = AgentTool<any, unknown> & {  ownerOnly?: boolean;  // 仅所有者可用标志};
```

参数读取工具 (`tools/common.ts`):

- readStringParam() - 字符串参数
- readNumberParam() - 数字参数
- readStringArrayParam() - 字符串数组
- readReactionParams() - 表情反应参数

结果构造工具:

- jsonResult() - JSON 结果
- imageResult() - 图像结果

#### Schema 规范化层

函数: `normalizeToolParameters()` (`pi-tools.schema.ts`)

针对不同 AI 提供商的特殊处理：

| 提供商 | 处理方式 |
| --- | --- |
| Anthropic | 保持完整 JSON Schema draft 2020-12 兼容 |
| OpenAI | 确保顶层有 `type: "object"` |
| Google/Gemini | 清理不支持的 `format` /约束关键字 |
| 所有 | 合并 `anyOf` / `oneOf` union schemas |

#### 策略管道层

函数: `applyToolPolicyPipeline()` (`tool-policy-pipeline.ts:65`)

管道步骤顺序（优先级从低到高）：

```sql
Profile Policy → Provider Profile → Global Policy → Agent Policy     → Group Policy → Sandbox Policy → Subagent Policy
```

策略配置结构:

```typescript
type ToolPolicyConfig = {  allow?: string[];      // 白名单  alsoAllow?: string[];  // 追加白名单  deny?: string[];       // 黑名单  profile?: "minimal" | "coding" | "messaging" | "full";};
```

#### 执行层

事件处理 (`pi-embedded-subscribe.handlers.tools.ts`):

- handleToolExecutionStart() - 记录开始时间，发出事件
- handleToolExecutionEnd() - 处理结果，运行 after hook

Hook 系统:

- before\_tool\_call - 可修改参数/阻止调用
- after\_tool\_call - 记录结果/循环检测

#### 插件系统

工具解析: `resolvePluginTools()` (`plugins/tools.ts`)

插件工具注册：

```typescript
type PluginToolRegistration = {  pluginId: string;  factory: OpenClawPluginToolFactory;  names: string[];  optional: boolean;  source: string;};
```

#### HTTP 调用 API

端点: `POST /tools/invoke` (`gateway/tools-invoke-http.ts`)

允许外部系统直接调用工具，支持：

- 认证验证
- 策略应用
- 结果返回

---

### 3.4.3 工具调用完整流程

```js
用户消息    │    ▼┌─────────────────┐│ Gateway 接收    │└────────┬────────┘         │         ▼┌─────────────────┐     createOpenClawCodingTools()│ 构建工具列表    │ ◀──────────────────────────────└────────┬────────┘         │         ▼┌─────────────────┐     applyToolPolicyPipeline()│ 应用策略过滤    │ ◀──────────────────────────────└────────┬────────┘         │         ▼┌─────────────────┐     normalizeToolParameters()│ 规范化 Schema   │ ◀──────────────────────────────└────────┬────────┘         │         ▼┌─────────────────┐│ 发送给 AI 模型  │└────────┬────────┘         │         ▼┌─────────────────┐│ AI 生成 tool_use│└────────┬────────┘         │         ▼┌─────────────────┐     before_tool_call hook│ 工具执行前检查  │ ◀──────────────────────────────└────────┬────────┘         │         ▼┌─────────────────┐│ tool.execute()  │└────────┬────────┘         │         ▼┌─────────────────┐     after_tool_call hook│ 工具执行后处理  │ ◀──────────────────────────────└────────┬────────┘         │         ▼┌─────────────────┐│ 返回 tool_result│└────────┬────────┘         │         ▼┌─────────────────┐│ AI 继续推理     │└─────────────────┘
```

---

### 3.4.4 关键设计特点

1.分层架构: 创建 → 定义 → 规范化 → 策略 → 执行，职责清晰

2.策略管道: 多级策略叠加，支持精细控制

3.Provider 适配: 自动处理不同 AI 提供商的 Schema 差异

4.插件扩展: 插件工具与核心工具统一管理

5.Hook 机制: 支持工具调用前后拦截处理

6.沙箱支持: 隔离环境下的工具执行

**3.5 Channels**

Channels是OpenClaw进行社交生态连接最重要的设计，它将AI能力真正注入到了用户的社交与工作动线中。

### 3.5.1 核心架构

#### Channel抽象设计

```bash
┌─────────────────────────────────────────────────────────────────────┐│                        ChannelPlugin 接口                           │├─────────────────────────────────────────────────────────────────────┤│  id: ChannelId                                                      ││  meta: ChannelMeta (label, docsPath, aliases)                       ││  capabilities: ChannelCapabilities (chatTypes, polls, threads...)   │├─────────────────────────────────────────────────────────────────────┤│                     12个独立适配器 (职责分离)                        │├─────────────────────────────────────────────────────────────────────┤│ config       │ 账户配置管理        │ outbound    │ 消息发送        ││ setup        │ 账户设置流程        │ status      │ 状态探测        ││ gateway      │ Gateway生命周期     │ security    │ 安全策略        ││ pairing      │ 配对管理            │ groups      │ 群组管理        ││ threading    │ 线程处理            │ mentions    │ 提及解析        ││ messaging    │ 消息扩展            │ directory   │ 目录查询        ││ resolver     │ 路由解析            │ actions     │ 消息动作        │├─────────────────────────────────────────────────────────────────────┤│ 可选: onboarding, auth, heartbeat, agentTools                      │└─────────────────────────────────────────────────────────────────────┘
```

#### 消息流转完整架构

```markdown
消息流向图==============================================================================
INBOUND（入站）                                               OUTBOUND（出站）━━━━━━━━━━━━━━━━                                             ━━━━━━━━━━━━━━━━
┌──────────────┐│ 外部平台      ││ Telegram     ││ Discord      ││ Slack        ││ WhatsApp     ││ iMessage     ││ MSTeams...   │└──────┬───────┘       │       │ 1. Webhook/Gateway Event       ▼┌──────────────┐       ┌─────────────────────────────────────────────┐│ Channel      │       │              PLUGIN REGISTRY                 ││ Monitor      ├──────▶│  - channels: PluginChannelRegistration[]   ││ (接收层)     │       │  - plugins: PluginRecord[]                  │└──────┬───────┘       │  - Lazy Loading + Caching                  │       │               └─────────────────────────────────────────────┘       │ 2. 去重 & 预处理       ▼┌──────────────┐│ Allowlist    │       优先级（从高到低）：│ 验证         │       1. binding.peer (精确用户/群组)└──────┬───────┘       2. binding.peer.parent (线程继承)       │               3. binding.guild + roles       │ 3. 通过       4. binding.guild       ▼               5. binding.team┌──────────────┐       6. binding.account│ resolveAgent │       7. binding.channel│ Route        │       8.default agent│ (路由解析)   │└──────┬───────┘       │       │ 4. 返回 {agentId, sessionKey, matchedBy}       ▼┌──────────────┐│ Session      │       Session Key格式:│ 管理         │       {agentId}:{mainKey}:{channel}:{accountId}:{peerKind}:{peerId}│              │└──────┬───────┘       │       │ 5. 持久化会话元数据       ▼┌──────────────┐│   Agent      ││  AI Engine   ││  处理消息    │└──────┬───────┘       │       │ 6. 生成回复       ▼┌──────────────┐                    ┌──────────────────────┐│   Outbound   │                    │   ChannelDock        ││   Deliver    │                    │   (轻量级元数据)     ││              │                    │  - capabilities      │└──────┬───────┘                    │  - outbound config   │       │                            │  - threading rules   │       │ 7. 加载 OutboundAdapter    └──────────────────────┘       ▼┌──────────────┐│  消息分块    ││  (chunker)   │└──────┬───────┘       │       │ 8. 文本/媒体发送       ▼┌──────────────┐                    ┌──────────────────────┐│  Channel     │                    │   具体实现           ││  Outbound    ├───────────────────▶│  - Telegram: bot.ts ││  Adapter     │                    │  - Discord: send.ts │└──────────────┘                    │  - Slack: send.ts   │                                    │  - WhatsApp: web    │                                    └──────────────────────┘
```

#### 插件生命周期

```css
┌─────────────────────────────────────────────────────────────────────┐│                         Channel 生命周期                            │├─────────────────────────────────────────────────────────────────────┤│                                                                     ││  1. 注册阶段                                                        ││     └─ 插件通过 registerChannel() 注册到 PluginRegistry            ││                                                                     ││  2. 初始化阶段                                                       ││     ├─ 轻量加载: getChannelDock() → 仅元数据                       ││     └─ 完整加载: getChannelPlugin() → 完整插件                      ││                                                                     ││  3. 配置阶段                                                        ││     ├─ SetupAdapter: resolveAccountId(), applyAccountConfig()      ││     └─ ConfigAdapter: listAccountIds(), resolveAccount()            ││                                                                     ││  4. 运行阶段                                                        ││     ├─ Gateway启动: startAccount() [可选]                          ││     ├─ 消息接收: inbound handlers                                   ││     ├─ 路由解析: resolveAgentRoute()                                ││     └─ 消息发送: OutboundAdapter.sendText/sendMedia()               ││                                                                     ││  5. 监控阶段                                                        ││     ├─ StatusAdapter: probeAccount(), auditAccount()               ││     └─ HeartbeatAdapter: checkReady()                               ││                                                                     │└─────────────────────────────────────────────────────────────────────┘
```

#### 核心适配器架构

```bash
┌─────────────────────────────────────────────────────────────────────┐│                      ChannelPlugin 适配器矩阵                       │├──────────────────┬──────────────────────────────────────────────────┤│ 核心适配器        │ 职责                                            │├──────────────────┼──────────────────────────────────────────────────┤│ config           │ 账户配置: listAccountIds, resolveAccount,       ││                  │ isEnabled, isConfigured, setAccountEnabled       │├──────────────────┼──────────────────────────────────────────────────┤│ setup            │ 账户设置: resolveAccountId, applyAccountConfig, ││                  │ validateAccountConfig                            │├──────────────────┼──────────────────────────────────────────────────┤│ outbound         │ 消息发送: sendText, sendMedia, sendPoll,        ││                  │ editText, deleteMessage, chunker                 │├──────────────────┼──────────────────────────────────────────────────┤│ status           │ 状态探测: probeAccount, auditAccount,            ││                  │ formatStatusSnapshot                             │├──────────────────┼──────────────────────────────────────────────────┤│ gateway          │ 生命周期: startAccount, stopAccount,             ││                  │ getRunningAccountIds, createInboundHandler       │├──────────────────┼──────────────────────────────────────────────────┤│ security         │ 安全策略: resolveDmPolicy, collectWarnings       │├──────────────────┼──────────────────────────────────────────────────┤│ pairing          │ 配对管理: resolvePairing, validatePairing        │├──────────────────┼──────────────────────────────────────────────────┤│ 功能适配器        │ 职责                                            │├──────────────────┼──────────────────────────────────────────────────┤│ groups           │ 群组: resolveRequireMention, resolveToolPolicy    │├──────────────────┼──────────────────────────────────────────────────┤│ threading        │ 线程: resolveReplyToMode, buildToolContext       │├──────────────────┼──────────────────────────────────────────────────┤│ mentions         │ 提及: resolveMentions, formatMention            │├──────────────────┼──────────────────────────────────────────────────┤│ directory        │ 目录: resolveUserDirectory, resolveGroupDirectory│├──────────────────┼──────────────────────────────────────────────────┤│ resolver         │ 路由: resolveAgentRoute (自定义路由逻辑)         │├──────────────────┼──────────────────────────────────────────────────┤│ actions          │ 动作: resolveMessageActions, handleAction        │├──────────────────┼──────────────────────────────────────────────────┤│ messaging        │ 消息扩展: resolveMessageMeta, formatMessage      │└──────────────────┴──────────────────────────────────────────────────┘
```

#### 目录结构映射

```bash
src/├── channels/                    # Channel核心抽象│   ├── plugins/                 # 插件系统│   │   ├── types.plugin.ts      # ChannelPlugin接口定义│   │   ├── types.adapters.ts    # 12个适配器接口│   │   ├── types.core.ts        # Capabilities, Meta等│   │   └── registry-loader.ts   # 加载器工厂│   ├── dock.ts                  # 轻量级Dock (共享代码路径)│   ├── registry.ts              # Channel ID规范化│   ├── allow-from.ts            # Allowlist匹配│   ├── channel-config.ts        # 配置匹配│   └── session.ts               # 会话状态管理│├── routing/                     # 路由系统│   ├── resolve-route.ts         # 路由解析 (核心: 291-443行)│   ├── bindings.ts              # Agent绑定管理│   └── session-key.ts           # Session Key构建│├── telegram/                    # Telegram实现│   └── bot.ts                   # Bot创建, Update去重├── discord/                     # Discord实现│   ├── monitor.ts               # Gateway连接│   ├── send.ts                  # 消息发送 (Components V2)│   └── ui.ts                    # UI容器├── slack/                       # Slack实现├── signal/                      # Signal实现├── imessage/                    # iMessage实现├── web/                         # WhatsApp Web实现│   └── whatsapp-heartbeat.ts    # 心跳检测│├── infra/outbound/              # 出站消息基础设施│   └── deliver.ts               # 消息发送流程│└── plugins/                     # 插件注册系统    └── registry.ts              # PluginRegistry实现
extensions/                      # 扩展插件├── msteams/                     # Microsoft Teams│   └── src/channel.ts├── matrix/                      # Matrix协议│   └── src/channel.ts├── zalo/                        # Zalo└── voice-call/                  # 语音通话
```

### 3.5.2 关键设计要点

1.分层抽象: Application → Channel Abstraction → Implementation → Plugin Registry

2.适配器模式: 12个独立适配器，职责清晰分离

3.性能优化: Dock轻量加载、延迟加载、路由缓存、Update去重

4.扩展性: 新增Channel只需实现 `ChannelPlugin` 接口并注册

5.安全隔离: 每个channel独立的security、pairing、allowlist逻辑

**3.6 上下文管理**

### 3.6.1 核心概念

Context（上下文） = OpenClaw 在一次运行中发送给模型的所有内容，受模型的上下文窗口（token 限制）约束。如下为上下文构成要素，包括系统提示词汇、工具列表+描述、Skills列表(仅元数据)、工作区位置 + 时间 + 运行时数据。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

注意：Context ≠ Memory。记忆可持久化到磁盘，Context 是模型当前窗口内的内容。

### 3.6.2 上下文窗口管理

上下文解析优先级：

1.显式覆盖 `contextTokensOverride` → 直接使用

2.配置参数 `context1m: true` (Anthropic 1M 模型) → 1,048,576 tokens

3.模型注册表 → 从 `models.json` 或 provider catalog 发现

4.配置文件覆盖 → `models.providers.*.models[].contextWindow`

5.Fallback → 使用传入的默认值

上下文窗口守卫： `src/agents/context-window-guard.ts`

```ini
CONTEXT_WINDOW_HARD_MIN_TOKENS = 16_000   // 低于此值阻断运行CONTEXT_WINDOW_WARN_BELOW_TOKENS = 32_000 // 低于此值警告
```

三种检查结果：

- shouldWarn: 窗口 < 32K tokens（可能体验不佳）
- shouldBlock: 窗口 < 16K tokens（无法正常工作）
- 来源标记: `model` | `modelsConfig` | `agentContextTokens` | `default`

---

### 3.6.3 上下文压缩

#### 压缩机制

`src/agents/compaction.ts`

当会话接近或超过上下文窗口时，OpenClaw 自动触发压缩：

```javascript
旧消息 ──→ LLM 总结 ──→ 紧凑摘要条目 ──→ 持久化到 JSONL
```

核心流程：

1.Token 估算 (`estimateMessagesTokens`) - 计算当前消息总 token

2.分块 (`chunkMessagesByMaxTokens`) - 按 token 限制分块

3.摘要生成 (`summarizeWithFallback`) - 带重试的摘要生成

4.历史裁剪 (`pruneHistoryForContextShare`) - 裁剪旧消息保持预算

#### 自适应分块

```apache
BASE_CHUNK_RATIO = 0.4   // 基础分块比例MIN_CHUNK_RATIO = 0.15   // 最小分块比例SAFETY_MARGIN = 1.2      // 20% 缓冲补偿估算误差
```

当消息平均大小 > 上下文 10% 时，自动减小分块比例。

#### 过大消息处理

```javascript
isOversizedForSummary(msg, contextWindow)// 单条消息 > 上下文 50% → 无法安全压缩
```

处理策略：

1.尝试完整压缩

2.失败 → 只压缩小消息，记录过大消息

3.最终回退 → 返回消息计数说明

---

### 3.6.4 上下文剪枝

#### 与压缩的区别

| 特性 | Compaction | Pruning |
| --- | --- | --- |
| 作用范围 | 整个历史 | 仅 toolResult 消息 |
| 持久化 | ✓ 写入 JSONL | ✗ 仅内存 |
| 触发时机 | 接近窗口上限 | 每次请求前 (TTL 过期时) |
| 内容变更 | 生成摘要 | 软修剪/硬清除 |

#### 剪枝配置

`src/agents/pi-extensions/context-pruning/settings.ts`

```javascript
DEFAULT_CONTEXT_PRUNING_SETTINGS = {  mode: "cache-ttl",  ttlMs: 5 * 60 * 1000,        // 5 分钟 TTL  keepLastAssistants: 3,       // 保护最后 3 条助手消息  softTrimRatio: 0.3,          // 上下文占用 > 30% 触发软修剪  hardClearRatio: 0.5,         // 上下文占用 > 50% 触发硬清除  minPrunableToolChars: 50_000,  softTrim: {    maxChars: 4_000,           // > 4K 字符触发软修剪    headChars: 1_500,          // 保留头部 1500 字符    tailChars: 1_500,          // 保留尾部 1500 字符  },  hardClear: {    enabled: true,    placeholder: "[Old tool result content cleared]",  },}
```

#### 剪枝执行流程

`src/agents/pi-extensions/context-pruning/pruner.ts`

```markdown
1. 检查 TTL 是否过期   ↓ 过期2. 计算上下文占用比例   ↓ 超过 softTrimRatio3. 软修剪：对可修剪工具结果截取 head + tail   ↓ 仍超过 hardClearRatio4. 硬清除：替换为占位符
```

保护机制：

- 不修改用户/助手消息
- 跳过包含图片的 toolResult
- 保护 bootstrap 阶段消息（第一条用户消息之前）
- 保护最后 N 条助手消息之后的工具结果

---

### 3.6.5 工具结果上下文守卫

`src/agents/pi-embedded-runner/tool-result-context-guard.ts`

#### 单条工具结果限制

```apache
SINGLE_TOOL_RESULT_CONTEXT_SHARE = 0.5  // 单条最多占上下文 50%TOOL_RESULT_CHARS_PER_TOKEN_ESTIMATE = 2// 更保守的估算
```

#### 执行逻辑

```javascript
// 1. 每条工具结果限制0.5

// 2. 总上下文预算 (75% headroom)0.75

// 3. 超预算时压缩最旧的工具结果//    替换为 "[compacted: tool output removed to free context]"
```

---

### 3.6.6 运行时上下文注入

#### 工作区文件注入

默认注入文件（如果存在）：

- AGENTS.md - 项目规则
- SOUL.md - 角色定义
- TOOLS.md - 工具指南
- IDENTITY.md - 身份信息
- USER.md - 用户偏好
- HEARTBEAT.md - 心跳状态
- BOOTSTRAP.md - 首次运行引导

截断配置：

```json
{  "agents": {    "defaults": {      "bootstrapMaxChars": 20000,       // 单文件上限      "bootstrapTotalMaxChars": 150000  // 总上限    }  }}
```

#### 压缩后上下文刷新

`src/auto-reply/reply/post-compaction-context.ts`

压缩完成后，重新注入 AGENTS.md 中的关键章节：

- \## Session Startup
- \## Red Lines

目的：确保模型在压缩后仍遵循关键规则。

#### Sandbox 上下文

`src/agents/sandbox/context.ts`

为沙箱会话提供：

- 容器信息 (`containerName`, `containerWorkdir`)
- 工作区映射 (`workspaceDir`, `agentWorkspaceDir`)
- Docker 配置 (`docker`)
- 工具权限 (`tools`)
- 浏览器桥接 (`browser`, `fsBridge`)

---

### 3.6.7 检查与调试命令

| 命令 | 作用 |
| --- | --- |
| `/status` | 快速查看窗口占用率 + 会话设置 |
| `/context list` | 查看注入文件大小、工具 schema 大小 |
| `/context detail` | 详细分解各组件大小 |
| `/usage tokens` | 每次回复显示 token 使用量 |
| `/compact` | 手动触发压缩 |

### 3.6.8 关键配置汇总

```json
{  "agents": {    "defaults": {      // 上下文窗口      "contextTokens": 200000,       // 硬性上限            // Bootstrap 注入      "bootstrapMaxChars": 20000,      "bootstrapTotalMaxChars": 150000,            // 压缩配置      "compaction": {        "mode": "auto",        "targetTokens": 0.7          // 目标占用率      },            // 剪枝配置      "contextPruning": {        "mode": "cache-ttl",        "ttl": "5m",        "keepLastAssistants": 3,        "softTrimRatio": 0.3,        "hardClearRatio": 0.5      }    }  },    // 模型上下文窗口覆盖  "models": {    "providers": {      "anthropic": {        "models": [          { "id": "claude-sonnet-4", "contextWindow": 200000 }        ]      }    }  }}
```

**3.7 SubAgent 架构详解**

### 3.7.1 核心概念

SubAgent（子智能体）是从现有 Agent 运行中生成的后台独立运行实例。它们在独立的会话中执行任务，完成后将结果自动通告回请求者的聊天渠道。

#### 关键特征

- 会话隔离：每个 SubAgent 拥有独立的会话键 `agent:<agentId>:subagent:<uuid>`
- 后台执行：非阻塞式运行，支持并行处理
- 结果通告：完成时自动向父会话推送结果摘要
- 嵌套支持：支持多层嵌套（最大5层深度，推荐2层）

---

### 3.7.2 架构组件

#### 会话键系统

会话键格式与深度：

| 深度 | 会话键格式 | 角色 | 能否派生子智能体 |
| --- | --- | --- | --- |
| 0 | `agent:<id>:main` | 主智能体 | 总是可以 |
| 1 | `agent:<id>:subagent:<uuid>` | 子智能体（编排者） | 仅当 `maxSpawnDepth >= 2` |
| 2 | `agent:<id>:subagent:<uuid>:subagent:<uuid>` | 子子智能体（叶子工作者） | 永远不能 |

深度计算逻辑（ `subagent-depth.ts:124-176` ）：

```typescript
export function getSubagentDepthFromSessionStore(  sessionKey: string | undefined | null,  opts?: { cfg?: OpenClawConfig; store?: Record<string, SessionDepthEntry> }): number {  // 从会话键解析基础深度  const fallbackDepth = getSubagentDepth(raw);    // 读取会话存储中的 spawnDepth 字段  const storedDepth = normalizeSpawnDepth(entry?.spawnDepth);    // 或通过 spawnedBy 链递归计算父深度 + 1  const parentDepth = depthFromStore(spawnedBy);  return parentDepth + 1;}
```

#### 注册表

核心数据结构（ `subagent-registry.types.ts:6-35` ）：

```typescript
export type SubagentRunRecord = {  runId: string;                    // 运行标识符  childSessionKey: string;          // 子会话键  requesterSessionKey: string;      // 请求者会话键  requesterOrigin?: DeliveryContext; // 请求者来源（渠道、账号等）  task: string;                     // 任务描述  cleanup: "delete" | "keep";       // 清理策略  label?: string;                   // 显示标签  model?: string;                   // 使用的模型  runTimeoutSeconds?: number;       // 运行超时  spawnMode?: SpawnSubagentMode;    // 运行模式  createdAt: number;                // 创建时间  startedAt?: number;               // 开始时间  endedAt?: number;                 // 结束时间  outcome?: SubagentRunOutcome;     // 运行结果  suppressAnnounceReason?: "steer-restart" | "killed"; // 抑制通告原因  endedReason?: SubagentLifecycleEndedReason; // 结束原因};
```

核心职责：

1.运行跟踪：维护所有活跃和历史 SubAgent 运行记录

2.生命周期监听：通过 `onAgentEvent` 监听 `lifecycle` 事件（start/error/end）

3.持久化：运行记录持久化到磁盘，支持网关重启后恢复

4.级联停止：停止父运行时自动停止所有子运行

5.孤儿检测：恢复时检测并清理孤儿运行（缺失会话条目）

初始化流程（ `subagent-registry.ts:488-518` ）：

```php
function restoreSubagentRunsOnce(){  if (restoreAttempted) return;  restoreAttempted = true;    // 1. 从磁盘恢复运行记录  const restoredCount = restoreSubagentRunsFromDisk({    runs: subagentRuns,    mergeOnly: true,  });    // 2. 协调孤儿运行  if (reconcileOrphanedRestoredRuns()) {    persistSubagentRuns();  }    // 3. 恢复未完成的工作  for (const runId of subagentRuns.keys()) {    resumeSubagentRun(runId);  }}
```

#### 派生逻辑

核心流程（ `subagent-spawn.ts:166-550` ）：

```ruby
┌─────────────────────────────────────────────────────────────┐│ 1. 权限与深度检查                                              ││    - 检查调用者深度 < maxSpawnDepth                           ││    - 检查活跃子运行数 < maxChildrenPerAgent                   ││    - 检查 agentId 允许列表                                    │└─────────────────────────────────────────────────────────────┘                            ↓┌─────────────────────────────────────────────────────────────┐│ 2. 创建子会话                                                  ││    - 生成子会话键: agent:<id>:subagent:<uuid>                ││    - 通过 sessions.patch 设置 spawnDepth                      ││    - 设置模型和思考级别                                        │└─────────────────────────────────────────────────────────────┘                            ↓┌─────────────────────────────────────────────────────────────┐│ 3. 线程绑定（可选）                                            ││    - 调用 subagent_spawning 钩子准备线程绑定                  ││    - 失败时回滚删除会话                                        │└─────────────────────────────────────────────────────────────┘                            ↓┌─────────────────────────────────────────────────────────────┐│ 4. 启动子运行                                                  ││    - 构建子智能体系统提示                                      ││    - 调用 gateway.agent() 启动运行                            ││    - 使用专属 lane: AGENT_LANE_SUBAGENT                      │└─────────────────────────────────────────────────────────────┘                            ↓┌─────────────────────────────────────────────────────────────┐│ 5. 注册运行记录                                                ││    - 调用 registerSubagentRun() 注册到 registry              ││    - 开始等待完成                              ││    - 触发 subagent_spawned 钩子                               │└─────────────────────────────────────────────────────────────┘
```

系统提示构建（ `subagent-announce.ts:921-1025` ）：

```typescript
export function buildSubagentSystemPrompt(params: {  requesterSessionKey?: string;  childSessionKey: string;  label?: string;  task?: string;  childDepth?: number;  maxSpawnDepth?: number;}) {  const canSpawn = childDepth < maxSpawnDepth;    const lines = [    "# Subagent Context",    \`You are a **subagent** spawned by the ${parentLabel} for a specific task.\`,    "",    "## Your Role",    \`- You were created to handle: ${taskText}\`,    "- Complete this task. That's your entire purpose.",    "",    "## Rules",    "1. **Stay focused** - Do your assigned task, nothing else",    "2. **Complete the task** - Your final message will be automatically reported",    "3. **Don't initiate** - No heartbeats, no proactive actions, no side quests",    "4. **Be ephemeral** - You may be terminated after task completion",    "5. **Trust push-based completion** - Descendant results auto-announce",    "6. **Recover from compacted/truncated tool output** - Re-read with smaller chunks",    // ...  ];    if (canSpawn) {    lines.push(      "## Sub-Agent Spawning",      "You CAN spawn your own sub-agents using \`sessions_spawn\`.",      "Use the \`subagents\` tool to steer, kill, or check status.",      "Your sub-agents will announce their results back to you automatically.",      // ...    );  } elseif (childDepth >= 2) {    lines.push(      "## Sub-Agent Spawning",      "You are a leaf worker and CANNOT spawn further sub-agents.",      // ...    );  }}
```

#### 通告机制

核心流程（ `subagent-announce.ts:1053-1382` ）：

```swift
┌─────────────────────────────────────────────────────────────┐│ 1. 等待运行结束                                                ││    - 等待嵌入式运行完成（如果是嵌入式）                        ││    - 调用 agent.wait 等待运行完成                              ││    - 读取最新助手回复或工具结果                                │└─────────────────────────────────────────────────────────────┘                            ↓┌─────────────────────────────────────────────────────────────┐│ 2. 构建通告消息                                                ││    - 提取运行结果文本                                          ││    - 计算运行统计（运行时间、token使用量、成本）               ││    - 生成状态标签（成功/超时/失败）                            │└─────────────────────────────────────────────────────────────┘                            ↓┌─────────────────────────────────────────────────────────────┐│ 3. 确定投递目标                                                ││    - 检查线程绑定路由（bound 模式）                            ││    - 调用 subagent_delivery_target 钩子（hook 模式）          ││    - 回退到请求者来源（fallback 模式）                         │└─────────────────────────────────────────────────────────────┘                            ↓┌─────────────────────────────────────────────────────────────┐│ 4. 投递通告                                                    ││    - 直接投递：调用 gateway.agent() 或 gateway.send()         ││    - 队列投递：当请求者忙时入队等待                            ││    - 嵌套处理：如果请求者是子智能体，向上冒泡                  │└─────────────────────────────────────────────────────────────┘                            ↓┌─────────────────────────────────────────────────────────────┐│ 5. 清理                                                        ││    - 更新会话标签（如果有）                                    ││    - 删除子会话（如果 cleanup: "delete"）                      ││    - 触发 subagent_ended 钩子                                  │└─────────────────────────────────────────────────────────────┘
```

嵌套通告冒泡（ `subagent-announce.ts:1222-1253` ）：

```javascript
// 如果请求者子智能体已结束，向上冒泡到其父if (requesterIsSubagent && !isSubagentSessionRunActive(targetRequesterSessionKey)) {  // 检查父会话是否还存在  const parentSessionEntry = loadSessionEntryByKey(targetRequesterSessionKey);  const parentSessionAlive = parentSessionEntry?.sessionId?.trim();    if (!parentSessionAlive) {    // 父会话已删除，回退到祖父    const fallback = resolveRequesterForChildSession(targetRequesterSessionKey);    if (fallback?.requesterSessionKey) {      targetRequesterSessionKey = fallback.requesterSessionKey;      targetRequesterOrigin = fallback.requesterOrigin;      requesterDepth = getSubagentDepthFromSessionStore(targetRequesterSessionKey);      requesterIsSubagent = requesterDepth >= 1;    }  }  // 如果父会话存活（只是没有活跃运行），继续向父注入}
```

#### 工具系统

工具定义（ `subagents-tool.ts:342-680` ）：

动作：

- list：列出活跃和最近的子运行
- kill <target>：停止指定的子运行（支持级联）
- steer <target> <message>：向运行中的子智能体发送指导消息

权限模型：

```cs
function resolveRequesterKey(params: {  cfg: ReturnType<typeof loadConfig>;  agentSessionKey?: string;}): ResolvedRequesterKey {  const callerSessionKey = resolveInternalSessionKey({ key: callerRaw, alias, mainKey });    if (!isSubagentSessionKey(callerSessionKey)) {    // 主智能体：查看自己的子运行    return { requesterSessionKey: callerSessionKey, callerIsSubagent: false };  }    const callerDepth = getSubagentDepthFromSessionStore(callerSessionKey, { cfg });  const maxSpawnDepth = cfg.agents?.defaults?.subagents?.maxSpawnDepth ?? DEFAULT_SUBAGENT_MAX_SPAWN_DEPTH;    if (callerDepth < maxSpawnDepth) {    // 编排者子智能体：查看自己的子运行    return { requesterSessionKey: callerSessionKey, callerIsSubagent: true };  }    // 叶子子智能体：查看父的子运行（兄弟运行）  const spawnedBy = callerEntry?.spawnedBy?.trim();  return { requesterSessionKey: spawnedBy || callerSessionKey, callerIsSubagent: true };}
```

级联停止（ `subagents-tool.ts:276-318` ）：

```typescript
async function cascadeKillChildren(params: {  cfg: ReturnType<typeof loadConfig>;  parentChildSessionKey: string;  cache: Map<string, Record<string, SessionEntry>>;}): Promise<{ killed: number; labels: string[] }> {  const childRuns = listSubagentRunsForRequester(params.parentChildSessionKey);  let killed = 0;  const labels: string[] = [];    for (const run of childRuns) {    if (!run.endedAt) {      const stopResult = await killSubagentRun({ cfg, entry: run, cache });      if (stopResult.killed) {        killed += 1;        labels.push(resolveSubagentLabel(run));      }    }        // 递归停止孙运行    const cascade = await cascadeKillChildren({      cfg,      parentChildSessionKey: run.childSessionKey,      cache,    });    killed += cascade.killed;    labels.push(...cascade.labels);  }    return { killed, labels };}
```

---

### 3.7.3 配置与限制

#### 核心配置项

```css
{  agents: {    defaults: {      subagents: {        maxSpawnDepth: 2,           // 最大派生深度（1-5，默认1）        maxChildrenPerAgent: 5,     // 每个会话最大活跃子运行数（1-20）8
0
60
        model: "claude-3-haiku",    // 子智能体默认模型        thinking: "medium",         // 默认思考级别      },    },    list: [{      agentId: "orchestrator",      subagents: {        allowAgents: ["*"],         // 允许派生任意 agentId      },    }],  },  tools: {    subagents: {      tools: {        deny: ["gateway", "cron"],  // 工具黑名单        allow: ["read", "exec"],    // 工具白名单      },    },  },}
```

#### 工具策略

默认策略：

- 叶子子智能体：无会话工具（ `sessions_*` ）
- 编排者子智能体（深度1，当 maxSpawnDepth >= 2）：
- 获得 `sessions_spawn` 、 `subagents` 、 `sessions_list` 、 `sessions_history`
- 仍被拒绝： `sessions_send` 、 `sessions_delete` 等系统工具

工具过滤逻辑（docs:229-236）：

```cs
// 深度1编排者获得会话工具if (isSubagentSessionKey(sessionKey) && depth === 1 && maxSpawnDepth >= 2) {  allowedTools.push("sessions_spawn", "subagents", "sessions_list", "sessions_history");}
// 深度2叶子工作者无会话工具if (depth >= 2) {  denySet.add("sessions_spawn");}
```

---

### 3.7.4 插件钩子系统

#### 钩子类型

| 钩子名称 | 触发时机 | 用途 |
| --- | --- | --- |
| `subagent_spawning` | 派生前 | 准备线程绑定，验证权限 |
| `subagent_spawned` | 派生成功后 | 记录日志，更新UI状态 |
| `subagent_delivery_target` | 确定投递目标 | 自定义通告路由 |
| `subagent_ended` | 运行结束 | 清理资源，发送告别消息 |

#### Discord 扩展示例（extensions/discord/src/subagent-hooks.ts）：

```cs
// 注册钩子export function registerDiscordSubagentHooks(){  const hookRunner = getGlobalHookRunner();    hookRunner.register("subagent_spawning", async (event, ctx) => {    // 创建 Discord 线程并绑定到子智能体会话    const thread = await createDiscordThread({      channelId: event.requester.to,      name: \`Subagent: ${event.label || event.agentId}\`,    });        return {      status: "ok",      threadBindingReady: true,      threadId: thread.id,    };  });    hookRunner.register("subagent_delivery_target", async (event, ctx) => {    // 将通告路由到绑定的 Discord 线程    const binding = getThreadBinding(event.childSessionKey);    if (binding) {      return {        origin: {          channel: "discord",          to: binding.channelId,          threadId: binding.threadId,        },      };    }    return { origin: event.requesterOrigin };  });}
```

---

### 3.7.5 并发与队列

### Lane 系统（src/process/lanes.ts）

```javascript
exportconstenumCommandLane {  Main = "main",        // 主会话  Cron = "cron",        // 定时任务  Subagent = "subagent", // 子智能体  Nested = "nested",    // 嵌套调用}
```

并发控制：

- 子智能体使用专属 `Subagent` lane
- 全局并发上限由 `maxConcurrent` 控制（默认8）
- 每个会话的子运行数由 `maxChildrenPerAgent` 控制（默认5）

#### 队列集成（subagent-announce.ts:645-702）

当请求者会话忙时，通告会被入队：

```javascript
async function maybeQueueSubagentAnnounce(params): Promise<"steered" | "queued" | "none"> {  const queueSettings = resolveQueueSettings({ cfg, channel, sessionEntry });  const isActive = isEmbeddedPiRunActive(sessionId);    // 1. 尝试 steer 模式  const shouldSteer = queueSettings.mode === "steer" || queueSettings.mode === "steer-backlog";  if (shouldSteer) {    const steered = queueEmbeddedPiMessage(sessionId, params.triggerMessage);    if (steered) return"steered";  }    // 2. 尝试 followup/collect 模式  const shouldFollowup =     queueSettings.mode === "followup" ||    queueSettings.mode === "collect" ||    queueSettings.mode === "steer-backlog" ||    queueSettings.mode === "interrupt";      if (isActive && shouldFollowup) {    enqueueAnnounce({      key: buildAnnounceQueueKey(canonicalKey, origin),      item: { announceId, prompt, sessionKey, origin },      settings: queueSettings,      send: sendAnnounce,    });    return"queued";  }    return"none";}
```

---

### 3.7.6 认证与安全

#### 认证继承（docs:196-204）

```php
// 子智能体认证由 agentId 决定，而非会话类型const authStore = loadAuthStore({ agentId: targetAgentId });
// 主智能体认证作为回退合并const mainAuthStore = loadAuthStore({ agentId: requesterAgentId });const mergedAuth = mergeAuthStores(authStore, mainAuthStore, {  agentProfilesOverride: true, // 子智能体配置优先});
```

#### 允许列表（docs:122-128）

```php
// 子智能体认证由 agentId 决定，而非会话类型const authStore = loadAuthStore({ agentId: targetAgentId });// 主智能体认证作为回退合并const mainAuthStore = loadAuthStore({ agentId: requesterAgentId });const mergedAuth = mergeAuthStores(authStore, mainAuthStore, {  agentProfilesOverride: true, // 子智能体配置优先});
```

---

### 3.7.7 典型场景

#### 并行研究

```php
用户: "研究这三个主题并生成报告"主智能体:  - sessions_spawn(task: "研究主题A", label: "research-a")  - sessions_spawn(task: "研究主题B", label: "research-b")  - sessions_spawn(task: "研究主题C", label: "research-c")  [等待通告...]research-a: ✅ 完成 - [结果摘要]research-b: ✅ 完成 - [结果摘要]research-c: ✅ 完成 - [结果摘要]
主智能体: 综合结果生成最终报告
```

#### 编排者模式（maxSpawnDepth=2）

```php
用户: "重构这个大型项目"主智能体:  - sessions_spawn(      task: "协调重构工作",      agentId: "orchestrator",      label: "refactor-coordinator"    )
refactor-coordinator（深度1编排者）:  - sessions_spawn(task: "重构模块A", label: "worker-a")  - sessions_spawn(task: "重构模块B", label: "worker-b")  - sessions_spawn(task: "重构模块C", label: "worker-c")  [等待子运行通告...]worker-a: ✅ 完成worker-b: ✅ 完成worker-c: ✅ 完成
refactor-coordinator: 综合结果，通知主智能体主智能体: 向用户报告完成
```

#### 线程绑定会话

```php
用户（在 Discord 线程中）: "监控这个服务的性能"主智能体:  - sessions_spawn(      task: "启动性能监控",      thread: true,      mode: "session"    )  [Discord 扩展创建专用线程][子智能体在专用线程中运行]
用户（在同一 Discord 线程）: "当前状态如何？"[消息路由到绑定的子智能体会话]子智能体: "当前 CPU 45%, 内存 2.1GB..."
用户: "/unfocus"[解除线程绑定，后续消息路由回主智能体]
```

---

### 3.7.8 总结

SubAgent 架构的设计哲学：

1.隔离与独立：每个子智能体在独立会话中运行，拥有独立的上下文、token 配额和工具集

2.推式通知：结果自动通告，避免轮询开销和复杂性

3.嵌套编排：支持多层嵌套，实现复杂的编排模式

4.资源可控：通过深度限制、并发上限和工具策略控制资源消耗

5.可扩展性：通过插件钩子系统支持自定义行为（线程绑定、路由策略等）

关键设计决策：

- 使用会话键深度而非独立字段跟踪嵌套层级
- 通告机制而非返回值，支持异步和非阻塞语义
- 工具策略按深度区分，编排者获得管理工具而工作者专注任务
- 持久化注册表确保网关重启后不丢失运行状态

这套架构使 OpenClaw 能够高效处理并行任务、长时间运行作业和复杂的多智能体协作场景。

篇幅原因更多精彩内容在《深入理解OpenClaw技术架构与实现原理（下）》，请持续关注～

大模型 · 目录

继续滑动看下一个

阿里云开发者

向上滑动看下一个

![kimi](data:image/png;base64,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)
