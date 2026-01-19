# API Reference

This reference covers the public surface identified from `__init__` exports, module `__all__`, entrypoints, and tests.

## Package: `stream_bulk`

**Purpose**: High-level API wrappers for filtering, differential expression, and running the dashboard pipeline. (Source: `stream_bulk/__init__.py`)

### `stream_bulk.filter_data`

**Signature**:

```python
def filter_data(
    counts: pd.DataFrame,
    group_labels: Optional[Sequence[str]] = None,
) -> tuple[pd.DataFrame, Iterable[bool]]:
```

**Behavior**: Filters low-expression genes using EdgeR CPM heuristics and returns the filtered DataFrame plus a boolean mask. (Source: `stream_bulk/api.py` `filter_data`)

**Parameters**:
- `counts`: counts table with genes as rows and samples as columns.
- `group_labels`: optional group labels used by EdgeR filtering.

**Returns**:
- `(filtered_counts, keep_mask)`.

**Raises**: propagates errors from EdgeR helper functions if inputs are invalid. (Source: `stream_bulk/api.py` `filter_data`)

### `stream_bulk.run_deg`

**Signature**:

```python
def run_deg(
    *,
    folder: str,
    rscript_path: Optional[str] = None,
    output_dir: Optional[str] = None,
    counts_pattern: str = "*_pseudobulk_counts.csv",
    counts_suffix: Optional[str] = None,
    metadata_suffix: str = "_metadata.csv",
    metadata_folder: Optional[str] = None,
    contrast_mode: str = "pairwise",
    interaction_groups: Optional[Sequence[str]] = None,
    confounder_cols: Optional[Sequence[str]] = None,
    sample_col: str = "Sample",
    group_col: str = "disease",
    donor_col: Optional[str] = None,
    relevel: Optional[str] = None,
    contrast_groups: Optional[tuple[str, str]] = None,
    blocking_method: str = "design",
    check_consistency: bool = True,
    require_both_groups: bool = True,
    fdr_thresh: float = 0.05,
    lfc_thresh_abs: float = 0.0,
    show_progress: bool = True,
    sva_enabled: bool = False,
) -> pd.DataFrame:
```

**Behavior**: Runs `edger_batch1.run_edger_batch` across a folder of pseudobulk counts and returns a summary DataFrame. Defaults `output_dir` to `STREAM/Pseudobulk`. (Source: `stream_bulk/api.py` `run_deg`)

**Returns**: a DataFrame with per-celltype run status and metadata. (Source: `edger_batch1.py` `run_edger_batch`)

### `stream_bulk.run_stream`

**Signature**:

```python
def run_stream(
    *,
    pseudobulk_dir: Optional[str] = None,
    output_dir: Optional[str] = None,
    gsva: bool = True,
    ai: bool = True,
    deep_research: bool = True,
    wgcna: bool = True,
    layer: Optional[str] = None,
    model: Optional[str] = None,
) -> None:
```

**Behavior**: Sets environment overrides, constructs the STREAM UI via `stream_app.build_stream_ui`, finds key widgets, and triggers the analysis button. (Source: `stream_bulk/api.py` `run_stream`)

**Raises**:
- `ValueError` if `pseudobulk_dir` is not provided and `PSEUDOBULK_DIR` is unset. (Source: `stream_bulk/api.py` `run_stream`)
- `RuntimeError` if `ipywidgets` is missing or the Run button cannot be located. (Source: `stream_bulk/api.py` `run_stream`)

## Module: `stream_bulk.cli`

**Purpose**: CLI entrypoint for `stream-bulk`. (Source: `stream_bulk/cli.py`)

### `stream_bulk.cli.main`

**Signature**:

```python
def main(argv: list[str] | None = None) -> None:
```

**Behavior**: Parses CLI flags and calls `stream_bulk.run_stream`. (Source: `stream_bulk/cli.py` `main`)

## Module: `edger_batch1`

**Purpose**: Batch runner for EdgeR-style analysis and optional interactive UI. (Source: `edger_batch1.py` module docstring)

### `edger_batch1.run_edger_batch`

**Signature**:

