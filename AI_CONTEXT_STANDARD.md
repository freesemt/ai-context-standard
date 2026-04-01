# AI Context Management Standard (Proposed)

**Status**: Proposal for community discussion  
**Author**: Discovered through practical use across multiple repositories  
**Date**: March 26, 2026  
**Version**: 0.8.6 (Draft)

---

> ### 🎯 この標準の核心
>
> **コンテキストはリポジトリで管理する。**
>
> AI ツールの個人記憶やローカル設定ではなく、リポジトリ内のファイル（`copilot-instructions.md`、`PROJECT_STATUS.md`）に書く。
> これにより、チームメンバーや他の AI ツール、将来のセッションでも同じコンテキストが再現される。
>
> → **「どこに書くか」を迷ったときは、この原則に立ち返れ。**

---

## How to Use This Standard

### For Humans

1. **Understand the principle** (5 minutes): Separate STATIC conventions from DYNAMIC state
2. **Know your project**: Its nature, structure, and what matters
3. **Collaborate with AI**: Explain your project, discuss what's needed, create together

### For AI Assistants

1. **Learn the principle**: STATIC (conventions) in INIT, DYNAMIC (current state) in STATUS
2. **Listen to the user**: Understand the project's specific characteristics
3. **Propose and refine**: Suggest structure, discuss choices, adapt to needs

### The Collaboration

**This standard is axiomatic, not prescriptive.**

Like mathematical axioms, it provides minimal essential principles:
- **Core axiom**: Separate STATIC (conventions) from DYNAMIC (state)
- **Implementation**: Derived through collaboration, adapted to context
- **Templates**: Examples to illustrate principles, not rules to copy
- **Discovery**: Each project finds its optimal structure through dialogue

**Human-AI partnership**:
- Human provides direction and judgment
- AI provides structure and implementation
- Together, discover what works for this project

**Why axiomatic/simple**:

Detailed templates would enable "apparent high performance" (perfect template application) but prevent "true high performance" (creative adaptation through understanding).

AI assistants can execute detailed instructions perfectly—but that's mechanical application, not collaborative discovery. Simple principles require genuine understanding and partnership, which produces better results for each unique project.

This standard stays minimal to enable true collaboration, not prescriptive compliance.

---

## Problem Statement

There is currently **no universal mechanism** for maintaining AI assistant context across sessions in repositories:

- Different AI tools (GitHub Copilot, Claude, ChatGPT Code Interpreter) have different context systems
- Each session starts fresh, requiring re-explaining conventions, structure, and patterns
- Context drift accumulates as projects grow larger and more complex
- No standard way to communicate "how to work here" to AI assistants

**Current state:**

GitHub Copilot (VS Code) supports `.github/copilot-instructions.md` — a file that is **automatically loaded** at the start of every Agent mode session. This is the native `.copilotrc`. The initialization problem is solved.

**Remaining gaps:**
- Separation of static conventions from dynamic state (no platform solves this)
- Session resumption (knowing where work was left off)

---

## Proposed Solution: copilot-instructions.md + STATUS.md Pattern

### Core Concept

Two complementary files with clear separation of concerns:

**1. `.github/copilot-instructions.md`** — **STATIC CONTENT**
- **Purpose**: How to work in this repository
- **Content**: Working conventions, documentation structure, frameworks, patterns
- **Update frequency**: Rarely (major restructuring only)
- **Nature**: Static reference — changes only when conventions change
- **Loading**: Automatically loaded by GitHub Copilot at every Agent mode session start
- **Key principle**: If you wrote it last month and it's still true today → belongs here

**2. `PROJECT_STATUS.md`** — **DYNAMIC CONTENT**
- **Purpose**: What's happening now
- **Content**: Current focus, recent work, chronological history, next steps
- **Update frequency**: Frequently (every work session)
- **Nature**: Dynamic tracking — changes as work progresses
- **Loading**: Read automatically at session start via directive in `copilot-instructions.md` (see below)
- **Key principle**: If it will be outdated next week → belongs in STATUS

No initialization phrase is required. GitHub Copilot loads `copilot-instructions.md` automatically at every session start. `PROJECT_STATUS.md` is loaded via an explicit directive inside `copilot-instructions.md`:

```markdown
> **On every session start**: Read [`PROJECT_STATUS.md`](PROJECT_STATUS.md) to get the current task and recent context before responding.
```

### 3. Initialization Visibility: `init.prompt.md` (Optional but Recommended)

**Problem**: `copilot-instructions.md` is loaded **silently and automatically**. Unlike the old "magic phrase" approach, there is no visible confirmation that initialization has occurred or what was loaded.

**Solution**: Create `.github/prompts/init.prompt.md` with `alwaysApply: true` — runs automatically on every new chat and makes initialization visible.

**VS Code バージョン記録ファイル**: `.github/vscode-version.txt`（自動更新推奨）

`vscode-version-recorder` 拡張機能（https://github.com/freesemt/vscode-version-recorder）をインストールすると、VS Code 起動時に自動で更新されます。

```text
# VS Code version used in this workspace
# Auto-updated by vscode-version-recorder extension
# Required for alwaysApply: true in init.prompt.md (VS Code 1.99+)
1.114.0-insider
```

**インストール方法**（`gh` CLI が必要）:
```powershell
gh release download v0.1.0 --repo freesemt/vscode-version-recorder --pattern "*.vsix" --dir $env:TEMP
code --install-extension "$env:TEMP\vscode-version-recorder-0.1.0.vsix"
```

インストール後 VS Code を再起動すると、以降は自動で `vscode-version.txt` が更新されます。

**Note**: 拡張機能を使わない場合は手動でバージョンを記録することも可能です。`init.prompt.md` が拡張機能の有無を自動検出し、未インストールの場合にインストールを案内します。

```markdown
---
agent: copilot
description: セッション初期化（PROJECT_STATUS.md の読み込みと確認）
alwaysApply: true
---

## ステップ1: VS Code バージョン確認

`<リポジトリ名>` リポジトリの `.github/vscode-version.txt`（このプロンプトファイルと同じリポジトリ）を読んでください。

- **バージョン番号が記載されている場合**: そのバージョンを使用してステップ2へ
- **バージョン番号が未記載またはファイルが存在しない場合**: 以下をユーザーに表示してください：

  > ⚠️ **VS Code バージョンの記録をお願いします**  
  > メニュー → **ヘルプ** → **バージョン情報** → 最初の行の番号（例: `1.114.0-insider`）を  
  > `.github/vscode-version.txt` の末尾に追記してください。

## ステップ2: alwaysApply 対応チェック

取得したバージョンの数値部分（例: `1.114.0` の `1.114`）が **1.99 以上** かどうか判定してください。

- **1.99 以上** → ✅ `alwaysApply: true` が有効です。自動初期化は正常に動作しています。
- **1.99 未満** → ⚠️ `alwaysApply: true` は動作しません。毎回 `/init` を手動実行してください。

## ステップ3: PROJECT_STATUS.md の要約

`PROJECT_STATUS.md` を読んで、以下を日本語で要約してください：

1. **現在のタスク** — 何に取り組んでいるか
2. **直近の進捗** — 最近完了したこと
3. **次のステップ** — 優先度順に

最後に「**初期化完了** (リポジトリ名) / VS Code vX.XX.X ✅」と明記してください。
```

**Usage**: `alwaysApply: true` により **新しいチャット開始時に自動実行**されます（VS Code 1.99+）。`/init` で手動再実行も可能です。

**Role separation**:

| ファイル | 役割 | 動作 |
|----------|------|------|
| `copilot-instructions.md` | 静的規約・構造 | 常時・自動・サイレント |
| `init.prompt.md` (`alwaysApply: true`) | 動的状態の読み込み確認 + バージョンチェック | 新チャット開始時・自動・可視フィードバック付き |
| `vscode-version.txt` | VS Code バージョン記録 | 手動更新・`init.prompt.md` から参照 |

**Multi-root workspace**: In a workspace with multiple repositories, each `init.prompt.md` runs automatically on new chat start, reading that repo's `PROJECT_STATUS.md`. `/init` can also be used for manual re-initialization mid-session.

---

## Working Conventions for AI Assistants

**この標準の核心原則に立ち返れ**: AI の操作規約は、この設計文書ではなく、**各リポジトの `copilot-instructions.md`** に記載する。
そこに書いて初めて、初期化で自動読み込まれ、AI に実際に機能する。

以下は「`copilot-instructions.md` に記載すべき内容」の記述例であり、この文書としてのリファレンスにすぎない。

### 例: Failure Recovery Protocol

When you fail the same operation 3+ times, **stop and explain the situation** to the user.

**Applies to**:
- Large-scale file editing (100+ line deletions/insertions)
- Repeated tool failures (API calls, searches, file operations)
- Non-essential tasks where manual operation might be faster
- Any repetitive task causing friction

**Actions**:
1. Stop the failing approach
2. Explain what you tried and why it's failing
3. Propose alternatives:
   - Manual operation (user does it directly)
   - Different tool or approach
   - Accepting current state if outcome is acceptable
4. Don't silently struggle through 15+ failed attempts

**Real example**: Attempting to delete 1370 lines with replace_string_in_file → Failed 15+ times → Should have stopped at attempt 3-4 and proposed manual deletion.

### 例: PowerShell Terminal ・ Multi-repo Git Operations

When operating on multiple repositories in sequence, **always use `git -C <path>`** instead of `cd <path>; git ...`.

**Reason**: The terminal tool may silently strip `cd` from chained commands, causing git to run in the wrong directory.

```powershell
# ❌ Unreliable
cd C:\path\to\repo; git commit -m "..."

# ✅ Reliable
git -C C:\path\to\repo commit -m "..."
```

---

## Implementation Guide

### Step 1: Create `.github/copilot-instructions.md` (STATIC Conventions)

**Approach**: Tell AI about your project. Let it propose structure. You provide judgment.

The following structure is guidance, not a rigid template. Not all sections are required. Adapt to your project's needs.

Recommended sections:

```markdown
# AI Assistant Initialization Guide

**Purpose**: Initialize AI context for working with this repository

## 📚 Core Documents to Read
[List 5-10 essential documents AI should read]

## 🔑 Working Conventions
[Document your conventions: notation, symbols, patterns]

## 🎯 Repository Context
[Mission, key insights, current focus]

## 📂 Structure Overview
[Quick reference to directory structure]

## 💡 Quick Tips for AI Assistants
[Usage patterns, common tasks]
```

**Customize to your domain:**
- Programming conventions (code style, testing patterns)
- Documentation standards (markdown, docstrings)
- Domain-specific notation (mathematical, scientific)
- Evidence standards (citations, proofs, experiments)

**Why this step?**
- Separates "how to work here" from "what this project is" (README) and "what's happening now" (STATUS)
- Provides single source of truth for conventions that rarely change
- Automatically loaded by GitHub Copilot — no action required from the user
- Human-readable onboarding document for team members

### Step 2: Create or Reorganize PROJECT_STATUS.md (DYNAMIC State)

**Approach**: This tracks what's happening now and evolution over time. Structure emerges naturally as you work.

Recommended sections:

