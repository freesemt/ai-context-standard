# Terminal Conventions (Windows PowerShell)

**Purpose**: Avoid wasted round trips when an AI assistant runs `git` / `gh` and other native commands in a Windows PowerShell terminal  
**Created**: October 21, 2026  
**Version**: 0.1.0 (Draft)  
**Part of**: [AI Context Standard v0.10.3](AI_CONTEXT_STANDARD.md)

**Single source of truth.** Adopting repositories do **not** copy these rules; their `copilot-instructions.md` carries only a pointer (see [Adopting](#adopting-in-a-repository)).

**Read this when**: you are about to commit or push from a terminal, or a `git` / `gh` command printed a red `NativeCommandError`.

Origin: AI-friendliness review of a molass-researcher session (2026-10). Related, documented elsewhere in the standard: [`git -C <path>` for multi-repo operations and `gh issue create --body-file`](AI_CONTEXT_STANDARD.md) ("Example: PowerShell Terminal"), and [non-ASCII file hazards (cp932, BOM)](TOOL_ROUTING_RULES.md).

---

## Convention 1 — A red `NativeCommandError` is not a failure

Windows PowerShell 5.1 wraps anything a native command writes to **stderr** as an error record, and the exit code may read 1. Git writes push / fetch / clone *progress* to stderr, and `gh` writes status lines such as `✓ Closed issue ...` there too.

```
git : To https://github.com/owner/repo.git           ← red NativeCommandError text
   8436af1..b50c3fa  main -> main                    ← this line is the result
```

**Rule**: judge success by the command's own result line (`a1b2c3d..e4f5a6b  main -> main`, `✓ Closed issue`) or by `git -C <repo> status -sb` (no `ahead` / `behind`). Do **not** retry because of the red text — a second push or a second `gh issue close` only adds noise.

## Convention 2 — Commit messages and issue bodies go through a file

`git commit -F -` fed from a here-string fails in this environment, and multi-line `-m` quoting is fragile.

**Rule**: write the message to a **fresh** file with the editor's file-creation tool (create tools refuse to overwrite, so use a new name per message), then:

```powershell
git -C C:\path\to\repo commit -F C:\path\to\message.txt
gh -R owner/repo issue create --title "..." --label enhancement --body-file C:\path\to\body.md
```

Keep trailers such as `Co-authored-by:` inside the message file.

## Convention 3 — Silence from `Select-Object -Last N` is expected

`| Select-Object -Last N` (and `| Out-String` on a long pipeline) prints nothing until the process ends. A long test run or build that looks silent is not hung; do not kill it. Use a generous `initial_wait`, or run it asynchronously and read the output when notified.

## Convention 4 — Tool calls in one batch run in parallel

When several tool calls are issued together they run concurrently. A call that **reads** an artifact (a PNG, a JSON file, a file written by a script) must be in a **later** batch than the call that **creates** it, otherwise it reads a stale or missing file.

---

## Adopting in a repository

Add this to `.github/copilot-instructions.md` (next to the "Notebook workflow" section). Do not paste the conventions themselves:

```markdown
## Terminal conventions (Windows PowerShell)
Read [TERMINAL_CONVENTIONS.md v0.1.0](https://github.com/freesemt/ai-context-standard/blob/main/TERMINAL_CONVENTIONS.md) before committing or pushing from a terminal, and whenever a `git` / `gh` command prints a red `NativeCommandError` (usually not a failure).
```

When this document's version changes, the pointer's version number is the only thing to update in adopting repos.

---

## Version History

**v0.1.0** (October 21, 2026) — Initial: four conventions (stderr progress is not failure, message files, buffered output, parallel tool batches).
