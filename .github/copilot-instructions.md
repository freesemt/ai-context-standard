<!-- AI Context Standard v0.8 - Adopted: 2026-04-02 -->
# AI Assistant Initialization Guide — ai-context-standard

**Purpose**: Initialize AI context for working with this repository

> **On every session start**: Read [`PROJECT_STATUS.md`](../PROJECT_STATUS.md) to get the current task and recent context before responding.

---

## 📚 Core Documents to Read

1. **This file** — working conventions and repository structure
2. **PROJECT_STATUS.md** — current version, recent changes, next steps
3. **AI_CONTEXT_STANDARD.md** — the standard itself (primary artifact)

---

## 🎯 Repository Context

**Project**: AI Context Management Standard  
**Repository**: https://github.com/freesemt/ai-context-standard  

**Mission**: Define and maintain a minimal, portable standard for managing AI assistant context across sessions in software repositories. The standard is discovered through practical use, not designed top-down.

**Primary artifact**: `AI_CONTEXT_STANDARD.md`  
**Current version**: See PROJECT_STATUS.md

---

## 📂 Repository Structure

```
ai-context-standard/
├── AI_CONTEXT_STANDARD.md    # The standard (primary artifact)
├── README.md                 # Overview and quick reference
└── .github/
    ├── copilot-instructions.md   # This file
    ├── prompts/
    │   └── init.prompt.md        # Auto-initialization prompt
    └── vscode-version.txt        # VS Code version record
```

---

## 🔑 Working Conventions

### Versioning

- Version format: `0.8.x (Draft)` — bump patch on each substantive change
- Update both `AI_CONTEXT_STANDARD.md` header and `README.md` version line
- Commit message format: `docs: v0.8.x - <summary of change>`

### Editing Principles

- This standard is **axiomatic**: keep it minimal. Resist adding prescriptive detail.
- Changes should be grounded in **practical experience**, not theory
- When a new practice is discovered, ask: "Does this belong in `copilot-instructions.md` of the adopting repo, or in the standard itself?"

### Cross-repo AI Operating Conventions

These conventions apply to AI assistants working in **any repo adopting this standard**.  
They belong here (the standard's home repo) rather than duplicated across all adopting repos.

**Failure Recovery**: When the same operation fails 3+ times, stop and explain to the user. Propose alternatives. Do not silently retry 15+ times.

**PowerShell multi-repo git**: Always use `git -C <path>` instead of `cd <path>; git ...`.  
The terminal tool may silently strip `cd` from chained commands.

```powershell
# ❌ Unreliable
cd C:\path\to\repo; git commit -m "..."

# ✅ Reliable
git -C C:\path\to\repo commit -m "..."
```

---

## 💡 Quick Tips for AI Assistants

- **Primary task**: Updating `AI_CONTEXT_STANDARD.md`, version bumping, keeping README in sync
- **Key judgment**: "Where does this belong?" — always return to the core principle (context lives in the repo)
- **Avoid**: Adding non-axiomatic detail, duplicating conventions across repos, using `/memories/` for repo-scoped facts