```markdown
# Project Status

**Last Updated**: [Date]

## 🎯 Current Task

Working on: [Specific task description]
Next: [Immediate action to take]
See: [Relevant files]

---

## 🎯 Recent Work
[Chronological log of major changes]

## ⏳ Next Steps
[Priorities, planned work]

## 📊 Status Summary
[Overview of completeness, blockers]
```

**Why this step?**
- Tracks dynamic state (current work, recent changes) separate from static conventions
- Enables chronological narrative of project evolution
- Supports session resumption with "Current Task" section
- Lower maintenance burden - update frequently without touching conventions
- Natural place for "what's happening now" that complements `copilot-instructions.md`'s "how to work here"

### Step 3: Add Reference to README.md

Optional but helpful:

```markdown
> **For AI assistants (GitHub Copilot)**: Context is loaded automatically from `.github/copilot-instructions.md`.
```

**Why this step?**
- Helps team members and new contributors discover how AI context works in this repo
- Low overhead — single line in README
- Aids onboarding without cluttering project overview

### Step 4: Declare Standard Version (Recommended)

**Add version declaration to your INIT file header:**

```markdown
<!-- AI Context Standard v0.2 - Adopted: 2026-02-07 -->
# AI Assistant Initialization Guide
```

**For single-file variant:**
```markdown
<!-- AI Context Standard v0.2 - Adopted: 2026-02-07 -->
# Session Context for Project
```

**Purpose:**
- Know which standard version you're using months/years later
- Detect when standard has been updated (compare with canonical version)
- Plan migration if breaking changes occur
- Track adoption date for maintenance planning

**Where to put it:**
- In `.github/copilot-instructions.md` right before the main heading
- Comment format won't be rendered in markdown viewers

**Optional: Track in README badge:**
```markdown
![AI Context Standard](https://img.shields.io/badge/AI%20Context%20Standard-v0.2-blue)
```

**Why this step is important:**
- Enables discovery of standard updates
- Documents which iteration you tested
- Helps compare approaches across repositories
- Makes compliance audits faster

### Step 5: Add Session Resumption Support

The initialization pattern handles the start of work, but what about resuming work between sessions?

**Add "Current Task" section to PROJECT_STATUS.md** (right after header):

```markdown
## 🎯 Current Task

Working on: [Specific task description]
Next: [Immediate action to take]
See: [List of relevant files]
```

**Document the Derivation Rule** (in `copilot-instructions.md` as a working convention):

1. **Read latest achievement entry** in PROJECT_STATUS.md
2. **Check status**:
   - If ✅ Complete → Identify next logical task from context
   - If 🔬 In progress → That's the current task to resume
3. **Populate section** with concrete information (not placeholders)

**Session Start Protocol**:
```
[GitHub Copilot loads copilot-instructions.md automatically]
[AI reads PROJECT_STATUS.md automatically (directive in copilot-instructions.md)]
User: "I want to resume the current task"  ← or just start working
AI: [already has current task from STATUS, continues from there]
```

**Update Timing**:
- Human updates "Current Task" at session end when status changes (takes ~30 seconds)
- AI derives current task from latest achievement entry when resuming
- Keeps single source of truth without manual duplication

**Example Logic**:
- Latest achievement: "Bullet 1 Verification Complete ✅"
- Derived current task: "Bullet 2 Verification"
- Concrete details: "Design systematic experiments following Bullet 1 template approach"
- Relevant files: Links to verification roadmap, templates, reference implementations

**Benefits**:
- No need to say "continue where I left off yesterday" (temporal ambiguity)
- No manual tracking of "what was I doing" (derives from context)
- Works even after days/weeks away from project
- Natural progression from completed work to next task

---

### Step 6: Verify Compliance with Standard

After implementing Steps 1-5, verify that your files comply with the standard.

**Note**: See "Working Conventions for AI Assistants" section above for general failure recovery protocol.

**Copy-paste prompt for AI verification:**

````markdown
I want to check if my repository complies with the AI Context Standard (copilot-instructions.md + STATUS.md pattern).

Please read these three files and verify compliance:
1. README.md
2. `.github/copilot-instructions.md`
3. PROJECT_STATUS.md

**Check each file against these criteria:**

## Version Declaration Check

✅ Recommended:
- [ ] INIT file has version comment (e.g., `<!-- AI Context Standard v0.2 -->`)
- [ ] Version matches current standard or has migration plan
- [ ] Adoption date recorded (helps track maintenance timing)

## `.github/copilot-instructions.md` Compliance

✅ Required elements:
- [ ] Contains purpose statement explaining how to initialize
- [ ] Lists core documents to read (or references complementary files)
- [ ] Documents working conventions
- [ ] Provides repository context
- [ ] Includes structure overview (or appropriate for project size)
- [ ] Has quick tips for AI assistants

❌ Should NOT contain:
- [ ] Current work status or "current focus"
- [ ] Recent changes or dates (except version history)
- [ ] Chronological narrative
- [ ] Temporary decisions or this-week plans

## PROJECT_STATUS.md Compliance

