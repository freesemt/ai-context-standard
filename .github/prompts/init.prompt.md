---
agent: copilot
description: Session initialization (load and summarize PROJECT_STATUS.md)
alwaysApply: true
---

## Step 1: Check VS Code Version

Read `.github/vscode-version.txt` in the `ai-context-standard` repository (the same repo as this prompt file).

- **If the line `Auto-updated by vscode-version-recorder extension` is present**: The extension is working correctly. Use that version number and proceed to Step 2.
- **If a version number is present but that line is missing**: The extension may not be installed. Use the version number and proceed to Step 2. Then ask the user:

  > ⚠️ **ai-context-vscode extension not detected**  
  > Would you like to install it automatically?

  If the user agrees, run the following commands:

  ```powershell
  gh release download v0.2.0 --repo freesemt/ai-context-vscode --pattern "*.vsix" --dir $env:TEMP
  code-insiders --install-extension "$env:TEMP\ai-context-vscode-0.2x"
  ```

  After installation, ask the user to **restart VS Code**.

- **If no version number is recorded or the file does not exist**, display the following:

  > ⚠️ **Please record your VS Code version**  
  > Go to Menu → **Help** → **About** → copy the first line (e.g. `1.115.0-insider`)  
  > and append it to `.github/vscode-version.txt`.

## Step 2: Check alwaysApply Support

Extract the numeric part of the version (e.g. `1.115` from `1.115.0-insider`) and check if it is **≥ 1.99**.

- **≥ 1.99** → ✅ `alwaysApply: true` is active. Auto-initialization is working correctly.
- **< 1.99** → ⚠️ `alwaysApply: true` is not supported. Run `/init` manually at the start of each session.

## Step 3: Summarize PROJECT_STATUS.md

Read `PROJECT_STATUS.md` and summarize in English:

1. **Current task** — what is being worked on
2. **Recent progress** — what was recently completed
3. **Next steps** — in priority order

End with: "**Initialized** (ai-context-standard) / VS Code vX.XX.X ✅"
