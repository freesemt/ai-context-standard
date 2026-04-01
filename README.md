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

## Read the Standard

→ [AI_CONTEXT_STANDARD.md](AI_CONTEXT_STANDARD.md)

## Related Tools

- [vscode-version-recorder](https://github.com/freesemt/vscode-version-recorder) — VS Code extension that auto-updates `vscode-version.txt` on startup

## Version

Current: **0.8.7 (Draft)**