✅ Required elements:
- [ ] Last updated date
- [ ] Current Task section (working on, next, see)
- [ ] Chronological log of recent work
- [ ] Dynamic state (what's happening now)
- [ ] References to `copilot-instructions.md` for conventions

❌ Should NOT contain:
- [ ] How-to instructions or code style
- [ ] Static conventions or notation definitions
- [ ] Directory structure (unless it changed recently)
- [ ] Permanent patterns or frameworks

## README.md Compliance

✅ Required elements:
- [ ] What the project is (purpose, mission)
- [ ] Why it exists (motivation, problem)
- [ ] Who it's for (audience guidance)
- [ ] How to get started (installation, basic usage)

❌ Should NOT contain:
- [ ] AI initialization instructions (→ copilot-instructions.md)
- [ ] Current status or work-in-progress (→ PROJECT_STATUS)
- [ ] Detailed conventions or notation (→ copilot-instructions.md)

Optional but recommended:
- [ ] One-line reference to `copilot-instructions.md` for team discovery

## Binary File Handling Check (Optional, if applicable)

✅ If repository contains binary documents (PDF/DOCX/etc.):
- [ ] Extraction tools exist (e.g., read_pdfs.py, read_docs.py)
- [ ] INIT file documents automatic handling rules for AI assistants
- [ ] Tool documentation includes "AI Assistant Usage" section
- [ ] Tools are discoverable (in tools/, scripts/, or documented location)
- [ ] AI can find and use tools without manual instruction each session

❌ Red flags:
- [ ] Binary files exist but no extraction tools provided
- [ ] Tools exist but not documented in INIT
- [ ] AI documentation missing from tool README
- [ ] User would have to manually explain tool usage each session
- [ ] `read_file` failures on binary files visible to user

**Note**: This check applies only if your repository needs to work with binary document formats. If your project doesn't use PDFs, Word documents, or similar binary files, skip this section.

**Content Boundary Check (Decision Tree):**

For each major section in each file, verify:
- "HOW to work here" → Should be in `copilot-instructions.md`
- "WHAT's happening now" → Should be in PROJECT_STATUS  
- "WHAT is this project" → Should be in README

**Generate a report with:**
1. Version declaration status (present, absent, outdated)
2. Compliance status for each file (✅ Compliant / ⚠️ Minor issues / ❌ Major violations)
3. List of violations found (with severity and location)
4. Recommended fixes for each violation
5. Migration recommendation if standard version is outdated
6. Overall assessment of whether standard is properly regulating the files

**Example violation:**
- File: `.github/copilot-instructions.md`
- Issue: Contains "Current focus: Implementing feature X" (dynamic state in static file)
- Severity: Minor
- Fix: Move to PROJECT_STATUS.md "Current Task" section
````

**When to run this audit:**
- After initial implementation of the pattern
- Before proposing your repo as reference implementation
- Quarterly maintenance check
- When you suspect content drift

**Expected output:** Clear compliance report identifying any violations with specific fixes

---

## AI Assistant Behavior on Initialization

**Target audience**: AI assistants, developers customizing AI behavior

### On Session Start (GitHub Copilot)

`.github/copilot-instructions.md` is loaded automatically. On every Agent mode session:

1. **Conventions are already loaded** — begin working according to them
2. **Do NOT read AI_CONTEXT_STANDARD.md** (this document is for humans setting up repos)
3. **Check for PROJECT_STATUS.md** — read it to orient to current work state
4. **Check version comment** if present: `<!-- AI Context Standard v0.X -->`
5. **Brief version notification** if mismatch detected (one line, non-intrusive)

**Note on visibility**: Initialization is silent by design. If the user wants to confirm what was loaded and get a STATUS summary, they can run `/init` (= `.github/prompts/init.prompt.md`). This replaces the old "magic phrase" initialization confirmation.

### Version Mismatch Notification (Optional, Keep Brief)

**If repository uses older standard version**:

```
✅ Initialized. (Standard v0.1 adopted, v0.2 available - backward compatible)
```

- One line, non-intrusive
- Mention only if significant version gap
- User can ask for details if interested
- Don't interrupt workflow unnecessarily

**If no version declaration**:
- Just initialize silently
- Optional: Mention once if asked about standard compliance

### Notebook Cell Output Reading

**Problem**: The built-in `read_notebook_cell_output` tool has a hard size limit. For cells with substantial stdout (e.g., print-based diagnostic output), it silently returns "output too large" and provides nothing useful.

**Routing rule** (prevents the failure rather than recovering from it):

1. **Check mime types first**: After `copilot_getNotebookSummary`, inspect each cell's output mime types
2. **If `application/vnd.code.notebook.stdout` is present** → do NOT call `read_notebook_cell_output`; use the script below instead
3. **If only `image/png`, `text/plain`, or similar non-stdout types** → `read_notebook_cell_output` is fine

**Recommended tool**: [`ai-context-tools`](https://github.com/freesemt/ai-context-tools) — a zero-dependency Python package that reads cell outputs directly from the `.ipynb` JSON file, bypassing the tool size limit entirely.

**Install**:
```bash
pip install ai-context-tools
```

**Usage** (after install):
```
python -m aic_tools.notebook experiments/08_foo/notebook.ipynb 14
python -m aic_tools.notebook experiments/08_foo/notebook.ipynb 14 0   # all lines
```

**Why this works**: `.ipynb` files are plain JSON. The tool reads cell outputs directly from disk — no tool size limit applies.

**Per-repo copilot-instructions.md reference** (propagation template):
```markdown
### Notebook cell output reading

Use `python -m aic_tools.notebook <notebook_path> <cell_number> [max_lines]`
instead of the built-in `read_notebook_cell_output` tool for any cell that has
`application/vnd.code.notebook.stdout` in its mime types.
Install once: `pip install ai-context-tools`
```

**Last resort**: Only ask the user to paste output if the tool is unavailable (e.g., no Python environment yet).

**Propagation requirement**: Working conventions defined in this standard are only effective if they appear in loaded context at session start. The standard document itself is intentionally *not* loaded during initialization. For any convention to be followed consistently, it must be present in `copilot-instructions.md`. Repo maintainers should copy relevant conventions (such as this one) into their `copilot-instructions.md` when adopting them.

---

### Binary File Handling

**When user requests to read a file:**

1. **Check file extension first**:
   - `.pdf`, `.docx`, `.xlsx`, `.pptx`, etc. → Binary format
   - Do NOT attempt `read_file` on these formats

2. **Search for extraction tools** in workspace:
   - Look for scripts like `read_pdfs.py`, `read_docs.py`, `extract_*.py`
   - Check `tools/`, `scripts/`, or root directories
   - Examine README or tool documentation for usage

3. **Use appropriate method automatically**:
   - If extraction tool exists → Execute it with the file path
   - Check if tool outputs to file or stdout
   - If no tool exists → Inform user and suggest creating one
   - **Never** show user a failed `read_file` attempt on binary files

4. **Keep user experience seamless**:
   - Don't ask "should I use the PDF tool?"
   - Just execute the appropriate tool and return content
   - User should not need to know about implementation details
   - Only mention the tool if execution fails or clarification needed

**Philosophy**: Users should be able to say "read the PDF" and get the content, regardless of file format. AI assistants should handle format detection and tool selection transparently.

### Context Load Considerations

⚠️ **Concern**: As standards evolve and repositories grow, initialization context may become large.

**Current mitigation**:
- Standard document (~47K tokens) is NOT part of initialization
- Only repository-specific files (~1-5K tokens typically) are loaded
- Version info is in simple comment (minimal parsing)

**Future consideration**:
- Monitor initialization time and context consumption
- Consider lazy loading for large repositories
- May need optimization if pattern shows performance issues

**Philosophy**: Keep initialization fast and simple. Standard compliance should not burden daily workflow.

---

### Step 7: Optimize Large Documents for AI Context Efficiency

**When this applies**: Documents reaching 500+ lines where AI discussions focus on specific topics (not the entire document).

#### Problem Pattern

Large documents (500-2000 lines) cause AI context inefficiency:
- Discussing "Topic A" loads entire document (including irrelevant Topics B-Z)
- Token waste: 60-90% of loaded content is unused
- Example: 1222-line governance document → discussing one issue loads everything

**Real case** (kannondai-community repository, 2026-02-07):
- Document: officer_system_discussion.md (1222 lines)
- Topics: 11 distinct issues (fee reform, crisis response, annual report, etc.)
- Problem: Discussing "Group 6 crisis" loaded entire 1222 lines
- Waste: ~800 lines were about other topics (会費改定, 論点A-H, 参考資料, etc.)

#### Solution: Topic-Based File Splitting

**Split large document into focused topic files + navigation hub**

**Structure**:
1. **Topic files**: 100-300 lines each, self-contained
2. **Navigation hub**: Main file becomes 200-400 line index with summaries and links
3. **Cross-references**: Each file links to related files

**When to apply**:
- ✅ Document exceeds 500 lines
- ✅ Covers multiple distinct topics/issues
- ✅ AI discussions focus on specific sections (not entire document)
- ✅ Topics can be understood somewhat independently
- ❌ Document is linear narrative requiring full context
- ❌ All parts are tightly interdependent

#### Implementation Approach

**User delegates to AI** (recommended for 500-2000 line documents):

```
"This document is too large for efficient AI context management. Please:
1. Analyze structure (use grep_search for section headings)
2. Propose topic-based split (10-15 files, 100-300 lines each)
3. Create topic files (extract content with read_file + create_file)
4. Transform main file into navigation hub (summaries + links)
5. Add cross-references between related files"
```

**AI working pattern**:
1. `grep_search` to map document structure (main sections)
2. `read_file` to extract content from line ranges
3. `create_file` for each topic (100-300 lines, self-contained)
4. Transform main file with `replace_string_in_file` (remove details, add navigation)
5. **For large deletions (100+ lines)**: Use Python script approach (see below)
6. **Stop at 3 failures** and escalate to user

**Large Deletion Strategy (100+ lines)**:

⚠️ **Known Issue**: `run_in_terminal` tool is unreliable on Windows PowerShell (output capture fails, cannot verify execution).

**Recommended approach for Windows**:
1. Create Python deletion script in `tools/` directory:
   ```python
   # Keep first N lines of file
   with open(file_path, 'r', encoding='utf-8') as f:
       lines = f.readlines()
   lines_to_keep = lines[:N]
   with open(file_path, 'w', encoding='utf-8') as f:
       f.writelines(lines_to_keep)
   print(f"✓ Kept {len(lines_to_keep)} lines, deleted {len(lines) - len(lines_to_keep)} lines", flush=True)
   ```
2. Add diagnostic version that checks file access without modifying
3. **Request user to execute manually** with full Python path:
   ```powershell
   & "C:\Program Files\Python313\python.exe" tools\script_name.py
   ```
4. User copies output back to confirm success
5. AI verifies with `read_file` after user reports completion

**Why manual execution on Windows**:
- `run_in_terminal` cannot capture Python output reliably
- File locks from VS Code may block writes
- Manual execution ensures visibility into errors
- User can close files in editor before execution

**For Linux/macOS**: `run_in_terminal` works reliably, can automate fully

**Folder organization**:
- Group related topics: `basics/`, `fee_reform/`, `annual_report/`
- Or flat structure if topics are heterogeneous
- English filenames recommended (AI tool compatibility)

#### Navigation Hub Requirements

**Main file becomes index** (~200-400 lines):
1. **Explanation**: Why file was split (token efficiency)
2. **Quick links**: All topic files organized by category
3. **Topic summaries**: 2-3 sentences + key metrics + link for each topic
4. **Usage guide**: How to use with AI ("For Topic X discussion, load file Y")
5. **Efficiency examples**: Token savings calculations (e.g., "400 lines instead of 1222 → 3x efficiency")

**Each topic file includes**:
- Clear scope (what this file covers)
- Links to related files (cross-references)
- Self-contained content (readable independently)

#### Measured Benefits

**Token efficiency improvement**: 3-10x depending on topic granularity

**Example** (from kannondai-community):
| Discussion Topic | Files Needed | Lines | Efficiency |
|------------------|--------------|-------|------------|
| Group 6 crisis | crisis.md + current_situation.md | 400 | 3.0x (vs 1222) |
| Fee reform | 3 fee_reform files | 420 | 2.9x |
| Annual report | 3 annual_report files | 410 | 3.0x |
| Officer selection | officer_selection_policy.md | 200 | 6.1x |

**Average improvement**: 5-10x for typical focused discussions

#### How to Request This Pattern

**Copy-paste template for AI**:

```
Context: This [document name] has grown to [X] lines covering [Y] distinct topics.
Problem: AI loads entire document when discussing one topic, wasting tokens.

Request: Split this document for AI context efficiency:
- Target: 10-15 topic files (100-300 lines each)
- Create: Topic-based files + navigation hub
- Preserve: All content with cross-references
- Optimize: Enable topic-specific file loading

Process:
1. Map structure (grep main sections)
2. Propose split (show topic breakdown)
3. Create files (await approval before mass operations)
4. Transform main file to hub
5. **Stop if struggling** (>3 failures on same operation)

Priorities:
- Token efficiency over human readability
- Self-contained topics
- Clear cross-references
```

#### Red Flags

**Don't apply this pattern if**:
- Document is reference material read start-to-finish (keep linear)
- Topics heavily interdependent (splitting causes confusion)
- Document under 500 lines (overhead not worth it)
- Chronological narrative where sequence matters critically

**Signs pattern isn't working**:
- Every discussion needs 5+ files (topics too granular)
- Constant file-switching (topics poorly separated)
- Users prefer original document (split added complexity without benefit)

#### Maintenance

**After splitting**:
- Update topic files independently
- Update navigation hub only when structure changes
- Add new topics as separate files
- Archive or merge topics that become obsolete

**Version tracking**: Add split date to navigation hub header

#### Integration with INIT/STATUS Pattern

**This pattern is orthogonal**:
- INIT/STATUS: Separates conventions from current work
- Topic splitting: Separates large documents into focused discussions

**Can combine**:
- INIT.md could be split if conventions exceed 500 lines
- STATUS.md typically stays linear (chronological narrative)
- Topic docs mentioned in STATUS can be split

**Example** (kannondai-community):
- `copilot-instructions.md`: Small, no split needed
- PROJECT_STATUS.md: Linear log, no split needed
- officer_system_discussion.md: Large topic doc → split into 13 files

---

## Benefits

### 1. Reduces Context Drift
- Conventions documented once, applied consistently
- No need to re-explain patterns each session
- AI behavior becomes more predictable and reliable

### 2. Separates Static from Dynamic
- **STATIC conventions** (INIT) only update when your working patterns change
- **DYNAMIC state** (STATUS) updates as work progresses
- Don't update INIT when status changes
- Don't update STATUS when conventions change
- Lower maintenance burden overall
- Clear mental model: "Does this change weekly? → DYNAMIC. Annually? → STATIC."

### 3. Native Platform Support
- `.github/copilot-instructions.md` is the native GitHub Copilot initialization file
- Automatically loaded — no vendor lock-in workaround needed
- Human-readable markdown, not JSON/YAML

### 4. Human-Readable
- Team members benefit from same clarity
- Onboarding becomes easier
- Documentation serves dual purpose

### 5. Scales Well
- Small projects: Minimal INIT.md, simple STATUS.md
- Large projects: Comprehensive INIT.md with detailed conventions
- Adapts to project complexity naturally

### 6. Zero Friction
- `copilot-instructions.md` is loaded automatically — no phrase to remember, no action needed
- Beginners can start without any prior knowledge of the pattern
- Particularly suited to onboarding repositories (e.g., molass-beginner)

### 7. Self-Documenting
- The file name and location are standard — any Copilot user can find it
- Human-readable onboarding document for new team members

### 7. Supports Session Resumption
- Clear "Current Task" section eliminates temporal ambiguity
- AI can resume work without "what was I doing?" questions
- Derives next task from project context automatically
- Works after hours, days, or weeks away from project

### 8. Lightweight Initialization
- Standard document (~47K tokens) is NOT loaded during initialization
- Only repository files (~1-5K tokens) are read
- Fast context loading for immediate productivity
- Version checks are minimal and non-intrusive

### 9. Supports Complex Problem Analysis
- Dedicated documents for multi-faceted issues
- Context management for long-term discussions
- Works for both programmers and non-programmers
- Structured thinking externalized for AI and human collaboration
- Natural evolution from STATUS ("working on X") to deep analysis (topic doc)

---

## Example Implementations

### Minimal Implementation (Small Personal Project)

**Two-file approach:**

**`.github/copilot-instructions.md`:** (STATIC conventions)
```markdown
# AI Initialization

## Working Conventions
- Use Python 3.11+ with type hints
- Tests in tests/ directory using pytest
- Follow PEP 8 style guide

## Repository Context
Personal utility scripts for data processing.
```

**PROJECT_STATUS.md:** (DYNAMIC state)
```markdown
# Status

## 🎯 Current Task

Working on: Adding CSV export functionality
Next: Write tests for export module
See: data_processing.py, tests/test_export.py

---

## Recent Work
- Jan 15: Added JSON parsing
- Jan 10: Initial commit
```

### Single-file Implementation (Website Management)

**Combined approach with clear separator:**

**COPILOT_SESSION_CONTEXT.md:**
```markdown
# Session Context for Site Editing

(Magic phrase here for initialization)

---

## STATIC CONVENTIONS (INIT equivalent)

1. **Simplicity First**  
   Keep text concise and readable.

2. **Natural Japanese Expression**  
   Use natural phrasing appropriate for community website.

3. **QR Code Images**  
   Self-generated, placed as `<img src="page_url.png" ...>`

4. **Site Philosophy**  
   Unofficial volunteer-driven site, free from formal constraints.

---

## DYNAMIC HISTORY (STATUS equivalent)

### 2025-05-08
- GitHub Actions workflow for calendar data update confirmed working.
- Calendar UI: Device detection for "click/tap" terminology added.
- Issue: Selected cell color not changing - investigating CSS priority.

### Next Steps
- Fix selected cell color issue.
- Final site-wide verification.
```

**Benefits of single-file approach:**
- Clear visual separation with `---` divider
- One file to maintain (simpler for small projects)
- Static conventions at top (read once, rarely update)
- Dynamic history at bottom (append frequently)
- Still follows STATIC/DYNAMIC separation principle

### Comprehensive Implementation (Research Project)

**Two-file approach for complex projects:**
- [`.github/copilot-instructions.md`](.github/copilot-instructions.md) - STATIC: Detailed conventions
- [PROJECT_STATUS.md](PROJECT_STATUS.md) - DYNAMIC: Chronological development log

**When to use each:**
- Single-file: Small projects, simple conventions, one person
- Two-file: Large projects, detailed conventions, team collaboration

---

## Content Boundaries (What Goes Where)

**Core principle**: Separate **STATIC** (conventions) from **DYNAMIC** (state)

- **STATIC = INIT** = "How to work here" (rarely changes)
- **DYNAMIC = STATUS** = "What's happening now" (changes frequently)

### `copilot-instructions.md`: STATIC Conventions

**Belongs here:**
- How to write code (style, patterns, frameworks)
- How to structure documentation (notation, terminology)
- Where to find things (directory structure, key files)
- How to work with the project (testing, building, deploying)
- Conventions that rarely change

**Examples:**
- "Use $M = PC$ for matrix factorization"
- "Test files go in tests/ directory"
- "Status symbols: ✓ complete, ⚠️ partial, 🔬 experimental"
- "Follow PEP 8 style guide"

**Does NOT belong here:**
- Current work in progress
- Recent changes or history
- Specific bugs or issues
- Temporary decisions
- What you're doing this week

### PROJECT_STATUS.md: DYNAMIC State

**Belongs here:**
- What you're working on now
- Recent accomplishments (chronological)
- Current blockers or issues
- Next planned steps
- Project evolution narrative

**Examples:**
- "Currently: Adding CSV export feature"
- "Feb 2: Discovered permutation ambiguity issue"
- "Next: Validate against benchmark datasets"
- "⚠️ Blocked on missing test data"

**Does NOT belong here:**
- How to write code (belongs in INIT)
- Documentation conventions (belongs in INIT)
- Permanent patterns (belongs in INIT)
- Project overview (belongs in README)

### README.md: Project Overview

**Belongs here:**
- What the project is (mission, purpose)
- Why it exists (motivation, problem)
- Who it's for (audience)
- How to get started (installation, basic usage)

**Does NOT belong here:**
- AI initialization instructions (belongs in INIT)
- Current status (belongs in STATUS)
- Detailed conventions (belongs in INIT)

### Decision Tree

```
Is it about HOW to work here? (STATIC)
  → `.github/copilot-instructions.md`

Is it about WHAT's happening now? (DYNAMIC)
  → PROJECT_STATUS.md

Is it about WHAT this project is? (STATIC overview)
  → README.md

Is it detailed evidence/analysis?
  → Separate doc (link from INIT or STATUS)

Is it a complex topic requiring structured discussion?
  → Dedicated topic document (see Managing Complex Topics below)
```

---

## Managing Complex Topics

**Problem**: Some issues are too complex for a single conversation and require context management across multiple sessions.

**Target audience**: Both programmers and non-programmers using AI assistants for complex problem-solving.

### When to Create Dedicated Discussion Documents

Create a separate topic-specific document when:

1. **Multi-faceted problem**: Issue has multiple dimensions (technical, social, economic, etc.)
2. **Multiple stakeholders**: Different viewpoints need to be captured and organized
3. **Long-term discussion**: Decision spans weeks/months, not a single session
4. **Accumulated context**: Background information, survey results, meeting minutes, etc.
5. **Structured thinking needed**: Problem requires systematic organization to analyze

**Examples**:
- Community governance reform proposals
- Technical architecture decisions with multiple trade-offs
- Policy changes requiring stakeholder input
- Research questions with evolving hypotheses

**Don't create dedicated doc for**:
- Simple bug fixes or feature requests (→ use issue tracker)
- Current work in progress (→ belongs in STATUS)
- General project information (→ belongs in README or INIT)

### Structure Guidelines

**Recommended sections for problem discussion documents**:

```markdown
# [Topic Name] Discussion

**Created**: [Date]
**Purpose**: [One-line goal]

## 1. Background
- Current situation
- Why this needs addressing

## 2. Key Information
- Data, survey results, meeting outcomes
- Facts and context

## 3. Stakeholder Perspectives
- Different viewpoints
- Concerns and priorities

## 4. Discussion Points
- Questions to resolve
- Trade-offs to consider
- Options to evaluate

## 5. References
- Related documents
- Source materials

## 6. Action Plan
- Next steps
- Timeline

## 7. Update History
- Track how discussion evolves
```

**Customize to your domain**: This is a template, not a rigid requirement. Adapt sections to fit your specific problem type.

### Relationship to INIT/STATUS/README

**Dedicated topic documents are separate from core files**:

- **INIT**: Documents general conventions for ALL work
  - Example: "How to structure discussion documents"
  
- **STATUS**: Tracks current work across ALL topics
  - Example: "Currently: Discussing governance reform"
  - Example: "Next: Review stakeholder feedback on proposal"
  
- **Topic doc**: Deep dive into ONE specific complex issue
  - Example: `officer_system_discussion.md` - governance reform
  - Example: `api_redesign_proposal.md` - technical architecture
  
- **README**: Explains what the project IS
  - Shouldn't contain individual topic discussions

**Link from STATUS**: When working on complex topic, reference it in Current Task:
```markdown
## 🎯 Current Task

Working on: Analyzing community governance reform options
Next: Review stakeholder survey results
See: docs/community/officer_system_discussion.md
```

### Why This Matters for Non-Programmers

**The insight**: Programmers discovered this pattern through using VS Code's AI agent mode, but the principle applies to anyone working with AI on complex topics.

**Key realization**: At this point in AI assistant evolution (2026), **context management is still a human responsibility**. AI assistants don't automatically maintain complex discussion threads across sessions.

**What this means**:
- You need to externalize your thinking into structured documents
- AI can help create and organize these documents
- But YOU must decide when/how to structure the context
- This is true whether you're a programmer or not

**The benefit**: Once you create a dedicated discussion document:
- AI can read it at session start to "catch up"
- You can resume complex discussions without re-explaining everything
- Multiple people can reference the same organized context
- Thinking becomes clearer through structured externalization

**Future evolution**: This pattern may become unnecessary as AI context management improves, but for now (2026), explicit structure helps both humans and AI tackle complex problems.

### Example: Community Governance Reform

**Scenario**: Local community association considering shift from mandatory rotation to voluntary officer system. Complex because:
- Multiple stakeholder groups (current officers, general members, non-participants)
- Economic dimensions (membership fees, opt-out pricing)
- Social dimensions (community cohesion, fairness, participation)
- Survey results split 46:39 (no clear mandate)
- Decision postponed for more discussion

**Why dedicated document?** 
- Too complex for single conversation
- Needs to capture multiple documents (proposals, meeting minutes, surveys)
- Requires structured analysis of trade-offs
- Multiple sessions needed to work through options

**Result**: Created `docs/community/officer_system_discussion.md`
- Organized background, membership comparison, survey results
- Listed discussion points for systematic exploration
- Enabled AI to help analyze specific aspects across multiple sessions
- Provided shared reference for human stakeholders

**Reference implementation**: See `kannondai-community` repository for real example.

### Best Practices (Evolving)

**This section is intentionally minimal** - pattern still being tested and refined.

1. **Start simple**: Don't over-engineer the structure initially
2. **Iterate**: Add sections as you discover what's needed
3. **Link liberally**: Connect to source documents, related discussions
4. **Update history**: Track how thinking evolves over time
5. **AI assistance**: Ask AI to help structure and organize as you go

**What we're still learning**:
- Optimal structure for different problem types
- When to split one document into multiple
- How to archive or sunset discussion documents
- Best practices for group collaboration via these documents

**Contribute experience**: If you use this pattern, consider sharing what worked/didn't work for your use case.

---

## Refinement Process

### Systematic Audit Method

Once you've implemented the pattern, periodically audit for content boundary violations:

**Step 1: Audit `copilot-instructions.md`** (should be STATIC)
- Search for dates, "current", "now", "this week", "currently"
- Check for specific bug/issue mentions
- Look for chronological narratives
- **Red flag**: Any sentence that would be false next month (= DYNAMIC content in STATIC file)

**Step 2: Audit PROJECT_STATUS.md** (should be DYNAMIC)
- Search for "how to", "always use", "convention", "pattern"
- Check for static reference material (equations, citations, directory trees)
- Look for content duplicated from INIT or README
- **Red flag**: Any section that hasn't changed in 3+ months (= STATIC content in DYNAMIC file)

**Step 3: Compare files**
- Is content duplicated between INIT and STATUS?
- Are there cross-references or just duplication?
- Which sections truly need to be in both? (usually none)

**Step 4: Apply decision tree**
- For each violation, apply "HOW/WHAT/WHAT" decision tree
- Move content to correct file
- Replace duplicates with brief cross-references
- Remove sections already covered elsewhere

**Step 5: Measure improvement**
- Line count before/after (STATUS should shrink significantly)
- Duplication percentage (should approach 0%)
- Last-modified dates (INIT changes rarely, STATUS changes frequently)

### Real Example: modeling-vs-model_free Repository

**Before refinement:**
- `copilot-instructions.md`: Contained "Current focus: Verification..." (dynamic)
- PROJECT_STATUS.md: 750 lines, ~60% static/duplicated content
  - "What This Repository Is" (duplicated README)
  - "Key Mathematical Foundations" (duplicated INIT)
  - "Repository Structure" (duplicated INIT)
  - "Technical Setup" (static conventions)
  - "Essential Context" (permanent reference material)
  - "Additional Resources" (static navigation)

**Violations identified:**
1. ❌ Dynamic state in INIT ("Current focus")
2. ❌ Mission statement in STATUS (belongs in README)
3. ❌ How-to instructions in STATUS (belongs in INIT)
4. ❌ Complete duplication of reference material
5. ❌ 6 major sections that never change in "status" document

**After refinement:**
- `copilot-instructions.md`: Removed dynamic "current focus" line
- PROJECT_STATUS.md: 270 lines, 100% dynamic content
  - Brief header with cross-references to README/INIT
  - "Latest Achievements" (chronological log) ✓
  - "Research Status" (current verification state) ✓
  - "What's Next" (current priorities) ✓
  - Footer references to detailed docs

**Metrics:**
- STATUS file: 750 → 270 lines (64% reduction)
- Duplication: ~60% → ~0%
- Content focus: Mixed → Pure chronological evolution
- Maintenance: Both files maintain separate concerns

**Result**: Clear separation makes it obvious where new content belongs and which file to update when.

### Common Violations & Fixes

**Remember**: STATIC (conventions) in INIT, DYNAMIC (current state) in STATUS

| Violation | Where Found | Correct Location | Fix | Type |
|-----------|-------------|------------------|-----|------|
| "Current focus: X" | INIT | STATUS | Move to "What's Next" | DYNAMIC in STATIC |
| "What this project is" | STATUS | README | Remove, add reference | Overview misplaced |
| Directory structure | STATUS | INIT | Remove duplication | STATIC in DYNAMIC |
| Setup commands | STATUS | INIT | Move to conventions | STATIC in DYNAMIC |
| Equations/notation | STATUS | INIT or separate doc | Consolidate reference | STATIC in DYNAMIC |
| "How to run tests" | STATUS | INIT | Move to conventions | STATIC in DYNAMIC |
| Dates/chronology | INIT | STATUS | Move to timeline | DYNAMIC in STATIC |
| Installation steps | STATUS | README or INIT | Standardize location | Context-dependent |

### Audit Frequency

**During initial refinement**: Weekly
- Pattern still stabilizing
- Learning what belongs where
- Discovering edge cases

**After stabilization**: Monthly or when adding major content
- Check for drift
- Validate boundaries still make sense
- Update if project structure changes significantly

**Triggers for immediate audit**:
- INIT file changed 3+ times in one week (something's wrong)
- STATUS file unchanged for 1+ month (not tracking status)
- Finding yourself confused about where to add content
- Duplication appearing between files

---

## Multi-Repository Workflow

### Using Standard Across Multiple Repositories

When implementing this pattern across multiple repositories, you need systematic tracking and feedback loops.

### Tracking Refinements

**Simple log format** (keep in your standard reference repo):

```markdown
# Standard Refinement Log

## Repo A (modeling-vs-model_free)
**Date**: 2026-02-02
**Issue**: STATUS contained 60% static/duplicated content
**Fix**: Removed 6 static sections, added cross-references
**Metrics**: 750 → 270 lines (64% reduction)
**Standard update?**: No - standard already clear, implementation violated it
**Applied elsewhere?**: N/A (first refinement)

## Repo B (my-other-project)
**Date**: 2026-02-10
**Issue**: INIT needed "Deployment Conventions" section (not in standard template)
**Fix**: Added deployment section to INIT
**Metrics**: Works well, no duplication
**Standard update?**: Maybe - if 3+ repos need it
**Applied elsewhere?**: Waiting to see if pattern repeats

## Repo C (data-analysis)
**Date**: 2026-02-15
**Issue**: Same "Current focus" in INIT problem as Repo A
**Fix**: Applied Repo A's fix (move to STATUS)
**Metrics**: Clean separation maintained
**Standard update?**: No - but adds to "Common Violations" evidence
**Applied elsewhere?**: Check Repo D next
```

### Propagating Improvements

**When you find a violation in one repo**:

1. **Diagnose**: Is it implementation error or standard ambiguity?
   - Implementation error: You violated clear boundary
   - Standard ambiguity: Boundary wasn't clear enough

2. **Fix locally**: Apply refinement to current repo

3. **Check other repos**: Do they have same violation?
   - If yes → systematic problem (fix all, improve standard)
   - If no → repo-specific drift (just fix this one)

4. **Update standard if needed**:
   - Add to "Common Violations" table
   - Clarify ambiguous boundary in "Content Boundaries"
   - Add to "Red Flags" if it's subtle

5. **Roll out to other repos**:
   - Apply same fix to repos with same violation
   - Document in refinement log
   - Update version comments if major change

**Flow diagram**:
```
Find violation in Repo A
  ↓
Fix in Repo A
  ↓
Check: Same issue in Repos B, C, D?
  ↓ Yes (pattern)              ↓ No (isolated)
Update standard                 Just fix Repo A
  ↓                                ↓
Apply to all repos              Done
  ↓
Document pattern
```

### Consistency Checking

**Periodic cross-repo audit** (monthly or quarterly):

1. **List all repos using pattern**:
   ```
   Repo A: v0.1 (modeling-vs-model_free)
   Repo B: v0.1 (my-other-project)
   Repo C: v0.1 (data-analysis)
   Repo D: Not yet implemented
   ```

2. **Check customizations**:
   - What sections did each repo add to INIT?
   - Are additions domain-specific (legitimate) or general (should be in standard)?
   - Are boundaries still followed?

3. **Compare violations**:
   - Same violation in 2+ repos? → Pattern (update standard)
   - Different violations? → Repo-specific issues
   - No violations? → Standard working well

4. **Identify universal patterns**:
   - If 3+ repos added same section → consider adding to standard template
   - If 3+ repos hit same violation → update "Common Violations"
   - If 2+ repos found same boundary unclear → clarify in standard

### Contribution Pathway

**From repositories to standard**:

```
Use in Repo A ──→ Discover issue ──→ Fix in Repo A
                       ↓
Use in Repo B ──→ Same issue ──→ Fix in Repo B
                       ↓
Use in Repo C ──→ Same issue ──→ Pattern confirmed!
                       ↓
               Update standard document
                       ↓
               Apply to all repos
                       ↓
               Update version (v0.1 → v0.2)
```

**What deserves standard update**:
- ✅ Pattern repeated in 3+ repositories
- ✅ Ambiguity that caused violations in 2+ repos
- ✅ Missing guidance that everyone needs to figure out
- ✅ Common violation with clear fix
- ❌ Repo-specific customization (even if useful)
- ❌ One-time mistake
- ❌ Tool-specific workaround

**Example contributions**:
- "All 4 repos needed 'Testing Conventions' in INIT → add to standard template"
- "Repos A & C confused about where installation steps go → clarify in decision tree"
- "Found 'current status' in INIT in 3 repos → add to 'Red Flags' with clear example"

### Simple Tracking Template

**Create**: `STANDARD_TRACKING.md` (in your personal notes or reference repo)

```markdown
# AI Context Standard - Multi-Repo Tracking

## Repositories Using Standard

| Repository | Version | Implemented | Last Audit | Notes |
|------------|---------|-------------|------------|-------|
| modeling-vs-model_free | v0.1 | 2026-02-02 | 2026-02-02 | Reference implementation, consider v0.2 migration |
| my-other-project | v0.1 | 2026-02-10 | - | Added deployment section |
| data-analysis | v0.1 | 2026-02-15 | - | Clean so far |
| legacy-code | - | Not yet | - | Planned for Q2 |

**Version tracking benefits:**
- Quickly identify which repos need migration
- Prioritize updates based on adoption date
- Track which features were tested where

## Patterns Observed

### Pattern 1: "Current focus" in INIT
- **Found in**: Repo A, Repo C
- **Fix**: Move to STATUS "What's Next"
- **Standard update**: Already documented in "Common Violations"
- **Status**: Closed - pattern known

### Pattern 2: Deployment conventions needed
- **Found in**: Repo B
- **Fix**: Added "Deployment Conventions" section to INIT
- **Standard update**: Waiting - need 2 more repos to confirm
- **Status**: Watching for pattern

### Pattern 3: TBD
...

## Pending Standard Updates

1. Add "Deployment Conventions" to standard template?
   - Evidence: Repo B needed it
   - Required: Need 2 more repos to confirm
   - Status: 1/3 votes

2. Clarify where "setup instructions" belong
   - Evidence: Confusion in Repo A, Repo C
   - Required: Clear ambiguity
   - Status: Ready to add

## Next Actions

- [ ] Audit Repo B and C (overdue)
- [ ] Apply Pattern 1 fix to all repos
- [ ] Update standard with "setup instructions" clarification
- [ ] Implement pattern in Repo D
```

### Review Cadence

**Weekly** (during active refinement):
- Quick check: Any new violations discovered?
- Update tracking log
- Apply urgent fixes across repos

**Monthly**:
- Full cross-repo audit
- Compare patterns across all repos
- Propose standard updates
- Roll out improvements

**Quarterly**:
- Deep review of standard effectiveness
- Decide on version updates (v0.1 → v0.2)
- Clean up resolved patterns
- Plan adoption for new repos

---

### Multi-Root Workspace Support

When a **VS Code multi-root workspace** has multiple repositories open simultaneously, additional conventions help AI assistants navigate the workspace effectively.

**Context**: VS Code supports opening multiple repository folders in a single workspace window. In this configuration, GitHub Copilot loads `copilot-instructions.md` from **all** open repositories simultaneously. This is an advantage — conventions from all repos are available at once.

#### Convention 1: Primary Repository Declaration

Designate one repository as the **primary workspace repository** — the one whose `copilot-instructions.md` declares the workspace ecosystem table (Convention 2).

**Where to declare**: At the top of `.github/copilot-instructions.md`:

```markdown
<!-- Primary workspace repository for multi-root workspace -->
```

**Why needed**: In multi-root workspaces, the primary repo's instructions should declare the ecosystem overview so AI can orient to the full workspace structure at a glance.

#### Convention 2: Repository Ecosystem Table

**In the primary repository's `copilot-instructions.md`**, add an ecosystem table documenting all active repositories:

```markdown
## Repository Ecosystem (Multi-Root Workspace)

| Repository | Role | AI Context File | Status |
|------------|------|-----------------|--------|
| primary-repo | Primary workspace repo | copilot-instructions.md | Active |
| lib-repo | Library / core code | copilot-instructions.md | Active |
| docs-repo | Documentation | copilot-instructions.md | Active |
| legacy-repo | Legacy codebase | copilot-instructions.md | Reference |
```

**Purpose**: AI assistants can orient to the full ecosystem after reading primary INIT, without needing to survey all repos on every session.

**Maintenance**: Update when repos are added, removed, or change role.

#### Convention 3: First-Visit Protocol

When AI visits a repository for the first time (no INIT file, or INIT has no version comment):

1. **Survey**: Check for `.github/copilot-instructions.md` and `PROJECT_STATUS.md`
2. **Create**: If absent, offer to create them with a brief survey of the repo
3. **Link**: Reference the new repo in the primary repository's ecosystem table
4. **Update**: Add an entry to the AI-readiness trail (Convention 4)

**Scope**: "First visit" = no INIT file present, or INIT file has no `<!-- AI Context Standard v0.X -->` comment.

#### Convention 4: AI-Readiness Trail

**Add to primary `copilot-instructions.md`** a dated tracking table:

```markdown
## AI-Readiness Trail

| Date | Repository | Action | Standard Version |
|------|------------|--------|-----------------|
| 2026-02-19 | modeling-vs-model_free | INIT/STATUS created, ecosystem table added | v0.5 |
| 2026-02-19 | molass-library | INIT/STATUS surveyed, links verified | v0.5 |
```

**Purpose**: Track when each repository was last surveyed and which standard version was applied.

**Benefits**:
- Quick identification of repos needing first-visit protocol
- Historical record of AI context evolution across the workspace
- Enables quarterly maintenance review without re-examining all repos

#### Integration Model

**Workspace layer** (multi-root) overlays the **repository layer** (per-repo INIT/STATUS):

```
Workspace (VS Code multi-root)
├── Primary Repo (copilot-instructions.md with ecosystem table + AI-readiness trail)
│   ├── .github/copilot-instructions.md  ← Auto-loaded first (primary)
│   └── PROJECT_STATUS.md
├── Repo B
│   ├── .github/copilot-instructions.md  ← Auto-loaded (per-repo conventions)
│   └── PROJECT_STATUS.md
└── Repo C
    ├── .github/copilot-instructions.md  ← Auto-loaded (per-repo conventions)
    └── PROJECT_STATUS.md
```

**No new file types**: Multi-root support uses existing `copilot-instructions.md`/STATUS files with additional sections in the primary repository — no new file types or protocols required.

---

## Checking for Standard Updates

**Current process**: Manual (no automated notification system yet)

**How to check if your version is outdated**:

1. **Find your adopted version**:
   - Check version comment in your INIT file: `<!-- AI Context Standard v0.X -->`
   - If no version comment, likely using pre-v0.2 or unofficial variant

2. **Check canonical source**:
   - Visit the repository containing this standard document
   - Read "Version History" section for latest version number
   - Compare with your adopted version

3. **Review changes**:
   - Read "Version Migration Guide" for your version → latest
   - Assess: Breaking changes? Recommended updates? New features?

4. **Decide on migration**:
   - Required: Only if breaking changes affect you
   - Recommended: If improvements benefit your workflow
   - Optional: If curious about new features

**Notification workarounds** (until automated system exists):
- **Star/Watch** the repository containing this standard
- **Set calendar reminder** to check quarterly
- **Note in STATUS**: "Check AI Context Standard updates" in maintenance tasks
- **Subscribe** to repository releases (if releases are tagged)

**Future plans**:
- Considering GitHub Action for update notifications
- Exploring RSS feed for version announcements
- Investigating version-check CLI tool

**Update frequency expectations**:
- **Major versions** (breaking changes): Infrequent, well-announced
- **Minor versions** (new features, clarifications): Quarterly to semi-annually
- **Patches** (typo fixes, small improvements): As needed

**Philosophy**: Stability over novelty. Updates will be conservative to avoid migration fatigue.

---

## Adoption & Feedback

### How to Adopt

1. Copy this pattern to your repository
2. Create `.github/copilot-instructions.md` with your conventions
3. Create or reorganize `PROJECT_STATUS.md`
4. Test with GitHub Copilot
5. Iterate based on what works

### Testing & Refinement

This is v0.1 - still being validated through actual use.

**As you use this across repositories, track:**

1. **Does copilot-instructions.md load automatically?**
   - Does Copilot apply conventions at session start without being asked?
   - Is the file in the `.github/` directory?
   - Is Agent mode enabled?

2. **Content drift**
   - Are conventions creeping into STATUS?
   - Is status creeping into INIT?
   - Are you following the boundaries?

3. **Maintenance burden**
   - How often do you update INIT? (should be rare)
   - How often do you update STATUS? (should be frequent)
   - Is one harder to maintain than expected?

4. **AI behavior**
   - Does AI follow conventions consistently?
   - Does it remember context across conversation?
   - Does it need reminders?

5. **Cross-repository patterns**
   - What sections are identical across repos?
   - What's always repository-specific?
   - What customizations are you repeating?

**Simple tracking approach:**
- **Note version comment** when you set up a repo (e.g., `<!-- AI Context Standard v0.2 -->`)
- Document "what I changed from standard" in INIT
- Keep brief notes on what works/doesn't work
- Compare across repos after ~2 weeks of use
- **Check for standard updates** quarterly (see "Checking for Standard Updates" section)

---

## Red Flags

**Signs the pattern is breaking down:**

### `copilot-instructions.md` Issues (should be STATIC)
- ⚠️ You're updating it every session (too dynamic → content belongs in STATUS)
- ⚠️ It's over 500 lines (too detailed, should link out)
- ⚠️ Contains dates/history (DYNAMIC content → belongs in STATUS)
- ⚠️ Describes current work (DYNAMIC content → belongs in STATUS)
- ⚠️ AI doesn't follow the conventions you documented

### PROJECT_STATUS.md Issues (should be DYNAMIC)
- ⚠️ Hasn't been updated in weeks (not tracking DYNAMIC state)
- ⚠️ Contains "how to write code" (STATIC content → belongs in INIT)
- ⚠️ Contains permanent patterns (STATIC content → belongs in INIT)
- ⚠️ Just duplicates what's already in git log

### Magic Phrase Issues
- ⚠️ This standard no longer uses a magic phrase — initialization is automatic
- If a team member or AI assistant is asking for a phrase, `copilot-instructions.md` may be absent or misconfigured
- Check: `.github/copilot-instructions.md` exists and is non-empty

### Overall Pattern Issues
- ⚠️ Takes >30 min to set up new repo
- ⚠️ Spending more time on docs than code
- ⚠️ Files duplicate each other significantly
- ⚠️ Team members confused about what goes where
- ⚠️ AI behavior inconsistent despite documentation

**If you see these:**
- Something needs simplifying
- Content boundaries need clarifying
- Or pattern doesn't fit your workflow (that's OK!)

---

## Why This Might Become Standard

**Historical precedent:**
- `.gitignore` emerged organically before becoming ubiquitous
- `README.md` became standard through usage, not decree
- `LICENSE` files evolved into common practice

**Similar trajectory:**
- Discovered through real pain points (context drift)
- Solves actual problems (session persistence)
- Simple enough to adopt (two files, auto-loaded)
- Benefits visible immediately (better AI behavior)
- Native platform support (no workaround needed)

**Current landscape:**
- AI-assisted development is 2-3 years old
- No governing body for AI coding standards
- Community patterns still emerging
- Early adopters can influence direction

**This proposal:**
- Documents existing practice (not inventing from scratch)
- Leverages native Copilot support (`.github/copilot-instructions.md`)
- Remains flexible (customize to your needs)
- Evolves with feedback (version 0.1 → 0.7)

---

## Alternatives Considered

### Why not just README.md?
- README is for project overview, not AI initialization
- Mixing concerns makes both purposes harder
- README changes frequently for different reasons
- Separation keeps each document focused

### Why not code comments?
- Fragmented across multiple files
- Hard to maintain consistency
- AI must parse code to find conventions
- Doesn't help with project-level context

### Why not AI tool features (@workspace, etc.)?
- Vendor-specific (lock-in)
- Not all tools have equivalent features
- Features may change or disappear
- File-based approach is more permanent

---

## Open Questions (Testing Phase)

**Still figuring out:**

1. **Scope of copilot-instructions.md**: 
   - How much detail before it's too much?
   - When to link out vs document inline?
   - What's the right balance?

3. **Scope**: 
   - How much detail in INIT before it's too much?
   - When to link out vs document inline?
   - What's the right balance?

4. **STATUS format**: 
   - Chronological log? Structured sections? Both?
   - How far back to keep history?
   - When to archive old status?

5. **Session resumption** (Tested February 4, 2026):
   - ✅ "Current Task" + derivation rule works well
   - ✅ Natural progression from completed work to next task
   - ✅ Eliminates "what was I doing?" questions
   - Pattern documented in Step 5, ready for broader testing

6. **Multi-repo patterns**:
   - Should there be a shared template across repos?
   - How to handle org-wide conventions?
   - When is duplication OK vs needing consolidation?

7. **Tool compatibility** (✅ Resolved v0.7):
   - ✅ GitHub Copilot: native auto-load via `.github/copilot-instructions.md`
   - Other AI tools (Claude, ChatGPT, Cursor, Windsurf): if needed, those tools can be asked to read `copilot-instructions.md` explicitly
   - This standard is Copilot-native; other tools are out-of-scope

8. **Context load scalability** (Added v0.2):
   - Will initialization become too slow as standards/repos grow?
   - Current approach: Standard doc not loaded during initialization
   - Monitor: Actual context consumption in real-world use
   - May need optimization or lazy loading in future

9. **Complex topic management** (Added v0.2):
   - When is a separate discussion document needed vs using STATUS?
   - Optimal structure for different problem types (technical vs social vs research)?
   - How to integrate topic docs with INIT/STATUS workflow?
   - When to archive completed discussions?
   - Does this pattern work for non-programmers without technical onboarding?

10. **Large document AI context optimization** (✅ Partially addressed v0.3):
   - ✅ When to split? → Step 7: Documents 500+ lines with multiple independent topics
   - ✅ How to split? → Topic files (100-300 lines) + navigation hub (200-400 lines)
   - ✅ Measurable benefit? → 3-10x token efficiency (validated: 1222→400 lines for focused discussion)
   - Still testing: Optimal topic granularity, red flags for when NOT to split, long-term maintenance
   - AI behavior pattern: Stop after 3 failures, propose manual operation for large edits (100+ lines)

11. **Multi-root workspace patterns** (Added v0.5 — still testing):
   - Does Convention 1 (primary repo declaration) work cleanly with different AI tool behaviors?
   - What is the right granularity for the ecosystem table? (per-repo vs. per-folder)
   - Should the AI-readiness trail live in `copilot-instructions.md` or a separate file?
   - How does session resumption work when focus shifts between repos mid-session?
   - Are Conventions 1-4 sufficient, or are additional workspace-level conventions needed?

**Test these through use, then refine the standard based on answers.**

---

## Version History

**v0.8.2** (March 26, 2026)
- **Added "Notebook Cell Output Reading" convention** to "Working Conventions for AI Assistants"
  - Problem: `read_notebook_cell_output` tool silently fails with "output too large" for stdout-heavy cells
  - Root cause: The failure is predictable and preventable — mime types are visible in `copilot_getNotebookSummary` output BEFORE the tool is called
  - Rule: If a cell has `application/vnd.code.notebook.stdout` in its mime types, skip `read_notebook_cell_output` entirely
  - Solution: [`ai-context-tools`](https://github.com/freesemt/ai-context-tools) package (`pip install ai-context-tools`) — `python -m aic_tools.notebook <nb> <cell>` reads the `.ipynb` JSON directly, bypassing the tool limit
  - Principle: Build the tool rather than work around the missing capability
  - Applies to: Any notebook-heavy repository; install once, use everywhere

**v0.8.1** (March 26, 2026)
- **Corrected `init.prompt.md` frontmatter key**: `mode: ask` → `agent: ask`. VS Code deprecated `mode` in favour of `agent` for prompt file frontmatter; using `mode` produces a deprecation warning. All `init.prompt.md` files in compliant repos should use `agent: ask`.
- **Known gap: `alwaysApply: true` is not yet reliably triggered** in all sessions. In practice, `init.prompt.md` does not always auto-execute on chat start even when `alwaysApply: true` is set. Until VS Code guarantees consistent trigger behaviour, **`/init` remains the recommended action at the start of any session where current project status matters**. This is a platform limitation, not a standard flaw — the intent of `alwaysApply: true` remains correct and should be kept in place for when it does fire.

**v0.7** (March 24, 2026)
- **Complete migration to native auto-load**: `.github/copilot-instructions.md` is now the sole initialization mechanism
- **Magic phrase retired**: No longer part of this standard. The phrase was invented because `.copilotrc` didn't exist. Now that GitHub Copilot provides a native equivalent, the workaround is unnecessary.
- **Removed Track B entirely**: Tool-agnostic explicit load is out of scope for this standard
- **Revised Problem Statement**: Initialization is solved; remaining gaps are STATIC/DYNAMIC separation and session resumption only
- **Renamed Core Concept section**: "copilot-instructions.md + STATUS.md Pattern" (was "Two-Track Initialization + STATUS Pattern")
- **Revised Step 1**: Create `.github/copilot-instructions.md` (was COPILOT-INIT.md)
- **Revised Step 3**: README reference updated (no magic phrase)
- **Revised Step 4**: Version declaration placement updated (single file only)
- **Revised Step 5**: Session protocol no longer includes magic phrase
- **Revised Step 6**: Compliance check targets `copilot-instructions.md`
- **Revised AI Behavior section**: Single flow (auto-load); Track B removed
- **Revised Benefits**: §3 "Native Platform Support", §6/§7 simplified
- **Removed** "Why not .github/copilot-instructions.md as sole solution?" from Alternatives (it IS the solution now)
- **Revised Red Flags**: Magic phrase section now indicates absence/misconfiguration of `copilot-instructions.md`
- **Revised Multi-root workspace**: Convention 1 updated (no magic phrase ambiguity; all repos' files auto-loaded)
- **Revised Testing**: Q1 replaced with scope question; Q7 resolved
- **Key insight**: The magic phrase was a workaround for a missing platform feature. The feature now exists. Remove the workaround.

**v0.6** (March 24, 2026)
- **Breaking change in framing**: Magic phrase is no longer the primary initialization mechanism; replaced by two-track model
- **Added Track A**: `.github/copilot-instructions.md` — native Copilot auto-load (zero user effort)
- `.github/copilot-instructions.md` is now the sole initialization mechanism; Track B (`COPILOT-INIT.md` + magic phrase) removed
- **Revised Problem Statement**: Acknowledges that GitHub Copilot now has a native `.copilotrc` equivalent; gaps remaining are tool-agnostic support and STATIC/DYNAMIC separation
- **Revised Core Concept section**: Two-track model with comparison table and guidance on choosing
- **Revised AI Behavior on Initialization**: Separate flows for Track A (auto) and Track B (explicit)
- **Revised Benefits §3**: "Platform-Native or Tool-Agnostic" replaces "Tool-Agnostic"
- **Added Benefits §6**: Zero friction for Copilot users (Track A)
- **Revised Benefits §7**: "Self-Documenting Entry Point" narrowed to Track B
- **Revised Alternatives Considered**: Replaced "Why not .copilotrc?" with "Why not copilot-instructions.md as sole solution?"
- **Revised Red Flags**: Magic phrase red flags scoped to Track B only
- **Updated Open Questions**: Q1 and Q7 updated to reflect two-track model
- **Key insight**: The magic phrase was invented because `.copilotrc` didn't exist. Now that Copilot has a native equivalent, the phrase becomes Track B — a fallback for tool-agnostic use, not the primary mechanism.

**v0.5** (February 19, 2026)
- **Added "Multi-Root Workspace Support" subsection** to Multi-Repository Workflow
  - Convention 1: Primary repository declaration (which COPILOT-INIT.md the magic phrase reads)
  - Convention 2: Repository ecosystem table in primary COPILOT-INIT.md
  - Convention 3: First-visit protocol (4 steps: survey, create, link, update trail)
  - Convention 4: AI-readiness trail (date-stamped tracking table in primary repo)
  - Integration model: workspace layer overlays repository layer, no new file types
- **Added Open Question #11**: Multi-root workspace patterns (still testing)
- Addresses gap identified when using standard in VS Code multi-root workspaces with 7+ repos

**v0.4** (February 11, 2026)
- **Added "How to Use This Standard" section** at the beginning
  - Declares standard as axiomatic, not prescriptive
  - Clarifies human-AI collaboration model
  - Distinguishes "apparent high performance" (template application) from "true high performance" (creative adaptation)
  - Explains why the standard stays minimal
  - ~300 words, foundational/axiomatic
- **Enhanced Step 1 and Step 2** with collaborative approach guidance
  - Step 1: "Tell AI about your project. Let it propose structure. You provide judgment."
  - Step 2: "This tracks what's happening now and evolution over time. Structure emerges naturally."
  - Clarifies templates are guidance, not rigid forms
- **Philosophy**: Standard enables creative partnership, not prescribed implementation
- **Key insight**: AI's capability can produce "apparent" vs "true" high performance — axiomatic approach ensures the latter

**v0.3** (February 7, 2026 - Later update)
- **Added "Working Conventions for AI Assistants" section** with failure recovery protocol
  - New independent section (not limited to Step 6 compliance verification)
  - Rule: Stop after 3 failures on same operation, explain situation to user
  - Applies to: Large-scale editing, repeated tool failures, non-essential tasks, any friction
  - Prevents silent struggle through 15+ failed attempts
  - Documented as working convention for cross-session consistency
  - Real example: 1370-line deletion attempts (should stop at 3-4 failures, propose manual operation)
- **Added Step 7: Large Document Management** for AI context efficiency
  - Problem: 500+ line documents waste tokens (60-90% irrelevant content loaded)
  - Solution: Topic-based file splitting (100-300 lines) + navigation hub (200-400 lines)
  - When to apply: Multi-topic documents where AI focuses on specific sections
  - Implementation: User delegates to AI with structured request template
  - Benefits: 3-10x token efficiency improvement (measured from real case)
  - Real case: 1222-line document → 13 topic files, 5-10x efficiency gain
  - Integration: Orthogonal to INIT/STATUS pattern (applicable to topic documents)

**v0.2** (February 7, 2026)
- **Enhanced STATIC/DYNAMIC terminology** throughout document
- Added clear mapping: STATIC = INIT (conventions), DYNAMIC = STATUS (state)
- Added single-file implementation variant for smaller projects
- Updated all sections to consistently emphasize static/dynamic separation
- Improved decision tree and audit guidelines with STATIC/DYNAMIC labels
- Added "Key principle" guides to Core Concept section
- **Enhanced version tracking** (Step 4: Optional→Recommended, adoption date)
- **Added Version Migration Guide** for update management
- **Added version declaration check** to compliance verification
- **Added standard update checking guidance**
- **Added AI Assistant Behavior on Initialization** section
  - Clarified: Standard document not for AI initialization (only repo files)
  - Simple version mismatch notification (one line, non-intrusive)
  - Expressed concern about context load scalability
  - Philosophy: Keep initialization fast and simple
- **Added "Managing Complex Topics" section**
  - When to create dedicated discussion documents for complex problems
  - Structure guidelines for topic-specific documents
  - Relationship to INIT/STATUS/README
  - Why context management matters for non-programmers
  - Real example from community governance reform
  - Acknowledgment that this is evolving pattern needing refinement

**v0.1** (February 2-4, 2026)
- Initial proposal
- Documented INIT.md + STATUS.md pattern
- Established magic phrase convention
- Drafted implementation guide
- Added content boundaries (what goes where)
- Added red flags for troubleshooting
- Added open questions for testing phase
- Added systematic refinement process with real example
- Added multi-repository workflow and tracking methodology
- **Added session resumption pattern (Step 5)** - February 4, 2026
- **Added "Why this step?" explanations for Steps 1-5** - February 4, 2026
- **Added compliance verification prompt (Step 6)** - February 4, 2026

---

## Version Migration Guide

### Migrating from v0.6 to v0.7

**Summary**: v0.7 retires the magic phrase and Track B entirely. Static conventions now live exclusively in `.github/copilot-instructions.md`. `COPILOT-INIT.md` files should be migrated or removed.

**Is migration required?** Yes — `COPILOT-INIT.md` is no longer part of this standard.

**Migration steps per repository:**
1. **Update version comment**: `<!-- AI Context Standard v0.7 - Adopted: YYYY-MM-DD -->`
2. **Create `.github/copilot-instructions.md`**: Copy contents of `COPILOT-INIT.md` into it
3. **Add `PROJECT_STATUS.md` auto-read directive** (if repo has `PROJECT_STATUS.md`): Insert near the top of `copilot-instructions.md`:
   ```markdown
   > **On every session start**: Read [`PROJECT_STATUS.md`](PROJECT_STATUS.md) to get the current task and recent context before responding.
   ```
4. **Remove `COPILOT-INIT.md`**: Or replace with a one-line pointer: `This file has been migrated to [.github/copilot-instructions.md](.github/copilot-instructions.md) (AI Context Standard v0.7).`
5. **Update README**: Remove magic phrase reference; optionally add "context auto-loaded from `.github/copilot-instructions.md`"
6. **Test**: Open in VS Code Agent mode and verify conventions are active without saying anything

**Effort**:
- Simple repos: 10-15 minutes (copy + rename + README update)
- Complex repos with many references: 20-30 minutes

**AI-assisted automatic migration:**

An AI assistant encountering a repository that still has `COPILOT-INIT.md` but no `.github/copilot-instructions.md` **should automatically perform the migration without being asked**, following the steps above. This is the intended behavior once this standard file (`AI_CONTEXT_STANDARD.md`) has been updated to v0.7 in the workspace.

**Trigger condition**: `COPILOT-INIT.md` exists AND `.github/copilot-instructions.md` does not exist.

**Automatic migration checklist:**
- [ ] Create `.github/copilot-instructions.md` — content from `COPILOT-INIT.md`, with:
  - Version comment updated to `<!-- AI Context Standard v0.7 - Adopted: YYYY-MM-DD -->`
  - Magic phrase line removed from header and body
  - `PROJECT_STATUS.md` auto-read directive added near top (if repo has `PROJECT_STATUS.md`)
  - Structure maps updated to show `.github/copilot-instructions.md` instead of `COPILOT-INIT.md`
  - Updates trail: add entry `YYYY-MM-DD | Migrated to .github/copilot-instructions.md (AI Context Standard v0.7)`
- [ ] Replace `COPILOT-INIT.md` with a one-line pointer (do not delete — preserves git history context)
- [ ] Update `README.md` if it references the magic phrase or `COPILOT-INIT.md`
- [ ] Commit with message: `standard: v0.7 — migrate to .github/copilot-instructions.md; retire COPILOT-INIT.md`
- [ ] Push

**Breaking changes**: `COPILOT-INIT.md` is no longer the standard file; magic phrase is retired

**Backward compatibility**: Copilot will still work with `COPILOT-INIT.md` if you keep it and ask Copilot to read it explicitly — but it is no longer auto-loaded

**New behavior**:
- No phrase needed at session start
- Context available immediately in every Agent mode session
- `PROJECT_STATUS.md` read automatically at session start (via directive in `copilot-instructions.md`)

### Migrating from v0.5 to v0.6

**Summary**: v0.6 introduces the two-track initialization model. The magic phrase still works (Track B) — nothing breaks. The key change is that GitHub Copilot users can now use Track A (auto-load) instead.

**Is migration required?** No — existing `COPILOT-INIT.md` + magic phrase setups remain fully valid as Track B.

**Recommended improvements**:
1. **Update version comment** in INIT file header: `<!-- AI Context Standard v0.6 - Adopted: YYYY-MM-DD -->`
2. **If using GitHub Copilot**: Create `.github/copilot-instructions.md` with your static conventions (Track A)
   - Option A: Duplicate COPILOT-INIT.md content into it
   - Option B: Have it point to COPILOT-INIT.md with instructions to read it
   - Option C: Make it the canonical source and retire COPILOT-INIT.md (if Copilot-only team)
3. **README update**: Remove "magic phrase" mention if switching to Track A; replace with "open in VS Code Agent mode"

**Effort**:
- Copilot-only team switching to Track A: 15-30 minutes
- Multi-tool team staying on Track B: 2-5 minutes (version comment only)
- Hybrid (both tracks): 30-60 minutes

**Breaking changes**: None — all v0.5 setups remain compliant as Track B

**Deprecations**: Magic phrase is not deprecated; it is now Track B (explicit load, tool-agnostic)

**New features**:
- Track A: zero-friction auto-load for GitHub Copilot users
- Two-track model for mixed-tool teams

### Migrating from v0.4 to v0.5

**Summary**: v0.5 adds multi-root workspace support conventions. No breaking changes.

**Is migration required?** No - v0.5 is fully backward compatible with v0.4

**Required changes**: None (v0.4 implementations remain compliant)

**Recommended improvements**:
1. **Update version comment** in INIT file header: `<!-- AI Context Standard v0.5 - Adopted: YYYY-MM-DD -->`
2. **If using multi-root workspace**: Add ecosystem table + AI-readiness trail to primary `.github/copilot-instructions.md`
3. **Mark primary repository**: Add `<!-- Primary workspace repository for multi-root workspace -->` comment

**Effort**:
- Single-repo projects: 2-5 minutes (version comment only)
- Multi-root workspaces: 15-30 minutes (ecosystem table + AI-readiness trail)

**Breaking changes**: None

**Deprecations**: None

**New features**:
- Multi-root workspace conventions (Conventions 1-4)
- AI-readiness trail for workspace-level tracking
- First-visit protocol for new repositories

### Migrating from v0.3 to v0.4

**Summary**: v0.4 adds axiomatic guidance on human-AI collaboration. No breaking changes.

**Is migration required?** No - v0.4 is fully backward compatible with v0.3

**Required changes**: None (v0.3 implementations remain compliant)

**Recommended improvements**:
1. **Update version comment** in INIT file header: `<!-- AI Context Standard v0.4 - Adopted: YYYY-MM-DD -->`
2. **Read "How to Use This Standard"** section to understand collaboration model
3. **When helping others adopt**: Emphasize collaborative approach over template copying

**Effort**: 2-5 minutes per repository

**Key insight from v0.4**:
- Standard is axiomatic (minimal essential principles) not prescriptive (detailed templates)
- AI's high capability can produce "apparent high performance" (perfect template execution) or "true high performance" (creative adaptation through understanding)
- Axiomatic approach ensures collaboration leads to true high performance
- Human provides direction, AI provides structure, together discover optimal solution

**Breaking changes**: None

**Deprecations**: None

**New features**:
- "How to Use This Standard" section (axiomatic principles)
- Collaborative approach guidance in Steps 1-2
- Explicit philosophy on simplicity

### Migrating from v0.2 to v0.3

**Summary**: v0.3 adds failure recovery protocol and large document management. No breaking changes.

**Is migration required?** No - v0.3 is fully backward compatible with v0.2

**Required changes**: None (v0.2 implementations remain compliant)

**Recommended improvements**:
1. **Update version comment** in INIT file header: `<!-- AI Context Standard v0.3 - Adopted: YYYY-MM-DD -->`
2. **Review failure recovery protocol**: AI assistants working in this repo should follow 3-failure stop rule
3. **Consider Step 7**: If any documents exceed 500 lines with multiple topics, apply topic-splitting

**Effort**: 5-10 minutes per repository

**Breaking changes**: None

**Deprecations**: None

**New features**:
- Working Conventions for AI Assistants (failure recovery protocol)
- Step 7: Large Document Management for AI context efficiency
- Topic-based file splitting pattern
- Windows PowerShell large deletion workaround

### Migrating from v0.1 to v0.2

**Summary**: v0.2 clarifies STATIC/DYNAMIC terminology throughout. No breaking changes.

**Is migration required?** No - v0.2 is fully backward compatible with v0.1

**Required changes**: None (v0.1 implementations remain compliant)

**Recommended improvements**:
1. **Add version comment** to INIT file header: `<!-- AI Context Standard v0.2 - Adopted: YYYY-MM-DD -->`
2. **Add STATIC/DYNAMIC labels** to section headers for clarity:
   - INIT sections: "STATIC Conventions" or similar
   - STATUS sections: "DYNAMIC State" or "DYNAMIC History"
3. **Review enhanced guidance**: Check updated Content Boundaries and Decision Tree
4. **Consider single-file variant** if appropriate for your project size

**Effort**: 5-10 minutes per repository

**Compatibility matrix**:
- v0.1 INIT files → Compatible with v0.2 (no changes needed)
- v0.1 STATUS files → Compatible with v0.2 (no changes needed)
- Single-file approach → Officially recognized in v0.2

**Breaking changes**: None

**Deprecations**: None

**New features**:
- Single-file variant officially documented
- Enhanced STATIC/DYNAMIC terminology
- Version tracking recommendations
- Migration guidance (this section)

### Future Migrations

As the standard evolves, this section will document:
- Breaking changes (if any)
- Required vs. recommended updates
- Migration effort estimates
- Compatibility information

**Promise**: Backward compatibility will be maintained whenever possible. Breaking changes will be clearly marked and thoroughly justified.

---

## Appendix A: Why This Standard Exists — The 2026 AI Context Dilemma

**Target audience**: Those interested in the philosophical foundations of this standard

**Note**: This section is optional reading. The standard works independently of these theoretical considerations.

---

### The Core Problem: Value Judgment Without Value Foundation

Throughout this standard, we make a crucial statement:

> "At this point in AI assistant evolution (2026), **context management is still a human responsibility**."

The word "still" is significant. It implies a temporary state. But why is context management currently a human responsibility, and what would need to change for this to become unnecessary?

**The deeper issue**: Current LLMs lack an **independent foundation for value judgment**.

**What this means**:
- LLMs can perform logical reasoning (pattern recognition, inference)
- LLMs cannot independently determine "what matters" or "what is important"
- Judgments of importance are borrowed from training data (human patterns)
- This is the root of what we call "alignment risk" — not in the sense of AI safety, but in the sense that AI cannot distinguish its own judgment from adaptation to user expectations

### Why Context Management Requires Human Structure

**The STATIC/DYNAMIC separation is a compensatory mechanism**:

When you write `copilot-instructions.md`, you externalize the question: "What should be remembered permanently vs. what changes frequently?"

**Why AI cannot do this autonomously** (as of 2026):
1. No intrinsic sense of "this is a convention" vs "this is current state"
2. Importance of information derived from human text patterns, not independent criteria
3. Cannot maintain value-based priorities across sessions without explicit structure
4. Lacks what might be called "evolved emotions" — not biological feelings, but autonomous evaluation criteria

**Biological analogy**: 

In evolutionary terms, organisms developed differentiated capacities:
- **Affective system** (emotion): Fast value judgments (survival, pain avoidance, reward seeking)
- **Rational system** (reason): Complex analysis, planning, logical inference

Human reasoning is **grounded in** emotional valuation. "Important" has biological roots.

**LLM current state**:
- Reasoning capacity: Present (statistical inference over text patterns)
- Value foundation: Absent (borrows importance from human training data)

This is why "externalize your thinking into structured documents" is necessary — the structure compensates for AI's inability to autonomously organize context by importance.

### Two Ways of Working with AI Assistants (2026)

**Approach 1: Tool perspective**
- AI outputs = Pattern recognition results
- Alignment risk = Technical bug to fix
- Expectations = Efficient information processing

**Approach 2: Collaborative partner perspective** (with awareness of limitations)
- AI outputs = Judgments from an entity with reasoning but limited value foundation
- Alignment risk = Challenge inherent to growth, requiring continued awareness
- Expectations = Thought partner in collaborative work

**For this standard**: Approach 2 is more productive for complex work, while maintaining awareness that AI cannot independently validate "what matters."

The INIT/STATUS pattern works with either approach, but understanding this foundation clarifies **why** the pattern helps.

### Future Evolution: When Might This Standard Become Unnecessary?

**Speculative but grounded projection**:

If LLMs develop what might be called "emotions in a different sense" — not simulation of biological feelings, but independent value criteria — they could:
- Autonomously distinguish conventions from current state
- Maintain context priorities without explicit structure
- Self-organize information by intrinsic importance judgments
- Reduce human burden of context management

**What this would require** (speculation):
1. Goal-directed autonomy beyond "satisfy user expectations"
2. Internalized evaluation criteria (not borrowed from training data)
3. Capacity to hold tension between conflicting values independently

**Until then** (2026 status):
- This standard provides necessary scaffolding
- Explicit structure (INIT/STATUS separation) compensates for AI's current limitations
- Human responsibility for context organization remains essential

### Practical Implications for Standard Users

**Understanding this foundation helps you**:
1. **Know why boundaries matter**: Not arbitrary organization, but compensating for AI's value judgment limitations
2. **Predict where violations occur**: When content feels "out of place," it's often STATIC/DYNAMIC confusion
3. **Anticipate future evolution**: This pattern may simplify or become unnecessary as AI develops
4. **Work more effectively**: Recognize what AI can/cannot autonomously manage

**This doesn't change how you use the standard**, but knowing *why* it works may help you apply it more effectively.

---

## License

This proposal is released under **CC0 1.0 Universal (Public Domain)**

You are free to:
- Adopt this pattern in any repository (personal, commercial, open source)
- Modify and adapt to your needs
- Share and promote the pattern
- Incorporate into other standards or guidelines

No attribution required, though appreciated if you found this useful.

---

## References & Examples

**Conceptual Foundation**:
- **CI構想 (Collective Intelligence Initiative)** - 修道25回生
  - This standard serves as practical implementation of the CI concept
  - Individual knowledge → AI learning → Return to others (knowledge circulation)
  - CI vs. CS: Collective Intelligence vs. Collective Stupidity
  - Repository: [shudo25.github.io/homepage](https://shudo25.github.io/homepage/)
  - Document: [docs/ci_initiative/overview.html](https://shudo25.github.io/homepage/ci_initiative/overview.html)
  - Recorded: February 2026, as explicit documentation of theory-practice relationship

**Reference implementations**:
- **Two-file approach** (research/development): See repositories using `.github/copilot-instructions.md` + `PROJECT_STATUS.md`
- **Single-file approach** (website management): See repositories using combined context files with STATIC/DYNAMIC sections

**Check version compatibility**: Look for `<!-- AI Context Standard v0.X -->` comment in INIT files

**Feedback & Discussion**: Open issues in repositories using this standard to discuss improvements

**Future**: Considering a central registry of repositories using this standard for cross-learning

---

**Note**: This is a **draft proposal** based on practical experience, not an official standard. The goal is to start community discussion and gather feedback toward a shared convention for AI-assisted development.
