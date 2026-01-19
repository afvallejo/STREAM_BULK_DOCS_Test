# User Guide

## Workflow: pseudobulk-only dashboard

1) Populate `STREAM/Pseudobulk` with counts and metadata CSVs.
2) Run edgeR to produce `*_edgeR_results.csv` for each cell type.
3) Run the dashboard with `stream-bulk` or `stream_bulk.run_stream`.

Sources: `edger_batch1.py` `DEFAULT_COUNTS_PATTERN`, `METADATA_SUFFIX`; `EdgeR_pipeline.py` `main`; `stream_bulk/api.py` `run_stream`; `stream_app.py` `build_stream_ui`.

### Example: run the dashboard

```bash
stream-bulk --pseudobulk-dir STREAM/Pseudobulk --output-dir STREAM/dashboards
```

(Source: `stream_bulk/cli.py` `main`)

## Workflow: differential expression (EdgeR)

### Example: batch EdgeR via Python API

```python
from stream_bulk import run_deg

summary = run_deg(
    folder="STREAM/Pseudobulk",
    sample_col="Sample",
    group_col="disease",
    contrast_mode="pairwise",
    contrast_groups=("control", "treated"),
    relevel="control",
    confounder_cols=["age", "sex"],
)
```

(Source: `stream_bulk/api.py` `run_deg`; `edger_batch1.py` `run_edger_batch`)

`run_deg` wraps `edger_batch1.run_edger_batch` and writes results into `STREAM/Pseudobulk` by default. (Source: `stream_bulk/api.py` `run_deg`)

### Example: interactive EdgeR UI (notebook)

```python
from edger_batch1 import launch_edger_batch_ui

launch_edger_batch_ui(folder="STREAM/Pseudobulk")
```

(Source: `edger_batch1.py` `launch_edger_batch_ui`)

## Workflow: GSVA scoring and limma contrasts

- GSVA scores are exported to `STREAM/GSVA/<tab>/<cell>_gsva_scores.csv`. (Source: `stream_app.py` `_export_gsva_scores`)
- Limma contrasts are executed by calling `edger_batch1.run_edger_batch` with `GSVA_limma_pipeline.py`. (Source: `stream_app.py` `_run_limma_batch_for_gsva_exports`)
- Limma results are written as `*_limma_gsva_results.csv` next to each GSVA score file. (Source: `GSVA_limma_pipeline.py` `main`)

### Example: enable GSVA in the dashboard

```python
from stream_bulk import run_stream

run_stream(pseudobulk_dir="STREAM/Pseudobulk", gsva=True)
```

(Source: `stream_bulk/api.py` `run_stream`)

## Workflow: WGCNA exports

When WGCNA is enabled, outputs are written under `STREAM/WGCNA/<cell_key>/` and include:
- `module_colors.csv`
- `module_trait_correlation.csv`
- `module_trait_pvalue.csv`
- `hub_genes_signed_kme.csv`
- `go_enrichment.csv` (if GO enrichment succeeds)
- `kegg_enrichment.csv` (if KEGG enrichment succeeds)
- `soft_threshold.csv`

(Source: `stream_app.py` WGCNA block around `module_colors_df.to_csv`)

## Workflow: AI summaries and harmonization

- AI features require `OPENAI_API_KEY`. (Source: `stream_app.py` `_fetch_openai_models`)
- Deep research uses `o4-mini-deep-research` with `web_search_preview`. (Source: `stream_app.py` deep research block)
- Structured output mode is controlled by `USE_STRUCTURED_OUTPUTS`. (Source: `structured_llm_wrapper.py` `create_per_cell_response_with_structured_output`)

### Example: generate a per-cell schema (from tests)

```python
from structured_llm_wrapper import get_per_cell_json_schema

schema = get_per_cell_json_schema(
    cell_key="t_cell",
    payload_id="test_abc123",
    project_title="Test PBMC Analysis",
    cell_type_label="T Cell",
    tissue_type="blood",
    contrast="stimulated_vs_control",
    species="human",
    gene_id_system="HGNC",
    baseline_fraction=0.6,
    delta_fraction=0.2,
    donors_n=15,
    weights="baseline_fraction",
    fdr=0.05,
    min_log2fc=0.25,
    generated_at="2025-01-15T10:30:00",
    prompt_checksum="sha1_abc123",
)
```

(Source: `test_structured_outputs.py` `test_per_cell_schema`)

## Workflow: concatenating harmonized outputs

`concat_harmonized.py` can merge JSON/Markdown outputs into CSV tables.

```bash
python concat_harmonized.py --inputs /path/to/harmonized/*.json --outdir /path/to/output
```

(Source: `concat_harmonized.py` `main`)
