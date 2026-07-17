# Understanding Modal GPU Specifications

## Overview

Modal's GPU API has evolved from class-based specifications to string-based specifications. As of February 2025, the class-based API (`modal.gpu.A100()`, `modal.gpu.H100()`, etc.) is deprecated.

## Current API (String-Based)

Use simple strings to specify GPUs in Modal function decorators:

### Single GPU
```python
@app.function(gpu="A100-40GB")
@app.function(gpu="A100-80GB")
@app.function(gpu="H100")
@app.function(gpu="A10G")
@app.function(gpu="T4")
@app.function(gpu="L4")
```

### Multiple GPUs
Append `:<count>` to specify multiple GPUs:
```python
@app.function(gpu="A100-40GB:2")  # 2x A100-40GB
@app.function(gpu="H100:4")       # 4x H100
@app.function(gpu="A10G:2")       # 2x A10G
```

## Deprecated API (Class-Based)

The following patterns trigger `DeprecationError` warnings:
```python
# DON'T USE - Deprecated
modal.gpu.A100(count=1)   # Use "A100-40GB" instead
modal.gpu.A100(count=2)   # Use "A100-40GB:2" instead
modal.gpu.H100(count=1)   # Use "H100" instead
modal.gpu.H100(count=2)   # Use "H100:2" instead
modal.gpu.A10G(count=1)   # Use "A10G" instead
```

## Type Annotations

The old `modal.gpu._GPU` type no longer exists. When storing or returning GPU configurations, use `str`:

```python
# Old (broken)
def get_gpu_config(spec: str) -> modal.gpu._GPU:
    return modal.gpu.A100(count=1)

# New (correct)
def get_gpu_config(spec: str) -> str:
    return "A100-40GB"
```

## GPU Configuration Dictionaries

When creating mappings for GPU configurations:

```python
# Old (deprecated)
GPU_CONFIGS = {
    "A100-1": modal.gpu.A100(count=1),
    "A100-2": modal.gpu.A100(count=2),
    "H100-1": modal.gpu.H100(count=1),
}

# New (correct)
GPU_CONFIGS: dict[str, str] = {
    "A100-1": "A100-40GB",
    "A100-2": "A100-40GB:2",
    "H100-1": "H100",
}
```

## Common GPU Types

| GPU Type | String Specification | Notes |
|----------|---------------------|-------|
| A100 40GB | `"A100-40GB"` | Most common A100 variant |
| A100 80GB | `"A100-80GB"` | High-memory variant |
| H100 | `"H100"` | Latest generation |
| A10G | `"A10G"` | Good for inference |
| T4 | `"T4"` | Budget option |
| L4 | `"L4"` | Efficient for inference |

## Migration Checklist

When updating Modal code from the deprecated API:

1. Replace `modal.gpu.A100(count=N)` with `"A100-40GB:N"` (or `"A100-40GB"` if N=1)
2. Replace `modal.gpu.H100(count=N)` with `"H100:N"` (or `"H100"` if N=1)
3. Replace `modal.gpu.A10G(count=N)` with `"A10G:N"` (or `"A10G"` if N=1)
4. Update type annotations from `modal.gpu._GPU` to `str`
5. Update any GPU config dictionaries to use string values
