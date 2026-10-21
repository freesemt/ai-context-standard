# ai-context-standard

The canonical source for the **AI Context Management Standard** — a minimal pattern for maintaining AI assistant context across sessions in software repositories.

## What Is This?

A practical standard discovered through real-world use with GitHub Copilot across multiple repositories.

**Core principle**: Separate STATIC conventions from DYNAMIC state.

| File | Role | When Updated |
|------|------|-------------|
| `.github/copilot-instructions.md` | How to work here (static conventions) | Rarely |
| `PROJECT_STATUS.md` | What's happening now (dynamic state) | Every session |
| `.github/prompts/init.prompt.md` | Initialization visibility + version check | Rarely |
| `.github/vscode-version.txt` | VS Code version record | Auto (via extension) |

## Companion Documents

| Document | Purpose |
|----------|---------|
| [NOTEBOOK_CONVENTIONS.md](NOTEBOOK_CONVENTIONS.md) | Jupyter notebook workflow conventions (cell output reading, execution, editing) |
| [TOOL_ROUTING_RULES.md](TOOL_ROUTING_RULES.md) | File editing safety — prevents corruption through systematic tool selection |
| [TERMINAL_CONVENTIONS.md](TERMINAL_CONVENTIONS.md) | Windows PowerShell terminal quirks for `git` / `gh` (red `NativeCommandError` ≠ failure, commit messages via file, buffered output, parallel tool batches); repos point to it instead of copying it |

## Read the Standard

→ [AI_CONTEXT_STANDARD.md](AI_CONTEXT_STANDARD.md)

## Related Tools

| Repository | Description |
|---|---|
| [ai-context-vscode](https://github.com/freesemt/ai-context-vscode) | VS Code extension — auto-updates `vscode-version.txt` on startup |
| [ai-context-tools](https://github.com/freesemt/ai-context-tools) | Python CLI tools for AI context management (published to PyPI) |
| [ai-context-standard](https://github.com/freesemt/ai-context-standard) | This repository — the standard document itself |

> Note: `freesemt/vscode-version-recorder` has been archived and superseded by `ai-context-vscode`.

## Version

Current: **0.10.3 (Draft)**
