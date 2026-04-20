# Marvis — LLM Wiki

A personal knowledge base where the LLM **compiles** knowledge into a persistent, interlinked wiki — not retrieves it on every query like RAG. Drop in source documents, and Claude Code reads, extracts, and weaves them into an ever-growing wiki of markdown pages with cross-references, contradiction flags, and layered synthesis. Browse the result in Obsidian.

## TL;DR — Three Things You'll Ever Type

Once the vault is cloned and opened in Claude Code, everything is just three commands:

```
# 1. Ingest — add a source to the wiki
ingest raw/articles/your-article.md

# 2. Ask — query the wiki in natural language
what's the relationship between X and Y?

# 3. Lint — periodic health check
lint
```

That's the entire surface area. Everything else in this README is explaining what those three commands do and why the output is worth keeping.

---

## Why Marvis

Most LLM-powered knowledge tools do one of two things:

| Tool class | How they work | What you end up with |
|---|---|---|
| **RAG systems** (ChatGPT Memory, Cursor's index, most "chat with your docs" apps) | Embed everything, retrieve at query time, regenerate answers on the fly | An invisible vector DB. Answers look fine but nothing is written down — you can't browse, audit, or share the knowledge |
| **AI-assisted note apps** (Notion AI, Obsidian Smart Connections, Logseq Copilot) | LLM helps you write individual notes; connections are manual or generated on demand | Your notes, plus some one-off summaries. The synthesis still lives in your head |
| **Marvis** | LLM reads sources once and **compiles** them into an explicit wiki; cross-references and contradictions are written to disk; every new source re-enriches existing pages | A growing, human-readable, git-tracked wiki you can open in any markdown editor. No embeddings, no hidden state |

**The wiki is the artifact.** It gets richer every time you feed it something new. You can diff it, grep it, publish it, hand it to a colleague, or just keep reading it in Obsidian. If the LLM disappeared tomorrow, your knowledge would still be there as plain markdown.

## Who This Is For (and Who It Isn't)

**Marvis is for you if:**
- You read/clip a lot of articles, papers, or docs on a few topics you care about
- You're already a Claude Code user (or willing to become one)
- You like Obsidian or plain-markdown workflows
- You want your knowledge base to be inspectable — one file per entity, one file per concept, git history for every change

**Marvis is probably not for you if:**
- You need multi-user collaboration — this is a single-user tool, merges and concurrent edits are out of scope
- You want everything to happen in one app — the ingest loop runs in Claude Code, browsing happens in Obsidian
- You're looking for zero-setup — you need a Claude Code subscription and some comfort with the terminal and git

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

## End-to-End Example

Say you drop `raw/articles/Pairing.md` into the vault and ask Claude Code to ingest it.

**What happens next:**

```
You: ingest raw/articles/Pairing.md

Claude Code:
  1. Reads CLAUDE.md (wiki schema)
  2. Reads raw/articles/Pairing.md
  3. Reads index.md to see what already exists
  4. Creates wiki/sources/pairing-source.md       ← new source summary
  5. Creates wiki/concepts/pairing.md             ← new concept page
  6. Updates wiki/entities/openclaw.md            ← enriches existing page with pairing details
  7. Updates wiki/entities/whatsapp-channel.md    ← adds the `pairing` DM policy
  8. Updates wiki/sources/whatsapp-source.md      ← back-links to the new concept
  9. Updates wiki/sources/chat-channels-source.md ← adds pairing to the safety section
 10. Updates index.md                             ← catalogs the new pages
 11. Appends log.md                               ← records the operation
 12. git commit -m "ingest: Pairing — created 2 pages, updated 4 pages"
```

**Now you can open Obsidian and:**
- Click `[[pairing]]` from any page that mentions it — goes straight to the concept
- See the Graph View show `openclaw`, `whatsapp-channel`, `pairing`, and the source pages all linked together
- Ask Claude Code "what's the difference between DM pairing and device pairing?" and get an answer citing `[[pairing]]` and `[[pairing-source]]`
- Later, ingest a new WhatsApp article — the existing `pairing` page gets updated in place, not duplicated

That is the compounding behavior. Every source makes the wiki denser, not longer.

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

# Marvis 中文操作手册

Marvis 是一个由 LLM 驱动的个人知识库系统。你负责提供原始资料和提问，Claude Code 负责所有的整理、归档、交叉引用和维护工作。你在 Obsidian 中浏览，Claude Code 在后台写作——wiki 会随着每次投喂自动生长。

## TL;DR——你一共只需要打三条命令

clone 好仓库、在 Claude Code 里打开之后，日常用法就是三条：

```
# 1. 投喂（Ingest）——把新资料加入 wiki
ingest raw/articles/你的文章.md

# 2. 提问（Ask）——用自然语言查询 wiki
X 和 Y 是什么关系？

# 3. 体检（Lint）——定期健康检查
lint
```

所有日常操作只有这三个动作。README 后面的内容都是在解释这三条命令背后发生了什么、为什么产出的东西值得留下来。

## 它和别的工具有什么不同

- **不是 RAG**——不是把资料塞进向量数据库再按问题召回。Marvis 在投喂时就把知识编译成显式的 wiki 页面，一次性写入磁盘，之后所有阅读和查询都是直接读这些 markdown 文件
- **不是 Notion AI / Obsidian Smart Connections**——那些工具帮你写"单篇笔记"，笔记之间的联系还得你自己脑补。Marvis 的重点是**跨源综合**——同一个实体或概念在不同资料里出现时，页面会被原地更新、矛盾会被并列标注
- **不是一站式 app**——投喂和维护在 Claude Code 里对话完成，浏览在 Obsidian 里做。两个工具各做擅长的事

**wiki 本身就是成品**。Claude Code 哪天用不了了，你的知识仍然是一堆干净的 markdown，随便用任何编辑器打开都能读。

## 它适合谁

**适合你，如果：**
- 你经常收集文章、论文、文档，内容集中在几个你关心的主题上
- 你已经是（或愿意成为）Claude Code 用户
- 你熟悉 Obsidian 或者习惯纯 markdown 工作流
- 你想要一个**可以被检查**的知识库：一个实体一个文件，一个概念一个文件，每次改动都有 git 记录

**不太适合你，如果：**
- 你需要多人协作——这是单人工具，合并和并发编辑不在设计目标里
- 你希望一个 app 搞定所有事——投喂在 Claude Code 里，浏览在 Obsidian 里
- 你讨厌命令行——需要 Claude Code 订阅，并且对终端和 git 有基本熟悉度

## 端到端例子

比如你把 `raw/articles/Pairing.md` 放进 vault，然后让 Claude Code 投喂。

```
你：ingest raw/articles/Pairing.md

Claude Code 的动作：
  1. 读 CLAUDE.md（wiki 规范）
  2. 读 raw/articles/Pairing.md
  3. 读 index.md 看现有页面
  4. 新建 wiki/sources/pairing-source.md        ← 新的 source 摘要
  5. 新建 wiki/concepts/pairing.md              ← 新的概念页
  6. 更新 wiki/entities/openclaw.md             ← 已有页面补入 pairing 相关内容
  7. 更新 wiki/entities/whatsapp-channel.md    ← 加入 pairing 的 DM 策略
  8. 更新 wiki/sources/whatsapp-source.md       ← 反向链接回新概念页
  9. 更新 wiki/sources/chat-channels-source.md  ← 在安全部分补充 pairing
 10. 更新 index.md                             ← 把新页面登记进目录
 11. 追加 log.md                               ← 记录这次操作
 12. git commit -m "ingest: Pairing — created 2 pages, updated 4 pages"
```

然后打开 Obsidian 你就能：
- 点任意提到 `[[pairing]]` 的地方，直接跳到概念页
- 在 Graph View 里看到 `openclaw`、`whatsapp-channel`、`pairing` 和各自的 source 页都互相连起来了
- 问 Claude Code "DM pairing 和 device pairing 有什么区别"，回答里会自动引用 `[[pairing]]` 和 `[[pairing-source]]`
- 以后再投喂一篇相关文章，`pairing` 这个页面会被**原地扩写**，不会出现重复页

这就是"每次投喂都让 wiki 更稠密而不是更长"的效果。

## 前置准备

- [Claude Code](https://claude.ai/claude-code)——负责读、写、维护 wiki 的 LLM
- [Obsidian](https://obsidian.md/)——浏览用（理论上任何 markdown 编辑器都行，但 Graph View 和 wikilink 让 Obsidian 是最合适的）
- Git——所有操作都走版本控制

## 快速开始

```bash
# 1. clone 仓库
git clone https://github.com/vwumumu/marvis.git
cd marvis

# 2. 在 Obsidian 里打开
#    File → Open Vault → 选 marvis 目录

# 3. 放入第一篇资料
#    把 markdown 文件丢进 raw/articles/

# 4. 让 Claude Code 投喂
#    在 marvis 目录下打开 Claude Code，说一句：
#    "ingest raw/articles/你的文章.md"

# 5. 在 Obsidian 里浏览结果
#    看 wiki/ 下的新页面，打开 Graph View 看连接
```

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

## 三大操作

### 1. Ingest（投喂）——加入新知识

把文件放进 `raw/`，然后对 Claude Code 说：

```
ingest raw/articles/你的文章.md
```

Claude Code 会：
- 读源文件，创建 source 摘要页
- 创建或更新实体页（人物、组织、工具）
- 创建或更新概念页（思想、框架）
- 更新 index、追加 log、git commit

### 2. Query（提问）——向 wiki 提问

```
X 和 Y 是什么关系？
帮我总结一下我看过的所有关于 Z 的资料。
A 和 B 的关键区别是什么？
```

回答会用 `[[wikilink]]` 引用 wiki 页面，Obsidian 里可点击。好的回答可以存到 `wiki/queries/` 或 `wiki/synthesis/`。

### 3. Lint（体检）——健康检查

每投喂 5-10 篇之后做一次体检：

```
lint
```

检查：断链、孤立页、应该存在但缺失的实体页、index 完整性、frontmatter 规范、矛盾点。

## 获取资料的方式

- **[Obsidian Web Clipper](https://obsidian.md/clipper)**——浏览器插件，一键把网页转成 markdown
- **复制粘贴**——手动存成 `.md`
- **PDF → markdown**——转换后丢进 `raw/`

## 典型用途

- **课题研究**——喂入多篇论文和文章，形成知识地图
- **读书笔记**——按章节投喂，自动构建人物、主题、情节的互联页面
- **竞品分析**——投喂多份资料，wiki 自动维护对比和矛盾点
- **课程学习**——投喂讲义和阅读材料，自动构建知识框架

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
