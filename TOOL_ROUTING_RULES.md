# Tool Routing Rules for File Operations

**Purpose**: Prevent file corruption and tool misuse through systematic tool selection  
**Created**: June 22, 2026  
**Version**: 0.1.1 (Draft)  
**Part of**: [AI Context Standard v0.10.1](AI_CONTEXT_STANDARD.md)

---

## Core Principle

**Before ANY file write operation, check file type and constraints.**

File corruption from wrong tool choice is:
- Silent (no error at write time)
- Destructive (UTF-8 BOM, JSON escaping failures)
- Recoverable only via git (if committed)

Prevention requires **explicit routing rules**, not memory or judgment under pressure.

---

## The Mandatory Pre-Write Checklist

Before calling ANY file editing tool (`replace_string_in_file`, `edit_notebook_file`, `edit_lines`, PowerShell commands), perform this check:

### Step 1: Identify File Type

```
file_extension = Path(file_path).suffix
```

### Step 2: Look Up Constraint

Use the table below.

### Step 3: Use Specified Tool

Do not use alternatives. Do not fall back to PowerShell.

---

## Tool Routing Table

| File Type | MUST USE | NEVER USE | Reason |
|-----------|----------|-----------|--------|
| `.ipynb` | `edit_notebook_file` (closed)<br>`aicEditNotebookCell` (open) | `replace_string_in_file`<br>PowerShell `Set-Content`<br>`WriteAllText` | JSON escaping mismatch<br>UTF-8 BOM injection |
| `.html` (duplicates) | `aic-edit-lines` (line numbers) | `replace_string_in_file` | "Multiple matches" error |
| `.py` (marimo) | `aic_tools.marimo_session` | Direct file edit during server run | Session cache desync |
| Any (duplicates) | `python -m aic_tools.edit_lines` | `replace_string_in_file` | Unambiguous line-based edit |
| `.md`, `.py`, `.ts` (unique content) | `replace_string_in_file` | PowerShell on Japanese Windows | cp932 garbling of UTF-8 |

---

## Extended Constraints

### Jupyter Notebooks (`.ipynb`)

**Problem**: 
- `.ipynb` files store cell source as JSON strings
- `replace_string_in_file` matches decoded text, but file has JSON escaping (`\"` vs `"`)
- Match always fails
- PowerShell fallback injects UTF-8 BOM → `JSONDecodeError` → VS Code "Failed to save"

**Correct Tools**:
1. **Notebook open in VS Code**: `aicEditNotebookCell(cellNumber=N, source="...")`
   - Writes to live document model
   - Markdown cells re-render automatically
2. **Notebook closed**: `edit_notebook_file(filePath, editType='edit', cellNumber, source)`
3. **Batch operations**: `python -m aic_tools.runcell` for execution + edit

**Reference**: [NOTEBOOK_CONVENTIONS.md](NOTEBOOK_CONVENTIONS.md) Convention 3

---

### HTML / Any File with Duplicate Content

**Problem**:
- `replace_string_in_file` requires unique match
- Files with duplicate sections (copy-paste, templates) cause "Multiple matches found" error
- Retrying with more context often fails (duplicates still exist)

**Correct Tools**:
1. **Line-based edit**: `python -m aic_tools.edit_lines <file> <start> <end> <content_file>`
   - Uses line numbers (1-indexed)
   - Unambiguous even with duplicates
   - Creates `.bak` backup automatically
2. **Deletion**: `python -m aic_tools.edit_lines <file> <start> <end> --delete`

**Install**: `pip install ai-context-tools`

---

### PowerShell on Japanese Windows

**Problem**:
- PowerShell 5.1 defaults to cp932 (Shift-JIS) encoding
- `Get-Content` / `.Replace()` / `Set-Content` on UTF-8 files:
  - Garbles multi-byte characters (–, Δ, Å, →, ×)
  - **Destructively consumes adjacent ASCII bytes** (irreversible without git)
- `Set-Content -Encoding utf8` injects UTF-8 BOM even for pure ASCII
  - `.ipynb` with BOM → `JSONDecodeError` → VS Code save failure

**Correct Tools**:
- `replace_string_in_file` (VS Code tool, UTF-8 safe)
- `edit_notebook_file` (for `.ipynb`)
- `aic-edit-lines` (for any file)
- Python `Path.write_text(..., encoding="utf-8")` (no BOM)

**NEVER**:
- `Get-Content` + `.Replace()` + `Set-Content`
- `[System.IO.File]::WriteAllText()` without explicit UTF-8 no-BOM

