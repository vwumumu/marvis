# Marvis — LLM Wiki

A personal knowledge base where the LLM **compiles** knowledge into a persistent, interlinked wiki — not retrieves it on every query like RAG. Drop in source documents, and Claude Code reads, extracts, and weaves them into an ever-growing wiki of markdown pages with cross-references, contradiction flags, and layered synthesis. Browse the result in Obsidian.

---

## How It Works

```
You drop a source file        Claude Code processes it         You browse in Obsidian
into raw/articles/       →    creates/updates wiki pages   →   with Graph View, search,
                              updates index, commits git       and wikilinks
```

Unlike RAG (retrieve-and-generate on every query), Marvis **pre-compiles** knowledge:
- Cross-references are built at ingest time, not query time
- Contradictions between sources are flagged immediately
- Each new source enriches existing pages — knowledge compounds

## Prerequisites

- [Claude Code](https://claude.ai/claude-code) — the LLM that reads, writes, and maintains the wiki
- [Obsidian](https://obsidian.md/) — the browsing interface (any markdown editor works, but Obsidian's Graph View and wikilinks make it ideal)
- Git — all operations are version-controlled

## Quick Start

```bash
# 1. Clone the repo
git clone https://github.com/vwumumu/marvis.git
cd marvis

# 2. Open in Obsidian
#    File → Open Vault → select the marvis directory

# 3. Add your first source
#    Save a markdown article to raw/articles/

# 4. Ingest it with Claude Code
#    Open Claude Code in the marvis directory and say:
#    "ingest raw/articles/your-article.md"

# 5. Browse the result in Obsidian
#    Check wiki/ for new pages, open Graph View to see connections
```

## Directory Structure

```
marvis/
├── CLAUDE.md          # Wiki schema — Claude Code reads this for all conventions
├── index.md           # Index — categorized catalog of all wiki pages
├── log.md             # Operations log — chronological record
├── raw/               # Source documents (read-only, never modified by LLM)
│   ├── articles/      # Web articles
│   ├── papers/        # Academic papers
│   ├── notes/         # Personal notes
│   ├── books/         # Book chapters/excerpts
│   └── assets/        # Image attachments
├── wiki/              # Wiki pages (maintained by Claude Code)
│   ├── sources/       # Source summaries (one per ingested source)
│   ├── entities/      # Entity pages (people, orgs, tools, places)
│   ├── concepts/      # Concept pages (ideas, frameworks, themes)
│   ├── synthesis/     # Cross-cutting analyses and comparisons
│   └── queries/       # Persisted Q&A answers
└── .obsidian/         # Obsidian config
```

## Three Core Operations

### 1. Ingest — Add Knowledge

Put a source file in `raw/`, then tell Claude Code:

```
ingest raw/articles/your-article.md
```

Claude Code will:
- Read the source and create a source summary page
- Create or update entity pages (people, orgs, tools)
- Create or update concept pages (ideas, frameworks)
- Update the index, append the log, commit to git

**Tips:**
- Ingest one source at a time for quality control
- Files in `raw/` are never modified — they're your immutable originals
- If a new source mentions an existing entity/concept, the existing page gets updated (no duplicates)
- Contradictions between sources are preserved with citations from both sides

### 2. Query — Ask Questions

Ask Claude Code anything about your wiki:

```
What's the relationship between X and Y?
Summarize everything I've read about Z.
What are the key differences between A and B?
```

Answers cite wiki pages with `[[wikilinks]]` — clickable in Obsidian. Valuable answers can be persisted to `wiki/queries/` or `wiki/synthesis/`.

### 3. Lint — Health Check

After every 5-10 ingests, run a health check:

```
lint
```

Checks for: broken wikilinks, orphan pages, missing entity pages, index completeness, frontmatter validity, and contradictions.

## Getting Source Files

- **[Obsidian Web Clipper](https://obsidian.md/clipper)** — browser extension, one-click article → markdown
- **Copy-paste** — save text as `.md` files manually
- **PDF → markdown** — convert and drop into `raw/`

## Use Cases

- **Research** — feed papers and articles on a topic, build a comprehensive knowledge map
- **Book notes** — ingest chapter by chapter, build interconnected pages for characters, themes, and plot
- **Competitive analysis** — feed multiple sources, let the wiki maintain comparisons and contradictions
- **Course learning** — ingest lecture notes and readings, auto-build a knowledge framework

## License

[MIT](LICENSE)

---

# Marvis 操作手册

Marvis 是一个由 LLM 驱动的个人知识库系统。你负责提供原始资料和提问，Claude Code 负责所有的整理、归档、交叉引用和维护工作。你在 Obsidian 中浏览，Claude Code 在后台写作——wiki 会随着每次投喂自动生长。

## 日常工作流

```
Obsidian（浏览端）              Claude Code（操作端）
     │                              │
     │  1. 把文章放到 raw/articles/  │
     │                              │
     │  2. "ingest raw/articles/xx.md"
     │                              │
     │         ← 自动创建/更新页面 → │
     │                              │
     │  3. 在 Obsidian 中浏览       │
     │     - 查看新页面             │
     │     - Graph View 看连接      │
     │     - 检查质量               │
     │                              │
     │  4. "这个概念和那个有什么关系？"
     │                              │
     │         ← 综合回答 →          │
     │                              │
     │  5. "把这个回答存下来"        │
     │                              │
     │  6. 积累一段时间后："lint"     │
     │                              │
     │         ← 健康报告 →          │
```

## 注意事项

1. **一次投喂一个源文件**——方便检查质量，防止错误传播
2. **定期在 Obsidian 中浏览**——人工抽检是保证质量的关键
3. **raw/ 目录永远不要修改**——这是你的原始资料备份
4. **所有操作都有 git 记录**——可以随时回溯任何变更
5. **wiki 页面由 Claude Code 维护**——你可以阅读，但一般不需要手动编辑
6. **CLAUDE.md 是核心规范**——每个新的 Claude Code 会话都会读取它来了解所有约定

## Obsidian 使用技巧

- **Graph View**：查看 wiki 的整体结构和页面之间的连接关系
- **反向链接面板**：查看哪些页面链接到了当前页面
- **Dataview 插件**：对 YAML frontmatter 做动态查询
- **搜索**：全文搜索 wiki 内容
- **附件下载**：在 Obsidian 设置中已配置附件保存到 `raw/assets/`
