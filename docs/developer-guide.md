# Developer Guide

## Project layout

- `stream_bulk/`: package API and CLI.
- `stream_app.py`: UI and dashboard pipeline.
- `edger_batch1.py`: batch EdgeR/GSVA runner and UI.
- `EdgeR_pipeline.py`, `GSVA_limma_pipeline.py`: CLI scripts.
- `structured_llm_wrapper.py`, `harmonization_prompt.py`, `concat_harmonized.py`: AI and harmonization utilities.
- `STREAM/` and `stream_bulk/assets/`: bundled assets.

(Source: repo tree; `pyproject.toml` `tool.setuptools`)

## Development setup

Unknown from provided code. The repository does not define a development install script, lint configuration, or formatting tooling. (Source: repository scan)

## Adding features safely

- Prefer minimal changes and follow existing module boundaries (`stream_bulk` API, `stream_app` pipeline, `edger_batch1` batch utilities). (Source: module layout)
- If you add environment variables, document them in `docs/configuration.md`.

## Where to add docs/tests

- Documentation lives under `docs/` and the root `README.md`.
- Tests are stand-alone Python scripts in the repository root (e.g., `test_structured_outputs.py`, `test_compression.py`).

(Source: repository tree)
