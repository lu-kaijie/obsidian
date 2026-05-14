# CLAUDE.md — Wiki Schema

## 指令文件同步规则

- `AGENTS.md` 必须保持为 `CLAUDE.md` 的等价镜像，供不同 code agent 使用
- 修改任一文件时，必须同步修改另一文件，避免两份规范漂移
- 若两份文件内容不一致，以最新修改后的规范为准，并立即完成对另一份文件的同步
- 新对话开始时，若读取了其中一份指令文件，应默认另一份应表达相同约束
 

## 会话启动流程

每次新对话开始时：

1. `git pull -r` 拉取手机端可能的新推送（rebase 模式，保持线性历史）
2. 读取当前指令文件了解规范（Claude 使用 `CLAUDE.md`，其他 agent 使用 `AGENTS.md`）
3. 读取 `index.md` 了解当前 wiki 内容状态
4. 扫描 `Clippings/` 找出 Web Clipper 新同步的文件（无规范 frontmatter 或 frontmatter 不完整），判断主题后移动到 `raw/<主题>/`，补全 frontmatter（`ingested: false`、`source_url`、`saved` 等）。然后扫描 `raw/` 找出所有未处理文件：
   - 未出现在 `ingest-state.yaml` 中的文件
   - `ingest-state.yaml` 中标记为 `ingested: false` 的文件
   - 直接位于 `raw/` 根目录下、缺少规范 frontmatter 的文件

   统一提示用户："发现 N 个未处理资料，是否现在 ingest？"
5. 读取 `inbox.md`，若其中有内容，提示用户："inbox 中有待处理链接，是否现在处理？"处理完成后删除已处理的行。
6. 读取 `log.md`，统计上次 lint 之后的 ingest 次数，若达到 10 次则提醒用户："距上次健康检查已摄入 10 篇资料，建议执行 lint。"
7. 等待用户指令

## 目录结构

```
/Clippings/    ← Web Clipper 落地目录（暂存，LLM 移到 raw 后可清空）
/raw/          ← 原始资料（只读，LLM 不修改）
  /assets/     ← 图片等附件
/notes/        ← 用户与 LLM 共同维护的工作笔记层（可编辑）
/wiki/         ← LLM 维护的知识页面
  overview.md  ← 知识库整体概览（Lint 时更新）
index.md       ← Wiki 内容目录（每次 ingest 后更新）
log.md         ← 操作日志（仅追加）
ingest-state.yaml ← 原始资料摄入状态表（机器可读）
CLAUDE.md      ← 本文件，schema 配置
```

## 资料来源一致性

- **网页文章：** 优先通过浏览器插件保存到 `Clippings/`（全文），LLM 移到 `raw/` 并生成 wiki 摘要
- **用户发送 URL：** LLM 通过 WebFetch 抓取（可能非全文），保存到 `raw/`，生成 wiki 摘要
- **raw/ 下始终保存原始内容**（全文优先）
- **notes/ 下始终保存用户自己的思考、摘记、问题与中间草稿**
- **wiki/ 下始终是 LLM 生成或整理后的稳定知识页**
- 概念分层始终遵循 `llm-wiki` 的 `Raw Sources / Wiki / Schema` 三层；`notes/` 只是 `Wiki` 层里的工作态子层，不是独立第四层

## 核心规则

