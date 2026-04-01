# Project Status — ai-context-standard

**Last Updated**: 2026-04-02

---

## 🎯 Current Task

Standard maintenance and self-application: applying the standard to this repository itself.

---

## 📋 Recent Work

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

1. Consider whether `README.md` version line needs updating (`0.8` → `0.8.6`)
2. Consider promoting from Draft toward a stable release
