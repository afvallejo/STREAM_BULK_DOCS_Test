# Testing

## Run tests

```bash
python test_structured_outputs.py
python test_compression.py
```

(Source: `test_structured_outputs.py`, `test_compression.py`)

## Test notes

- Tests are simple scripts, not a test framework invocation. (Source: test files)
- `test_structured_outputs.py` exercises structured output schema helpers. (Source: `test_structured_outputs.py`)
- `test_compression.py` validates gzip+base64 encoding used in the dashboard. (Source: `test_compression.py`)
