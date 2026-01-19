# Getting Started

## Requirements

- Python >= 3.9. (Source: `pyproject.toml` `[project]`)
- Dependencies are installed by `pip` from `pyproject.toml`. (Source: `pyproject.toml` `[project]`)

## Install

```bash
pip install git+https://github.com/<your-org>/<your-repo>.git
```

(Source: `pyproject.toml` `[project]`)

For a local editable install from the bundled snapshot folder:

```bash
pip install -e ./Install
```

(If you're already in `Install`, use `pip install -e .`.)

## Prepare inputs

1) Create a pseudobulk folder that contains count and metadata CSVs:

- Counts: `*_pseudobulk_counts.csv`
- Metadata: `*_metadata.csv`

(Source: `edger_batch1.py` `DEFAULT_COUNTS_PATTERN`, `METADATA_SUFFIX`)

2) Ensure counts and metadata are consistent:

- The counts CSV first column is a gene identifier; remaining columns are numeric sample counts. (Source: `EdgeR_pipeline.py` `_read_counts`)
- The metadata CSV must include a sample column and group column matching the counts. (Source: `EdgeR_pipeline.py` `main`)

3) Generate edgeR results in the pseudobulk folder using one of:

- `stream_bulk.run_deg(...)` (Python API). (Source: `stream_bulk/api.py` `run_deg`)
- `edger_batch1.run_edger_batch(...)` (Python API). (Source: `edger_batch1.py` `run_edger_batch`)
- `python EdgeR_pipeline.py <counts> <metadata> <sample_col> <group_col> ...` (CLI). (Source: `EdgeR_pipeline.py` `main`)

The dashboard expects `*_edgeR_results.csv` files to exist for each cell type. (Source: `stream_app.py` `build_stream_ui`)

## First run (CLI)

```bash
stream-bulk --pseudobulk-dir STREAM/Pseudobulk --output-dir STREAM/dashboards
```

(Source: `stream_bulk/cli.py` `main`)

Expected output:

- An HTML dashboard written to `STREAM/dashboards/<dataset>_<layer>_<timestamp>.html` (or `STREAM_OUTPUT_DIR`). (Source: `stream_app.py` `build_stream_ui`)

## First run (Python)

```python
from stream_bulk import run_stream

run_stream(pseudobulk_dir="STREAM/Pseudobulk", output_dir="STREAM/dashboards")
```

(Source: `stream_bulk/api.py` `run_stream`)

## Notes

- `run_stream` uses `ipywidgets` to drive the UI even in headless mode; missing it raises an error. (Source: `stream_bulk/api.py` `run_stream`)
- AI features require `OPENAI_API_KEY` or they will be skipped. (Source: `stream_app.py` `_fetch_openai_models`)
- If you see a scanpy/numba cache error like "cannot cache function ... no locator available", set `NUMBA_DISABLE_JIT=1` before running `stream-bulk` or importing `stream_app`.
