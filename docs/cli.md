# Command Line Interface

## `stream-bulk`

**Invocation**:

```bash
stream-bulk --pseudobulk-dir <path> [--output-dir <path>] [--gsva/--no-gsva] [--ai/--no-ai] [--deep-research/--no-deep-research] [--wgcna/--no-wgcna] [--layer <value>] [--model <value>]
```

(Source: `stream_bulk/cli.py` `_build_parser`)

**Options**:
- `--pseudobulk-dir`: path to pseudobulk directory. (Source: `stream_bulk/cli.py`)
- `--output-dir`: output directory for dashboards. (Source: `stream_bulk/cli.py`)
- `--gsva/--no-gsva`: enable/disable GSVA scoring (ULM enrichment remains). (Source: `stream_bulk/cli.py`)
- `--ai/--no-ai`: enable/disable AI summaries. (Source: `stream_bulk/cli.py`)
- `--deep-research/--no-deep-research`: enable/disable deep research (requires AI). (Source: `stream_bulk/cli.py`)
- `--wgcna/--no-wgcna`: enable/disable WGCNA. (Source: `stream_bulk/cli.py`)
- `--layer`: optional layer override. (Source: `stream_bulk/cli.py`)
- `--model`: optional OpenAI model override (passed via environment). (Source: `stream_bulk/cli.py` and `stream_bulk/api.py` `run_stream`)

## `EdgeR_pipeline.py`

**Invocation**:

```bash
python EdgeR_pipeline.py <counts_csv> <metadata_csv> <sample_column> <group_column> [donnor_column] [confounder1] [confounder2] ...
```

(Source: `EdgeR_pipeline.py` `main`)

**Environment variables**:
- `RELEVEL`, `CONTRAST_GROUPS`, `INTERACTION_GROUPS`, `USE_DUPLICATE_CORRELATION`. (Source: `EdgeR_pipeline.py` `main`)

## `GSVA_limma_pipeline.py`

**Invocation**:

```bash
python GSVA_limma_pipeline.py <gsva_csv> <metadata_csv> <sample_column> <group_column> [donnor_column]
```

(Source: `GSVA_limma_pipeline.py` `main`)

**Environment variables**:
- `RELEVEL`, `CONTRAST_MODE`, `CONTRAST_GROUPS`, `INTERACTION_GROUPS`, `BLOCKING_METHOD`, `GSVA_SVA_ENABLED`. (Source: `GSVA_limma_pipeline.py` `main`)

## `concat_harmonized.py`

**Invocation**:

```bash
python concat_harmonized.py --inputs <glob_or_paths> --outdir <dir> [--filter-significant] [--q_genes <float>] [--q_terms <float>] [--q_tfs <float>] [--q_edges <float>]
```

(Source: `concat_harmonized.py` `main`)

**Exit codes**:
- `0`: success, tables written.
- `1`: no JSON input files found.
- `2`: no tables produced.

(Source: `concat_harmonized.py` `main` return values)
