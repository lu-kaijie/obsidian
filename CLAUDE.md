# CLAUDE.md — Wiki Schema

## 目录结构

```
/raw/          ← 原始资料（只读，LLM 不修改）
  /assets/     ← 图片等附件
/wiki/         ← LLM 维护的知识页面
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

1. 用户将资料放入 `raw/` 下对应子目录
2. LLM 读取资料，与用户讨论要点
3. 在 `wiki/` 下创建或更新相关页面（摘要页、概念页、实体页）
4. 更新 `index.md`
5. 在 `log.md` 追加一条记录：`## [日期] ingest | 资料标题`

### Query（查询）

1. 读取 `index.md` 找到相关页面
2. 读取相关 wiki 页面，综合回答
3. 有价值的回答可以作为新页面存入 `wiki/`
4. 在 `log.md` 追加记录：`## [日期] query | 问题摘要`

### Lint（健康检查）

检查：页面间矛盾、孤立页面（无入链）、过时内容、缺失交叉引用
追加记录：`## [日期] lint | 检查结果摘要`

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
