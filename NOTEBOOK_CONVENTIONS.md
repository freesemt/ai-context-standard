# Notebook Collaboration Conventions

**Status**: Draft  
**Version**: 0.2.5
**Date**: April 27, 2026
**Context**: Companion to [AI Context Standard](AI_CONTEXT_STANDARD.md) for Jupyter notebook workflows in VS Code Agent mode

---

## Problem

Jupyter notebooks in VS Code Agent mode create friction points not present in plain code files:

- **Cell identity is invisible**: The AI sees internal cell IDs; the human sees execution counts that change every run. Neither side can reliably point to "that cell."
- **Kernel management conflicts**: The AI may auto-configure a kernel (creating venvs, selecting wrong interpreters) before the human has a chance to choose.
- **Output reading limitations**: Built-in tools truncate large cell outputs, leaving the AI unable to see what the human sees.
- **Cell ownership ambiguity**: When the AI silently creates, deletes, or reorders cells, the human loses track of notebook structure.

---

## Conventions

### 1. Cell Identification

Every executable cell starts with a numbered comment on the first line:

```python
# [1] Load data and quick decomposition
```

- Numbers are sequential: `[1]`, `[2]`, `[3]`, ...
- The description is brief (what the cell does)
- Markdown cells do not need numbering (they serve as section headers)
- When cells are inserted or reordered, renumber all cells to maintain sequence

**In conversation**, both human and AI refer to cells by this label: "cell [3]" or "the `[3] Run optimizer` cell."

### 2. Kernel Management

**Principle**: The human selects the kernel. The AI does not auto-configure.

- The AI should not call kernel configuration tools unless explicitly asked
- Each repo's `copilot-instructions.md` should document the project's kernel preference (global Python, venv, conda, specific path, etc.)
- If no kernel is selected when the AI needs to run a cell, it should ask rather than configure one

### 3. Cell Output Reading

The built-in `read_notebook_cell_output` tool may silently fail with "output too large" even for moderate stdout-heavy cells. Use this fallback chain:

