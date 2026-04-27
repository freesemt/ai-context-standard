# Project Status — ai-context-standard

**Last Updated**: 2026-04-27

---

## 🎯 Current Task

Standard maintenance complete. No active task.

---

## 📋 Recent Work

### 2026-04-27 (v0.9.0 → v0.9.1)

- **User memory as cross-project propagation layer** (v0.9.1): new section in `AI_CONTEXT_STANDARD.md` documenting user memory as the right home for cross-project behavioral defaults; explains what belongs in user memory vs. repo `copilot-instructions.md`; includes guidance on seeding user memory on first adoption
- **NOTEBOOK_CONVENTIONS.md v0.2.4**: `aicRunCellAsync` added to Convention 5 (⏳ cells), responsibility table, AI cueing examples, and Tooling section
- **ai-context-vscode README**: `aicRunCellAsync` tool section added

---

### 2026-04-16 (v0.8.9 → v0.9.0 → ongoing)

- **vscode-version-recorder merged into ai-context-vscode**: `updateVersionFiles()` ported into `ai-context-vscode/src/extension.ts`; extension bumped to v0.2.0 with `onStartupFinished` activation
- **All vscode-version-recorder references replaced**: `AI_CONTEXT_STANDARD.md` install commands, all 11 `init.prompt.md` files across adopting repos updated to reference `freesemt/ai-context-vscode v0.2.0`
- **`freesemt/vscode-version-recorder` archived** on GitHub (superseded by `ai-context-vscode`)
- **AI-friendliness as normative principle** (v0.9.0): moved above "examples" disclaimer in `AI_CONTEXT_STANDARD.md`; template `copilot-instructions.md` now includes "AI-Friendliness Scope" subsection
- **`NOTEBOOK_CONVENTIONS.md` v0.1.0**: published; all adopting repos updated with notebook workflow reference
- **`freesemt/ai-context-vscode` GitHub repo created and pushed**; `freesemt/ai-context-tools` pushed to PyPI

### 2026-04-02 (v0.8.2 → v0.8.6)

- **v0.8.3**: `init.prompt.md` template — `agent: ask` → `agent: copilot`; clarified `vscode-version.txt` repo path
- **v0.8.4**: Added PowerShell `git -C` convention to Working Conventions section
- **v0.8.5**: Added core principle box to document header; redirected Working Conventions to `copilot-instructions.md`; removed `/memories/` from role table (contradicts repo-based principle)
- **v0.8.6**: Removed `/memories/` from role separation table (same reason)
- **2026-04-02**: Applied standard to this repository itself (added `copilot-instructions.md`, `PROJECT_STATUS.md`, `init.prompt.md`, `vscode-version.txt`)

### Key design decisions made today

- `agent: ask` in `init.prompt.md` was intentional at the time, but `agent: copilot` is now the correct choice (VS Code 1.99+)
- `/memories/` is not the right place for AI operating conventions — `copilot-instructions.md` is
- Cross-repo AI conventions belong in the standard's own `copilot-instructions.md`, not duplicated across adopting repos

---

## ⏳ Next Steps

1. Publish `ai-context-vscode` v0.2.0 as a GitHub Release (so `gh release download` works for new adopters)
2. Consider promoting from Draft toward a stable release
