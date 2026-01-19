# STREAM Bulk Tutorial

This tutorial mirrors the workflow in `Install/STREAM_Bulk_Tutorial.ipynb` and walks through a full STREAM Bulk run using a pseudobulk directory.

Expected inputs (update the paths to match your data):
- `.../STREAM/Pseudobulk/sample1_metadata.csv`
- `.../STREAM/Pseudobulk/sample1_pseudobulk_counts.csv`

## 1) Setup: paths and imports

Make sure `stream_bulk` is importable in this environment.

```python
from pathlib import Path
import pandas as pd

pseudobulk_dir = Path("PATH/TO/STREAM/Pseudobulk")
metadata_path = pseudobulk_dir / "sample1_metadata.csv"
counts_path = pseudobulk_dir / "sample1_pseudobulk_counts.csv"

assert metadata_path.exists(), f"Missing metadata: {metadata_path}"
assert counts_path.exists(), f"Missing counts: {counts_path}"
pseudobulk_dir
```

## 2) Inspect metadata

```python
metadata = pd.read_csv(metadata_path)
metadata.head()
```

Metadata columns in this example:
- `sample_name` (sample IDs)
- `groupA` (groups for contrasts)
- `donor`, `batch`, `sex`, `age` (possible blocking/confounders)

## 3) Inspect counts (first rows)

```python
counts_preview = pd.read_csv(counts_path, nrows=5)
counts_preview.head()
```

## 4) Sanity check: sample IDs align

```python
sample_col = "sample_name"
group_col = "groupA"
donor_col = "donor"
confounders = ["batch", "sex", "age"]

missing_in_counts = set(metadata[sample_col].astype(str)) - set(counts_preview.columns)
missing_in_counts
```

If `missing_in_counts` is empty, the metadata sample IDs match the count matrix columns.

## 5) Optional: filter low-expression genes

This uses EdgeR's CPM heuristic (via `stream_bulk.filter_data`).

```python
from stream_bulk import filter_data

counts_full = pd.read_csv(counts_path)
counts_full = counts_full.set_index(counts_full.columns[0])
counts_full = counts_full.loc[:, metadata[sample_col].astype(str)]

group_labels = metadata[group_col].astype(str).tolist()
filtered_counts, keep_mask = filter_data(counts_full, group_labels=group_labels)
filtered_counts.shape
```

## 6) Run EdgeR batch (optional if results already exist)

If you already have `sample1_*_edgeR_results.csv` in the pseudobulk folder, you can skip this step.

```python
from stream_bulk import run_deg

summary = run_deg(
    folder=str(pseudobulk_dir),
    output_dir=str(pseudobulk_dir),
    sample_col=sample_col,
    group_col=group_col,
    donor_col=donor_col,
    confounder_cols=confounders,
    contrast_mode="pairwise",
    contrast_groups=("control", "treatment"),
    relevel="control",
    show_progress=False,
)
summary
```

## 7) Run the dashboard pipeline

Set toggles as needed. This example disables AI and deep research to avoid OpenAI requirements.

### 7a) Configure GSVA limma to pairwise

This avoids the "interaction mode selected" warning by explicitly setting a pairwise contrast.

```python
from IPython import get_ipython

last_contrast_config = {
    "contrast_mode": "pairwise",
    "contrast_groups": ("control", "treatment"),
    "interaction_groups": None,
    "relevel": "control",
    "blocking_method": "design",
}

ipy = get_ipython()
if ipy is not None:
    ipy.user_ns["last_contrast_config"] = last_contrast_config

last_contrast_config
```

### 7b) Option A — run with the UI (interactive model selection)

This launches the widget UI so you can pick the LLM model from the dropdown.
Click **Run analysis** in the UI to execute the pipeline.

```python
import os
import stream_app

dashboard_dir = Path("STREAM/dashboards")
os.environ["PSEUDOBULK_DIR"] = str(pseudobulk_dir)
os.environ["STREAM_OUTPUT_DIR"] = str(dashboard_dir)

# Optional: set a default model for the dropdown.
# os.environ["OPENAI_MODEL"] = "gpt-4o"

ui = stream_app.build_stream_ui(adata_in=None, display_widget=True)
ui
```

Programmatically set deep research selections before clicking **Run analysis**:

