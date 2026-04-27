# Repository Instructions

## Project Shape
- This is a notebook-first Python project; the executable source of truth is `obfuskacja_projekt_etap4.ipynb`.
- `context/` contains Polish planning/report material and presentation notes, not importable code.
- There are no root manifests, lockfiles, CI workflows, test configs, or packaged Python modules in the repo.

## Runtime
- Use the local venv at `.venv/` when running Python commands; `python` is not available on this machine, but `.venv/bin/python` is.
- Do not commit `.venv/`; this repo currently has no `.gitignore`.
- The notebook setup cell installs missing `numpy`, `matplotlib`, and `ipywidgets` via pip, then imports the standard libs used by the project.
- The notebook supports local Jupyter/VS Code and Google Colab; in Colab the setup cell enables the custom widget manager for `ipywidgets`.

## Notebook Architecture
- Core classes live in notebook cells: `BaseObfuscator`, `IdentifierRenamer`, `DeadCodeInserter`, `StringEncryptor`, `ControlFlowFlattener`, `ObfuscationPipeline`, `EntropyDetector`, and `CodeGenerator`.
- Default pipeline order is `string_encrypt, dead_code, cff, renaming`; keep `renaming` last because earlier steps introduce identifiers that also need renaming.
- `PARAM_SCHEMA` is the single source of truth for obfuscator parameters; it drives validation, `default_config()`, `build_pipeline()`, and UI widget generation.
- Config persistence is notebook-local JSON through `save_config(config, "config.json")` and `load_config("config.json")`, both gated by `validate_config()`.
- The UI entrypoint is `make_ui()` followed by the `etap4-display-ui` cell (`ui = make_ui(); display(ui)`).

## Verification
- There is no automated test suite. Verify changes by running the notebook top-to-bottom in Jupyter/VS Code/Colab.
- At minimum, rerun the setup cell first, then the changed cells plus dependent cells: schema/validation, pipeline builder, config I/O, UI helpers, `make_ui()`, and display.
- Existing documented smoke checks: all code cells execute, `save_config -> load_config` round-trips, invalid config is rejected, and the full pipeline preserves `bubble_sort` semantics on the listed sample cases in `context/etap4_podsumowanie_i_prezentacja.md`.

## Editing Notes
- Preserve Polish notebook text and labels unless the task explicitly asks for translation.
- Avoid adding a separate Python package structure unless requested; current project deliverables are the notebook plus context documents.