```python
def run_edger_batch(
    folder: str = DEFAULT_FOLDER,
    rscript_path: str = DEFAULT_RSCRIPT,
    counts_pattern: str = DEFAULT_COUNTS_PATTERN,
    counts_suffix: Optional[str] = None,
    metadata_suffix: str = METADATA_SUFFIX,
    metadata_folder: Optional[str] = None,
    contrast_mode: str = "pairwise",
    interaction_groups: _InteractionGroups = None,
    confounder_cols: Optional[Sequence[str]] = None,
    sample_col: str = "Sample",
    group_col: str = "disease",
    donor_col: Optional[str] = None,
    relevel: Optional[str] = None,
    contrast_groups: _ContrastGroups = None,
    blocking_method: str = "design",
    output_dir: str = DEFAULT_OUTPUT_DIR,
    check_consistency: bool = True,
    require_both_groups: bool = True,
    fdr_thresh: float = 0.05,
    lfc_thresh_abs: float = 0.0,
    show_progress: bool = True,
    sva_enabled: bool = False,
) -> pd.DataFrame:
```

**Behavior**: Iterates over count files, validates metadata, runs `rscript_path` via subprocess, renames outputs, and returns a summary DataFrame. (Source: `edger_batch1.py` `run_edger_batch`)

**Returns**: DataFrame with columns including `celltype`, `counts`, `metadata`, `status`, `result_file`, `up_n`, `down_n`, `contrast_mode`, `contrast_selection`, `blocking_method`, `stdout`, `stderr`. (Source: `edger_batch1.py` `run_edger_batch`)

### `edger_batch1.launch_edger_batch_ui`

**Signature**:

```python
def launch_edger_batch_ui(
    folder: str = DEFAULT_FOLDER,
    rscript_path: str = DEFAULT_RSCRIPT,
    counts_pattern: str = DEFAULT_COUNTS_PATTERN,
    counts_suffix: Optional[str] = None,
    metadata_suffix: str = METADATA_SUFFIX,
    metadata_folder: Optional[str] = None,
    display_widget: bool = True,
):
```

**Behavior**: Builds an `ipywidgets` UI to configure and run `run_edger_batch` interactively. (Source: `edger_batch1.py` `launch_edger_batch_ui`)

### Constants

- `DEFAULT_FOLDER`, `DEFAULT_METADATA_FOLDER`, `COUNTS_SUFFIX`, `METADATA_SUFFIX`, `DEFAULT_COUNTS_PATTERN`, `DEFAULT_RSCRIPT`, `DEFAULT_OUTPUT_DIR`. (Source: `edger_batch1.py` `__all__`)

## Module: `stream_app`

**Purpose**: Builds and runs the STREAM analysis UI and dashboard pipeline. (Source: `stream_app.py` `build_stream_ui` docstring)

### `stream_app.build_stream_ui`

**Signature**:

```python
def build_stream_ui(adata_in: Optional[object] = None, *, display_widget=True, extra_globals: Optional[Mapping[str, object]] = None):
```

**Behavior**: Constructs the UI and performs pseudobulk inference when `adata_in` is None; the dashboard HTML is written when the Run analysis button is triggered. (Source: `stream_app.py` `build_stream_ui`)

## Module: `EdgeR_pipeline`

**Purpose**: CLI for EdgeR-style differential expression using inmoose. (Source: `EdgeR_pipeline.py` module docstring)

### `EdgeR_pipeline.main`

**Signature**:

```python
def main() -> None:
```

**Usage**:

```
python EdgeR_pipeline.py <counts_csv> <metadata_csv> <sample_column> <group_column> [donnor_column] [confounder1] [confounder2] ...
```

(Source: `EdgeR_pipeline.py` `main`)

## Module: `GSVA_limma_pipeline`

**Purpose**: CLI for limma contrasts on GSVA score matrices. (Source: `GSVA_limma_pipeline.py` module docstring)

### `GSVA_limma_pipeline.main`

**Signature**:

```python
def main() -> None:
```

**Usage**:

```
python GSVA_limma_pipeline.py <gsva_csv> <metadata_csv> <sample_column> <group_column> [donnor_column]
```

(Source: `GSVA_limma_pipeline.py` `main`)

## Module: `structured_llm_wrapper`

**Purpose**: Structured output helpers for OpenAI responses and schema generation. (Source: `structured_llm_wrapper.py` module docstring)