1. **`aicReadLiveCellOutput`** (preferred) — reads directly from the live VS Code document model; no save required, no size limit. Install [`ai-context-vscode`](https://github.com/freesemt/ai-context-vscode) to enable this tool.
2. **`aic_tools.notebook`** — reads from the `.ipynb` file on disk; reflects the *last saved* state only.  
   `py -m aic_tools.notebook <notebook_path> <cell_number> [max_lines]`  
   Install: `pip install ai-context-tools`. On Windows, prefer the `py` launcher over bare `python` (the latter may resolve to the wrong interpreter or trigger a Microsoft Store stub dialog).
3. **Raw JSON parse** — last resort; load the `.ipynb` with `json` and walk `cells[i]['outputs']` directly. Useful when only one specific output (e.g. an embedded PNG) is needed.
4. **Ask the user** — only if none of the above are available; do not silently skip the output.

**Note on save state**: options 2 and 3 read from disk, so cell outputs must be saved first. `aicReadLiveCellOutput` has no such requirement — it sees the live state the human sees.

**Widget-rendered outputs**: Figures rendered into `ipywidgets.Output` (interactive dashboards, progress monitors) are **not persisted** to `cell.outputs` — only widget-view metadata is. None of the tools above can recover them. Either ask the user for a screenshot, or have the upstream tool optionally `savefig()` to disk (env-gated). See [ai-context-tools#6](https://github.com/freesemt/ai-context-tools/issues/6).

### 4. Cell Creation and Modification

- **Propose before creating**: Describe what the new cell will do and where it will go before inserting it
- **Do not silently delete or reorder cells**: Always confirm with the human
- **Preserve cell [N] labels**: When editing a cell, keep the `# [N]` comment line consistent with the current numbering

#### Editing cell source — correct tool order

`.ipynb` files are JSON on disk. `replace_string_in_file` **always fails on notebook cell source** because the tool matches against decoded text, but the file stores JSON-escaped strings (e.g. `\"` on disk vs `"` in the tool's view). The correct tools are:

| Situation | Tool |
|-----------|------|
| Edit existing cell content | `edit_notebook_file` with `editType=edit` |
| Insert a new cell | `edit_notebook_file` with `editType=insert` |
| Delete a cell | `edit_notebook_file` with `editType=delete` |
| `replace_string_in_file` fails on a notebook | **Use `edit_notebook_file` instead — never fall back to PowerShell** |

**Never use PowerShell (`Set-Content`, `Get-Content`, `.Replace()`) to edit `.ipynb` files.** Two failure modes:
- `Set-Content -Encoding utf8` injects a UTF-8 BOM → `JSONDecodeError: Unexpected UTF-8 BOM` → VS Code "Failed to save" error.
- `Get-Content` without explicit encoding defaults to cp932 (Shift-JIS) on Japanese Windows → silently garbles non-ASCII bytes.

**BOM recovery**: `git checkout <file>`, then:
```python
import pathlib
p = pathlib.Path("notebook.ipynb")
p.write_text(p.read_text(encoding="utf-8-sig"), encoding="utf-8")
```

### 5. Long-Running Cells and Responsibility Division

Mark long-running cells with `⏳` in the cell ID comment:

```python
# [12] ⏳ Run floor-constrained optimizer (long-running)
```

Mark paired "check progress" cells with `🔄`:

```python
# [13] 🔄 Check floor optimizer progress
```

Behavior:

- The AI should not block waiting on `⏳` cells
- The AI can fire `⏳` cells with `aicRunCellAsync` (fire-and-forget) — confirm with the human first, then start without blocking the conversation
- The human can also run the cell directly and report back when results are ready
- `🔄` cells are designed for repeated re-running to check status

#### Responsibility division

`⏳` cells define the boundary between human and AI responsibility:

| Responsibility | Cell types | Rationale |
|---|---|---|
| **Human or AI** | `⏳` (long-running) | The human decides when to spend time. The AI may fire with `aicRunCellAsync` after confirming — fire-and-forget keeps the conversation responsive. |
| **AI** | All other cells | Setup, analysis, diagnostics, visualization, summaries — fast, deterministic, re-runnable |

In practice: after a `⏳` cell completes (whether fired by the AI with `aicRunCellAsync` or run by the human directly), the AI takes over — running the downstream cells, reading outputs, and reporting findings.

This division is **independent of correctness**: the AI should confirm with the human before firing `⏳` cells.

#### AI cueing

When the AI has completed all preparatory work and a `⏳` cell is the next required step, it should **explicitly prompt the human** rather than stopping silently:

> "Everything is set up. Shall I fire cell [6] ⏳ now? I'll use `aicRunCellAsync` so the conversation stays responsive — I'll monitor progress and take over from cell [6a] once it completes."

Or, if the human prefers to run it:

> "Everything is set up. Please run cell [6] ⏳ when ready — I'll take over from cell [6a] onwards once it completes."

This prevents the human from wondering whether the AI is waiting, finished, or stuck.

### 6. Conformance Declaration

Notebooks following these conventions include an HTML comment in their first markdown cell:

```markdown
<!-- Follows NOTEBOOK_CONVENTIONS.md (ai-context-standard) -->
```

This tells both human and AI that cell `[N]` labels, `⏳`/`🔄` markers, and the other conventions in this document are in effect.

**Discovery order**: The primary trigger is the `copilot-instructions.md` reference (see [Adopting These Conventions](#adopting-these-conventions) below) — the AI reads this at session start, before opening any notebook. The HTML comment is a secondary confirmation visible when the AI reads the notebook itself.

### 7. "Run All" Compatibility

**Principle**: Every notebook should produce a complete, correct result when **Run All** is used — without manual intervention.

This means:

- **No forward references**: A cell must not use variables defined in a later cell
- **No "re-run this cell later" patterns**: If a cell depends on an asynchronous result (optimizer, training job, API call), it must wait for that result itself
- **Guard against missing results**: Use `try/except NameError` or conditional checks so that a cell failing upstream does not cascade into unrelated tracebacks

**Waiting for asynchronous results**: When a cell consumes results from a long-running operation started earlier in the notebook, it should poll until the results are available (with a timeout):

```python
# [5] Compare quick vs rigorous (waits for results)
if wait_for_results(folder, timeout=600):
    result = load_result(folder)
    # ... use result ...
else:
    print("Timed out waiting for results.")
```

The specific waiting mechanism is project-dependent. Examples:
- Filesystem polling (check for output files)
- API status checks
- Job queue queries

**Relationship to Convention 5**: The `⏳` cell that *starts* the long-running operation returns immediately. The *result-consuming* cell downstream is the one that waits. The `🔄` progress-check cells remain useful for interactive monitoring but are not required for "Run All" correctness.

### 8. Live Kernel Objects as Ground Truth

When a `⏳` cell returns a result object into kernel scope (a running job handle, a fitted model, a monitor object), that object is the **primary source of truth** for any subsequent status query. Do not bypass it to read raw files on disk.

Priority order:

1. **Live kernel object** — query its properties or methods directly (fastest, always current)
2. **Saved cell output** — read via `aicReadLiveCellOutput` or `aic_tools.notebook` (reflects last save)
3. **Raw files on disk** — parse log files, checkpoints, etc. (last resort; only when kernel is unavailable)

Each adopting repo's `copilot-instructions.md` should document what result objects `⏳` cells produce and how to query them.

---

## Tooling

When a friction point cannot be solved by convention alone, a tool belongs in one of two packages:

**[`ai-context-vscode`](https://github.com/freesemt/ai-context-vscode)** — VS Code extension tools (require VS Code; access the live document model):
- `aicReadLiveCellOutput` — reads cell output from the live document model; no save required, no size limit (solves Convention 3, preferred)
- `aicListNotebookCells` — lists all cells with type, execution count, and output summary from the live model
- `aicKernelEval` — evaluates a Python expression in the live Jupyter kernel and returns its `repr`; the canonical way to query kernel-scope objects (long-running job handles, fitted models, monitors) without inserting a new cell (supports Convention 8)
- `aicRunCellAsync` — fires a notebook cell and returns immediately (fire-and-forget); keeps the conversation responsive during long-running cells (supports Convention 5)

**[`ai-context-tools`](https://github.com/freesemt/ai-context-tools)** — Python package tools (work anywhere; read from disk):
- `aic_tools.notebook` — reads cell outputs from the saved `.ipynb` file (solves Convention 3, fallback)

Domain-specific readiness checks (Convention 5 and 7 support):
- `Decomposition.has_rigorous_results(analysis_folder)` — lightweight filesystem check for optimizer results (molass-library). Use in `🔄` progress-check cells.
- `Decomposition.wait_for_rigorous_results(analysis_folder, timeout=600)` — blocking poll until results are parseable (molass-library). Use in result-consuming cells for "Run All" compatibility.

Future candidates:
- Cell renumbering utility (automates Convention 1 maintenance)
- Kernel status checker (supports Convention 2)
- General-purpose file watcher for `ai-context-tools` (domain-agnostic readiness polling)

---

## Adopting These Conventions

Reference this document from your repo's `copilot-instructions.md`:

```markdown
## Notebook workflow
See [NOTEBOOK_CONVENTIONS.md v0.2.0](https://github.com/freesemt/ai-context-standard/blob/main/NOTEBOOK_CONVENTIONS.md)
Kernel preference: global Python (`py`). Do not create venvs.
```

Each repo adds its own project-specific choices (kernel, workflow details) while the conventions here remain general.