```python
deep_research_cells = ["sample1"]  # update to match labels in the dropdown
import ipywidgets as widgets

deep_count = None
deep_checkbox = None
bio_checkbox = None
deep_dropdowns = []

def _iter_widgets(node):
    if isinstance(node, widgets.Widget):
        yield node
        for child in getattr(node, "children", ()) or ():
            yield from _iter_widgets(child)

for w in _iter_widgets(ui):
    desc = getattr(w, "description", "")
    if isinstance(w, widgets.Checkbox) and desc == "Generate biology insights":
        bio_checkbox = w
    elif isinstance(w, widgets.Checkbox) and desc == "Deep research":
        deep_checkbox = w
    elif isinstance(w, widgets.BoundedIntText) and desc == "Deep research cells:":
        deep_count = w
    elif isinstance(w, widgets.Dropdown) and desc.startswith("Cell "):
        deep_dropdowns.append(w)

if bio_checkbox is not None:
    bio_checkbox.value = True
if deep_checkbox is not None:
    deep_checkbox.value = True

if deep_count is not None:
    max_cells = getattr(deep_count, "max", len(deep_research_cells))
    deep_count.value = max(1, min(len(deep_research_cells), int(max_cells)))
    deep_dropdowns = [
        w for w in _iter_widgets(ui)
        if isinstance(w, widgets.Dropdown) and getattr(w, "description", "").startswith("Cell ")
    ]

lookup = {}
if deep_dropdowns:
    options = list(deep_dropdowns[0].options)
    for label, value in options:
        if value is None:
            continue
        label_text = str(label).strip()
        value_text = str(value).strip()
        if value_text:
            lookup[value_text.lower()] = value
        if label_text:
            lookup[label_text.lower()] = value
        if "[" in label_text and label_text.endswith("]"):
            base = label_text.rsplit("[", 1)[0].strip()
            slug = label_text[label_text.rfind("[") + 1 : -1].strip()
            if base:
                lookup[base.lower()] = value
            if slug:
                lookup[slug.lower()] = value

for idx, cell in enumerate(deep_research_cells):
    if idx >= len(deep_dropdowns):
        break
    key = str(cell).strip().lower()
    selected = lookup.get(key) or lookup.get(key.replace(" ", "_"))
    if selected is not None:
        deep_dropdowns[idx].value = selected
```

### 7c) Option B — headless run (set model programmatically)

Use this when you want to run without the UI.
Set `model_override` to a valid OpenAI model id to force it.

```python
from stream_bulk import run_stream
from IPython import get_ipython
import stream_app as _stream_app
import os

os.environ["OPENAI_API_KEY"] = "..."
dashboard_dir = Path("STREAM/dashboards")

# Clear any previous env overrides so arguments below take effect.
for _key in ("STREAM_GSVA_ENABLED", "STREAM_AI_ENABLED", "STREAM_DEEP_RESEARCH_ENABLED", "STREAM_WGCNA_ENABLED"):
    os.environ.pop(_key, None)

# Headless model override (set to a valid model id or keep None to use defaults).
model_override = "gpt-5-nano"  # e.g., "gpt-4o"
if model_override:
    os.environ["OPENAI_MODEL"] = model_override
else:
    os.environ.pop("OPENAI_MODEL", None)

last_contrast_config = {
    "contrast_mode": "pairwise",
    "contrast_groups": ("control", "treatment"),
    "interaction_groups": None,
    "relevel": "control",
    "blocking_method": "design",
}

ipy = get_ipython()
if ipy is not None:
    ipy.user_ns["last_contrast_config"] = last_contrast_config

# Ensure the GSVA limma cache uses pairwise settings for this run.
_stream_app._GSVA_LIMMA_CONFIG.update(last_contrast_config)

run_stream(
    pseudobulk_dir=str(pseudobulk_dir),
    output_dir=str(dashboard_dir),
    gsva=True,
    ai=True,
    deep_research=True,
    wgcna=True,
    model=model_override,
)
```

## 8) Locate the generated dashboard

```python
list(dashboard_dir.glob("*.html"))
```

## 9) Optional: run via CLI

If `stream-bulk` is installed in your environment, you can run the same pipeline from a shell:

```bash
stream-bulk --pseudobulk-dir "PATH/TO/STREAM/Pseudobulk" \
  --output-dir "STREAM/dashboards" \
  --no-ai --no-deep-research
```

## 10) Optional: enable AI summaries

To enable AI summaries and deep research, set `OPENAI_API_KEY` in your environment and run again with `ai=True`.

Example (do not paste real keys into shared notebooks):

```python
import os
os.environ["OPENAI_API_KEY"] = "..."
run_stream(
    pseudobulk_dir=str(pseudobulk_dir),
    output_dir=str(dashboard_dir),
    gsva=True,
    ai=True,
    deep_research=True,
    wgcna=False,
)
```