- **raw/** 内容只读，LLM 只读取，不修改
- **notes/** 是工作笔记层，允许用户与 LLM 共同编辑
- **wiki/** 由 LLM 完全维护，用户只读
- **ingest-state.yaml** 记录 raw 文件的摄入状态，代替在 raw frontmatter 中回写状态
- 每次操作后更新 `index.md` 和 `log.md`
- Wiki 页面使用 Obsidian `[[链接]]` 格式互相引用
- 优先保持 `llm-wiki` 三层分工清晰：`raw = source`，`wiki = compiled knowledge`，`schema = index + rules + state`
- **notes/** 也属于知识库图的一部分，但它属于 `Wiki` 层内部的工作台，不是独立 source layer
- **wiki/** 可以由 `raw` 直接沉淀而来，也可以由 `notes` 经过 `promote` 沉淀而来；但外部事实来源仍只认 `raw/`
- 所有外部事实、引用、结论的最终可追溯来源应回指 `raw/`，不要把 `notes/` 当成原始来源替代物

## 操作流程

### Ingest（摄入新资料）

1. 用户将资料放入 `raw/` 下对应子目录，或发送 URL 由 LLM 保存
2. LLM 读取资料，给出摘要要点，并在末尾附上阅读评级：
   - 🔵 扫一眼即可 — 信息量有限、观点普通、或与已有内容高度重叠
   - 🟡 值得追问 — 有新观点或数据，不需要读原文，但可以深挖
   - 🔴 建议深读 — 包含重要新概念、与已有内容矛盾、或影响实际决策
3. 若用户需要保留自己的阅读痕迹、问题、反驳或行动项，可在 `notes/` 下创建或更新对应笔记
4. 在 `wiki/` 下创建或更新相关页面（摘要页、概念页、实体页）
5. 更新 `index.md`
6. 更新 `ingest-state.yaml`，记录 source path、source_url、ingested、ingested_date、linked wiki pages
7. 在 `log.md` 追加一条记录：`## [日期] ingest | 资料标题`
8. 询问用户是否推送到 GitHub（`git add -A && git commit -m "ingest: 资料标题" && git push`）

### Note（记录个人笔记）

1. 用户可直接在 `notes/<主题>/` 下新增笔记，或让 LLM 根据 `raw/`/`wiki/` 内容代写初稿
2. 笔记内容优先记录个人理解、摘抄、疑问、反驳、任务、实验计划，而不是重复 raw 全文
3. 笔记必须显式链接其参考来源：`raw/` 原文、相关 `wiki/` 页面，或其他 `notes/`
4. 重要但尚未固化的观点保留在 `notes/`，不要过早写入 `wiki/`
5. 所有值得保留的 `notes/` 页面都应纳入知识图：更新 `index.md`，补齐链接、`sources`、`wiki_pages`、`status`
6. 若笔记引用外部文章、网页、论文或资料，优先补入 `raw/` 作为可信边界内的原始来源；`notes/` 只保存你的工作态理解
7. 在 `log.md` 追加一条记录：`## [日期] note | 笔记标题`
8. 若该笔记会继续演化，默认不单独推送；由用户决定是否与其他变更一起提交

### Promote（从笔记沉淀到 wiki）

1. 当 `notes/` 中某篇笔记形成稳定结论时，LLM 将其整理为 `wiki/` 页面，或合并进现有 `wiki/` 页面
2. 保留 `notes/` 中的过程性思考，不要求删除原笔记；只需补充其对应的 `wiki_pages`
3. 更新 `index.md`
4. 在 `log.md` 追加一条记录：`## [日期] promote | 笔记标题`
5. 询问用户是否推送到 GitHub

### Query（查询）

1. 读取 `index.md` 找到相关页面
2. 优先读取相关 `wiki/` 页面；若问题涉及个人判断、未定稿观点或进行中的思考，再补充读取相关 `notes/` 页面
3. 综合回答时需区分稳定知识与个人草稿，避免把 `notes/` 中未验证内容表述成定论
4. 有价值的回答可以作为新页面存入 `wiki/`，或先存入 `notes/` 待后续沉淀
5. 在 `log.md` 追加记录：`## [日期] query | 问题摘要`
6. 若产生了新 wiki 页面，询问用户是否推送到 GitHub

### Lint（健康检查）

1. 检查：页面间矛盾、孤立页面（无入链）、过时内容、缺失交叉引用、`notes/` 中长期未整理但高价值的草稿、应 `promote` 但尚未沉淀的笔记
2. 更新 `wiki/overview.md` 反映当前知识库整体状态
3. 追加记录：`## [日期] lint | 检查结果摘要`

## Raw 文件 Frontmatter 格式

所有保存到 `raw/` 的文件必须包含：

```markdown
---
title: 文章标题
source_url: https://...
saved: YYYY-MM-DD
tags: [主题]
---
```

`raw/` 中不记录摄入状态；摄入状态统一记录在 `ingest-state.yaml`。

## Ingest State 文件格式

`ingest-state.yaml` 维护原始资料的机器可读状态，例如：

```yaml
sources:
  - path: raw/ai/2026-05-07-claude-personal-guidance.md
    source_url: https://example.com/article
    ingested: true
    ingested_date: 2026-05-07
    wiki_pages:
      - wiki/ai/claude-personal-guidance.md
```

## Wiki 页面格式

```markdown
---
tags: [概念/实体/摘要/分析]
sources: []
updated: YYYY-MM-DD
---

# 页面标题

正文内容，使用 [[链接]] 引用相关页面。
```

## Notes 页面格式

所有保存到 `notes/` 的文件建议包含：

```markdown
---
title: 笔记标题
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags: [主题]
status: draft
sources: []
wiki_pages: []
---

# 笔记标题

正文内容，使用 [[链接]] 引用相关页面。
```

约定：

- `status` 可取：`draft`、`reviewed`、`promoted`
- `sources` 用于记录关联的 `raw/` 路径、URL 或其他来源
- `wiki_pages` 用于记录该笔记已沉淀到哪些 `wiki/` 页面
- `notes/` 允许保留不完整段落、待办项和问题清单，但应尽量避免无标题碎片

## 主题分类

主题子目录在 `raw/`、`notes/` 和 `wiki/` 下按需创建，不预设，随内容自然生长。

目录结构示例：
```
raw/
├── ai/
├── finance/
├── health/
└── assets/     ← 图片附件，不按主题细分
notes/
├── ai/
├── finance/
└── health/
wiki/
├── ai/
├── finance/
└── health/
```

`notes/` 不按来源组织，只按主题或项目组织。

## 来源分层规则

- `raw/ai/` 是 AI 资料默认归档区
- 当资料来源满足“官方一手、连续系列、长期追踪”时，可在 `raw/ai/` 下按来源分层
- 当前已确定采用来源分层的目录：
  - `raw/ai/anthropic/research/`
  - `raw/ai/anthropic/engineering/`
- 其他零散来源暂不单独建来源目录，继续放在 `raw/ai/`
- `wiki/` 仍按主题组织，不按来源组织

## 链接摄入流程

用户发送 URL 时：

1. 抓取链接内容
2. 判断所属主题，在 `raw/<主题>/` 下保存为 markdown 文件（含完整 frontmatter）
3. 若主题子目录不存在则自动创建
4. 跨主题资料放最相关的主题目录，边界模糊时询问用户
5. 文件命名：`YYYY-MM-DD-标题slug.md`
6. 继续执行 Ingest 流程（讨论要点 → 更新 wiki → 更新 index/log → 询问是否推送）

## Index 维护规则

- `index.md` 除了维护 `wiki/` 入口外，还应维护一个“个人笔记”区，只列出值得后续继续加工或频繁回看的 `notes/` 页面
- `notes/` 中纯临时、纯过程性、很快会删除的草稿不必全部进入 `index.md`
- 当某篇 `notes/` 已 `promoted` 且不再需要单独关注时，可从“个人笔记”区移除，保留其在 `wiki/` 中的沉淀结果
- `index.md` 的维护目标不是只做 wiki 目录，而是覆盖整个知识图的主要入口：稳定知识看 `wiki/`，工作态知识看 `notes/`