### `structured_llm_wrapper.format_harmonization_json_to_markdown`

**Signature**:

```python
def format_harmonization_json_to_markdown(payload: Dict[str, Any]) -> str:
```

(Source: `structured_llm_wrapper.py` `format_harmonization_json_to_markdown`)

### `structured_llm_wrapper.get_per_cell_json_schema`

**Signature**:

```python
def get_per_cell_json_schema(
    *,
    cell_key: str,
    payload_id: str,
    project_title: str,
    cell_type_label: str,
    tissue_type: str | None,
    contrast: str,
    species: str,
    gene_id_system: str,
    baseline_fraction: float | None,
    delta_fraction: float | None,
    donors_n: int | None,
    weights: str | None,
    fdr: float,
    min_log2fc: float,
    generated_at: str,
    prompt_checksum: str,
) -> Dict[str, Any]:
```

(Source: `structured_llm_wrapper.py` `get_per_cell_json_schema`)

### `structured_llm_wrapper.get_harmonization_json_schema`

**Signature**:

```python
def get_harmonization_json_schema(
    *,
    cell_type_keys: list[str],
    run_id: str,
    date: str,
    species: str,
    fdr: float,
    min_log2fc: float,
    bg_rule: str,
    weights: str,
) -> Dict[str, Any]:
```

(Source: `structured_llm_wrapper.py` `get_harmonization_json_schema`)

### `structured_llm_wrapper.get_consensus_json_schema`

**Signature**:

```python
def get_consensus_json_schema(
    *,
    run_hash: str,
    date: str,
    species: str,
    fdr: float,
    min_log2fc: float,
    bg_rule: str,
    weights: str,
    cell_type_keys: list[str],
) -> Dict[str, Any]:
```

(Source: `structured_llm_wrapper.py` `get_consensus_json_schema`)

### `structured_llm_wrapper.create_per_cell_response_with_structured_output`

**Signature**:

```python
def create_per_cell_response_with_structured_output(
    llm_client,
    model: str,
    prompt_text: str,
    max_tokens: int,
    *,
    cell_key: str,
    payload_id: str,
    project_title: str,
    cell_type_label: str,
    tissue_type: str | None,
    contrast: str,
    species: str,
    gene_id_system: str,
    baseline_fraction: float | None,
    delta_fraction: float | None,
    donors_n: int | None,
    weights: str | None,
    fdr: float,
    min_log2fc: float,
    generated_at: str,
    prompt_checksum: str,
    use_structured_output: bool = True,
) -> Tuple[Optional[str], Optional[Dict[str, Any]]]:
```

(Source: `structured_llm_wrapper.py` `create_per_cell_response_with_structured_output`)

### `structured_llm_wrapper.create_harmonization_response_with_structured_output`

**Signature**:

```python
def create_harmonization_response_with_structured_output(
    llm_client,
    model: str,
    prompt_text: str,
    max_tokens: int,
    *,
    cell_type_keys: list[str],
    run_id: str,
    date: str,
    species: str,
    fdr: float,
    min_log2fc: float,
    bg_rule: str,
    weights: str,
    use_structured_output: bool = True,
) -> Tuple[Optional[str], Optional[Dict[str, Any]]]:
```

(Source: `structured_llm_wrapper.py` `create_harmonization_response_with_structured_output`)

## Module: `concat_harmonized`

**Purpose**: Flatten harmonized JSON/Markdown outputs into tables. (Source: `concat_harmonized.py` module docstring)

### `concat_harmonized.concat_payloads`

**Signature**:

```python
def concat_payloads(
    payloads: List[Tuple[Dict[str, Any], str]],
    *,
    markdowns: Optional[List[Tuple[str, str]]] = None,
    outdir: Optional[str] = None,
    filter_significant: bool = False,
    q_genes: Optional[float] = None,
    q_terms: Optional[float] = None,
    q_tfs: Optional[float] = None,
    q_edges: Optional[float] = None,
) -> Dict[str, Any]:
```

(Source: `concat_harmonized.py` `concat_payloads`)

### `concat_harmonized.main`

**Signature**:

```python
def main(argv: Optional[List[str]] = None) -> int:
```

(Source: `concat_harmonized.py` `main`)
