# File Formats

## Pseudobulk counts CSV (`*_pseudobulk_counts.csv`)

- CSV with a header row.
- First column is a gene identifier, used as row names.
- Remaining columns are numeric counts per sample.

(Source: `EdgeR_pipeline.py` `_read_counts`; `edger_batch1.py` `DEFAULT_COUNTS_PATTERN`)

## Pseudobulk metadata CSV (`*_metadata.csv`)

- CSV with columns that include the sample column and group column.
- Optional columns include donor and confounders.
- Sample IDs must match the counts column names.

(Source: `EdgeR_pipeline.py` `main`; `edger_batch1.py` `run_edger_batch`)

## EdgeR results CSV (`*_edgeR_results.csv`)

- Produced by `EdgeR_pipeline.py` as `<contrast>_edgeR_results.csv`.
- Includes `gene_symbol`, CPM columns, `log2FoldChange`, `LR`, `PValue`, `FDR`, `de`, `sig`.

(Source: `EdgeR_pipeline.py` `main` output table)

## GSVA scores CSV (`*_gsva_scores.csv`)

- Written by the dashboard under `STREAM/GSVA/<tab>/<cell>_gsva_scores.csv`.
- Values are the transpose of the in-memory GSVA score matrix.

(Source: `stream_app.py` `_export_gsva_scores`)

## GSVA limma results CSV (`*_limma_gsva_results.csv`)

- Written next to GSVA scores by `GSVA_limma_pipeline.py`.
- Includes a `gene_set` column plus limma `topTable` output columns (e.g., `logFC`, `P.Value`, `adj.P.Val`).

(Source: `GSVA_limma_pipeline.py` `main`)

## Dashboard HTML

- Output file name pattern: `<dataset>_<layer>_<timestamp>.html`.
- Written to `STREAM/dashboards` or `STREAM_OUTPUT_DIR`.

(Source: `stream_app.py` `build_stream_ui`)

## Harmonization outputs

- When enabled, harmonization markdown and JSON files are written to `HARMONIZATION_OUTPUT_DIR` or the per-cell prompt directory.

(Source: `stream_app.py` harmonization write block)

## concat_harmonized outputs

- `concat_payloads` returns a dict with `frames` and `written` keys.
- When `outdir` is set, it writes CSVs for: `summary`, `genes`, `pathways`, `regulators`, `interactions`, `disease`, `hypotheses`, `json_payloads`, `catalog_long`, plus `markdown_*` tables when markdown is provided.

(Source: `concat_harmonized.py` `concat_payloads`)
