# Notebook Collaboration Conventions

**Status**: Draft  
**Version**: 0.1.0  
**Date**: April 16, 2026  
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

Use tools in this priority order:

1. **`aicReadLiveCellOutput`** (preferred) — reads directly from the live VS Code document model; no save required, no size limit. Install [`ai-context-vscode`](https://github.com/freesemt/ai-context-vscode) to enable this tool.
2. **`aic_tools.notebook`** (fallback) — reads from the `.ipynb` file on disk; reflects the *last saved* state only. Install with `pip install ai-context-tools`.
3. **Ask the user** — only if neither tool is available; do not silently skip the output.

**Note on save state**: `aic_tools.notebook` reads from disk, so cell outputs must be saved first. `aicReadLiveCellOutput` has no such requirement — it sees the live state the human sees.

### 4. Cell Creation and Modification

- **Propose before creating**: Describe what the new cell will do and where it will go before inserting it
- **Do not silently delete or reorder cells**: Always confirm with the human
- **Preserve cell [N] labels**: When editing a cell, keep the `# [N]` comment line consistent with the current numbering

### 5. Long-Running Cells

Mark long-running cells with `⏳` in the cell ID comment:

```python
# [12] ⏳ Run floor-constrained optimizer (long-running)
```

Mark paired "check progress" cells with `🔄`:

```python
# [13] 🔄 Check floor optimizer progress
```

Behavior:

- The AI should not wait or block on `⏳` cells
- The human runs the cell and reports back when results are ready
- `🔄` cells are designed for repeated re-running to check status

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

---

## Tooling

When a friction point cannot be solved by convention alone, a tool belongs in one of two packages:

**[`ai-context-vscode`](https://github.com/freesemt/ai-context-vscode)** — VS Code extension tools (require VS Code; access the live document model):
- `aicReadLiveCellOutput` — reads cell output from the live document model; no save required, no size limit (solves Convention 3, preferred)
- `aicListNotebookCells` — lists all cells with type, execution count, and output summary from the live model

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
See [NOTEBOOK_CONVENTIONS.md v0.1.0](https://github.com/freesemt/ai-context-standard/blob/main/NOTEBOOK_CONVENTIONS.md)
Kernel preference: global Python (`py`). Do not create venvs.
```

Each repo adds its own project-specific choices (kernel, workflow details) while the conventions here remain general.
