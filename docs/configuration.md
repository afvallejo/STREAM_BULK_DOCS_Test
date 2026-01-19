# Configuration

This package is configured primarily via environment variables. All variables listed below are taken directly from code paths that call `os.getenv` or set environment overrides.

## Core paths

- `PSEUDOBULK_DIR`: pseudobulk root directory. Required by `run_stream` if not passed as an argument. (Source: `stream_bulk/api.py` `run_stream`; `stream_app.py` pseudobulk lookup)
- `STREAM_OUTPUT_DIR`: output directory for dashboards. Defaults to `STREAM/dashboards` when not set. (Source: `stream_bulk/api.py` `run_stream`; `stream_app.py` `build_stream_ui`)
- `STREAM_LOG_DIR`: log directory override; defaults to `STREAM/logs`. (Source: `stream_app.py` `_setup_run_logger`)
- `OPENAI_MODEL_CACHE_PATH`: JSON cache for the OpenAI model list; defaults to `STREAM/prompts/openai_models.json`. (Source: `stream_app.py` `_DEFAULT_MODEL_CACHE_PATH`, `_load_openai_model_catalog`)
- `CELL_PROMPT_DIR`: per-cell prompt output directory; also used as a fallback for concat outputs. (Source: `stream_app.py` per-cell prompt path resolution)
- `DEEP_RESEARCH_PROMPT_DIR`: directory for deep-research prompt/response files. (Source: `stream_app.py` deep research block)
- `CONCAT_OUTDIR`: output directory for `concat_harmonized` tables when invoked from the dashboard. (Source: `stream_app.py` concat config)
- `HARMONIZATION_OUTPUT_DIR`: directory for harmonization markdown/JSON files. (Source: `stream_app.py` harmonization write block)

## Feature toggles

- `STREAM_GSVA_ENABLED`: enable or disable GSVA scoring and limma batch. (Source: `stream_app.py` `_gsva_runtime_enabled`; `stream_bulk/api.py` `run_stream`)
- `STREAM_AI_ENABLED`: enable or disable AI summaries. (Source: `stream_app.py` `build_stream_ui`)
- `STREAM_DEEP_RESEARCH_ENABLED`: enable or disable deep research (requires AI). (Source: `stream_app.py` `build_stream_ui`)
- `STREAM_WGCNA_ENABLED`: enable or disable WGCNA exports. (Source: `stream_app.py` WGCNA block)
- `SAVE_HARMONIZATION_FILES`: set to `0` to skip saving harmonization markdown/JSON. (Source: `stream_app.py` harmonization write block)
- `USE_STRUCTURED_OUTPUTS`: enable or disable structured output mode for OpenAI calls. (Source: `structured_llm_wrapper.py`)

Boolean env flags are parsed with `1,true,yes,y,on` as truthy. (Source: `stream_app.py` `_env_flag`; `stream_bulk/api.py` `_env_override_bool`)

## OpenAI settings

- `OPENAI_API_KEY`: required for AI summaries and model catalog. (Source: `stream_app.py` `_fetch_openai_models`)
- `OPENAI_MODEL`: set by `run_stream` when `model` is provided; no direct usage in `stream_app.py` was found. (Source: `stream_bulk/api.py` `run_stream`)

## Per-cell prompt settings

- `PER_CELL_SPECIES` (default `human`). (Source: `stream_app.py` per-cell settings)
- `PER_CELL_GENE_IDS` (default `HGNC`). (Source: `stream_app.py` per-cell settings)
- `PER_CELL_WEIGHTS` (default `baseline_fraction`). (Source: `stream_app.py` per-cell settings)
- `PER_CELL_MAX_OUTPUT_TOKENS` (default `20000`). (Source: `stream_app.py` per-cell settings)
- `PER_CELL_FDR` (default `0.05`). (Source: `stream_app.py` per-cell settings)
- `PER_CELL_MIN_LOG2FC` (default `0.25`). (Source: `stream_app.py` per-cell settings)

## Harmonization settings

- `HARMONIZE_SPECIES` (default `human`). (Source: `stream_app.py` harmonization settings)
- `HARMONIZE_FDR` (default `0.05`). (Source: `stream_app.py` harmonization settings)
- `HARMONIZE_MIN_LOG2FC` (default `0.25`). (Source: `stream_app.py` harmonization settings)
- `HARMONIZE_BACKGROUND_RULE` (default `intersection`). (Source: `stream_app.py` harmonization settings)
- `HARMONIZE_WEIGHTS` (default `baseline_fraction`). (Source: `stream_app.py` harmonization settings)
- `HARMONIZE_GENE_META` (default `Stouffer`). (Source: `stream_app.py` harmonization settings)
- `HARMONIZE_TERM_META` (default `Stouffer`). (Source: `stream_app.py` harmonization settings)
- `HARMONIZE_REG_META` (default `Stouffer`). (Source: `stream_app.py` harmonization settings)
- `HARMONIZE_INTERX_META` (default `Stouffer`). (Source: `stream_app.py` harmonization settings)
- `HARMONIZE_PROMPT_LIMIT` (default `400000`). (Source: `stream_app.py` harmonization settings)
- `HARMONIZATION_MAX_OUTPUT_TOKENS` (default `100000`). (Source: `stream_app.py` harmonization settings)

## Deep research settings

- `DEEP_RESEARCH_PROMPT_DIR`: directory for deep research prompt/response files. (Source: `stream_app.py` deep research block)

## Concat settings

- `CONCAT_FILTER_SIGNIFICANT`: enable q-value filtering when concatenating. (Source: `stream_app.py` concat config)
- `CONCAT_Q_GENES`, `CONCAT_Q_TERMS`, `CONCAT_Q_TFS`, `CONCAT_Q_EDGES`: q-value thresholds. (Source: `stream_app.py` concat config)

## EdgeR and GSVA limma settings

These environment variables are consumed by the CLI pipelines:

- `RELEVEL`: reference group. (Source: `EdgeR_pipeline.py` `main`; `GSVA_limma_pipeline.py` `main`)
- `CONTRAST_GROUPS`: comma-separated contrast groups. (Source: `EdgeR_pipeline.py` `main`; `GSVA_limma_pipeline.py` `main`)
- `INTERACTION_GROUPS`: four comma-separated groups for interaction contrasts. (Source: `EdgeR_pipeline.py` `main`; `GSVA_limma_pipeline.py` `main`)
- `USE_DUPLICATE_CORRELATION`: enable duplicateCorrelation in EdgeR pipeline. (Source: `EdgeR_pipeline.py` `main`)
- `CONTRAST_MODE`: `pairwise` or `interaction` for GSVA limma. (Source: `GSVA_limma_pipeline.py` `main`)
- `BLOCKING_METHOD`: `design` or `duplicatecorrelation` for GSVA limma. (Source: `GSVA_limma_pipeline.py` `main`)
- `GSVA_SVA_ENABLED`: toggles SVA attempt in GSVA limma pipeline. (Source: `GSVA_limma_pipeline.py` `main`)

## Tissue type (harmonization prompt)

- `Tissue_type`, `TISSUE_TYPE`, `tissue_type`, `TissueType`, `tissueType`: resolved case-insensitively to supply tissue context. (Source: `harmonization_prompt.py` `_resolve_tissue_type_env`)
