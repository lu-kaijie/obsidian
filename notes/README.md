# Notes

`notes/` 是这套知识库的工作笔记层。

- `raw/` 保存外部原始资料，只读
- `notes/` 保存个人理解、摘记、问题、反驳、行动项和中间草稿
- `wiki/` 保存整理后的稳定知识页

建议按主题建子目录，例如：

```text
notes/
├── ai/
├── finance/
└── health/
```

当前与 AI 主题强相关的综述、面试材料、方法论笔记，优先放在 `notes/ai/`。

建议 frontmatter：

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
```

`status` 约定：

- `draft`：草稿，仍在思考
- `reviewed`：已经整理过一轮
- `promoted`：核心内容已沉淀进 `wiki/`
