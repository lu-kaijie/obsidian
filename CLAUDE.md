# CLAUDE.md — Wiki Schema

## 会话启动流程

每次新对话开始时：

1. 读取 `CLAUDE.md`（本文件）了解规范
2. 读取 `index.md` 了解当前 wiki 内容状态
3. 扫描 `raw/` 找出未处理文件，包含两种情况：
   - frontmatter 中 `ingested: false` 的文件
   - 直接位于 `raw/` 根目录下、缺少规范 frontmatter 的文件（来自 Obsidian Web Clipper）

   对于 Web Clipper 文件，先判断主题移动到对应子目录，再补全 frontmatter（`ingested: false`、`source_url`、`saved` 等），然后统一提示用户："发现 N 个未处理资料，是否现在 ingest？"
4. 读取 `inbox.md`，若其中有内容，提示用户："inbox 中有待处理链接，是否现在处理？"处理完成后删除已处理的行。
5. 读取 `log.md`，统计上次 lint 之后的 ingest 次数，若达到 10 次则提醒用户："距上次健康检查已摄入 10 篇资料，建议执行 lint。"
6. 等待用户指令

## 目录结构

```
/raw/          ← 原始资料（只读，LLM 不修改）
  /assets/     ← 图片等附件
/wiki/         ← LLM 维护的知识页面
  overview.md  ← 知识库整体概览（Lint 时更新）
index.md       ← Wiki 内容目录（每次 ingest 后更新）
log.md         ← 操作日志（仅追加）
CLAUDE.md      ← 本文件，schema 配置
```

## 核心规则

- **raw/** 内容只读，LLM 只读取，不修改
- **wiki/** 由 LLM 完全维护，用户只读
- 每次操作后更新 `index.md` 和 `log.md`
- Wiki 页面使用 Obsidian `[[链接]]` 格式互相引用

## 操作流程

### Ingest（摄入新资料）

1. 用户将资料放入 `raw/` 下对应子目录，或发送 URL 由 LLM 保存
2. LLM 读取资料，给出摘要要点，并在末尾附上阅读评级：
   - 🔵 扫一眼即可 — 信息量有限、观点普通、或与已有内容高度重叠
   - 🟡 值得追问 — 有新观点或数据，不需要读原文，但可以深挖
   - 🔴 建议深读 — 包含重要新概念、与已有内容矛盾、或影响实际决策
3. 在 `wiki/` 下创建或更新相关页面（摘要页、概念页、实体页）
4. 更新 `index.md`
5. 将 raw 文件的 `ingested` 改为 `true`，`ingested_date` 填写当天日期
6. 在 `log.md` 追加一条记录：`## [日期] ingest | 资料标题`
7. 询问用户是否推送到 GitHub（`git add -A && git commit -m "ingest: 资料标题" && git push`）

### Query（查询）

1. 读取 `index.md` 找到相关页面
2. 读取相关 wiki 页面，综合回答
3. 有价值的回答可以作为新页面存入 `wiki/`
4. 在 `log.md` 追加记录：`## [日期] query | 问题摘要`
5. 若产生了新 wiki 页面，询问用户是否推送到 GitHub

### Lint（健康检查）

1. 检查：页面间矛盾、孤立页面（无入链）、过时内容、缺失交叉引用
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
ingested: false
ingested_date:
---
```

ingested 完成后改为：
```yaml
ingested: true
ingested_date: YYYY-MM-DD
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

## 主题分类

主题子目录在 `wiki/` 和 `raw/` 下按需创建，不预设，随内容自然生长。

目录结构示例：
```
raw/
├── ai/
├── finance/
├── health/
└── assets/     ← 图片附件，不按主题细分
wiki/
├── ai/
├── finance/
└── health/
```

## 链接摄入流程

用户发送 URL 时：

1. 抓取链接内容
2. 判断所属主题，在 `raw/<主题>/` 下保存为 markdown 文件（含完整 frontmatter）
3. 若主题子目录不存在则自动创建
4. 跨主题资料放最相关的主题目录，边界模糊时询问用户
5. 文件命名：`YYYY-MM-DD-标题slug.md`
6. 继续执行 Ingest 流程（讨论要点 → 更新 wiki → 更新 index/log → 询问是否推送）