**Recovery from BOM**: `git checkout <file>`, then:
```python
p = Path("file.ipynb")
p.write_text(p.read_text(encoding="utf-8-sig"), encoding="utf-8")
```

---

## Decision Flow

```
┌─────────────────────────────────┐
│ Before ANY file write operation │
└────────────┬────────────────────┘
             │
             ▼
      ┌──────────────┐
      │ File type?   │
      └──┬───┬───┬───┘
         │   │   │
    .ipynb │   │ other
         │   │   │
         │   │   └──────► replace_string_in_file
         │   │            (if unique content)
         │   │
         │   └─ .html / duplicates ──► aic-edit-lines
         │
         └─ Notebook open? ─┬─ Yes ──► aicEditNotebookCell
                            │
                            └─ No ───► edit_notebook_file
```

---

## Implementation Guidance

### For AI Assistants

**Commit to this workflow**:

1. When user requests file edit, FIRST extract file path
2. Check file extension
3. Look up constraint in Tool Routing Table
4. If constraint exists, use specified tool
5. If uncertain, search: `tool_search("edit <file_type> file")`
6. Document tool choice: "Using X because file is Y"

**When primary tool fails**:

1. DO NOT fall back to PowerShell
2. DO search for specialized tool: `tool_search("edit notebook")`, `tool_search("line edit")`
3. DO check this document
4. If no specialized tool found, ASK USER for guidance

**Self-check questions**:
- Did I check the file extension?
- Did I verify this tool is safe for this file type?
- Is there a constraint I'm ignoring?
- Am I choosing convenience over correctness?

---

### For Humans

**When setting up a new project**:

1. Reference this document in `.github/copilot-instructions.md`:
   ```markdown
   Read [TOOL_ROUTING_RULES.md](https://github.com/freesemt/ai-context-standard/blob/main/TOOL_ROUTING_RULES.md)
   before any file write operation.
   ```

2. Add project-specific constraints if needed:
   ```markdown
   | `.toml` (pyproject.toml) | `replace_string_in_file` | Manual edit during build | TOML parser may be running |
   ```

3. Store in user memory if AI frequently works on specific file types in your workspace

---

## Deferred Tool Loading (VS Code Agent Mode)

In VS Code Agent mode, many tools are **deferred** — they are not loaded into the agent's active tool set until explicitly requested via `tool_search`. Calling a deferred tool without first loading it causes a **silent hang** with no error message.

**Deferred tools that require `tool_search` first**:
- `aicEditNotebookCell`, `edit_notebook_file`, `aicReadLiveCellOutput`
- `aicRunCellAsync`, `aicListNotebookCells`, `aicKernelEval`
- `copilot_getNotebookSummary`, `run_notebook_cell`
- `github-pull-request_*`, `mcp_gitkraken_cli_*`

**Rule**: Before calling ANY tool with `aic`, `edit_notebook`, `copilot_get`, `run_notebook`, or `github-pull-request` in its name, call `tool_search` first:

```python
tool_search("read notebook cell output")   # → loads aicReadLiveCellOutput
tool_search("edit notebook cell")          # → loads aicEditNotebookCell
tool_search("list notebook cells")         # → loads aicListNotebookCells
```

**Symptom of violation**: The agent's response is sent but nothing happens. The chat appears to hang. The user must retry.

**Fallback when unsure**: Provide the code/content directly in the chat response instead of calling the tool. This avoids the dependency entirely.

---

## Updates

**Version 0.1.1** (July 14, 2026):
- Added "Deferred Tool Loading" section: rule for calling `tool_search` before deferred tools to prevent silent hangs

**Version 0.1.0** (June 22, 2026):
- Initial release
- Core routing table: `.ipynb`, `.html` duplicates, PowerShell hazards
- Added `aic-edit-lines` as canonical solution for duplicate-content problem
- Decision flow diagram

**Maintenance**: This document should be updated when:
- New file type constraints discovered through incidents
- New safe tools added to ecosystem (ai-context-tools, ai-context-vscode)
- Tool APIs change (VS Code, extensions)

---

## References

- [NOTEBOOK_CONVENTIONS.md](NOTEBOOK_CONVENTIONS.md) — Jupyter notebook workflow conventions
- [ai-context-tools](https://github.com/freesemt/ai-context-tools) — Python CLI utilities including `edit_lines`
- [ai-context-vscode](https://github.com/freesemt/ai-context-vscode) — VS Code extension providing `aicEditNotebookCell`, `aicReadLiveCellOutput`

---

**License**: MIT License — Part of the AI Context Standard ecosystem
