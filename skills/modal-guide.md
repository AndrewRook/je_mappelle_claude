# Modal Development Guide

Generic guide for developing with Modal across all projects.

## Image Configuration

### Including Local Source Code

Modal images must include project source code to run your modules. Use `add_local_dir()` with `copy=True`:

```python
image = (
    modal.Image.debian_slim(python_version="3.11")
    .apt_install(...)
    .pip_install(...)
    .env({"PYTHONPATH": "/root/app"})
    .workdir("/root/app")
    # add_local_dir MUST be last, or use copy=True
    .add_local_dir("src", remote_path="/root/app/src", copy=True)
    .add_local_dir("config", remote_path="/root/app/config", copy=True)
)
```

**Key points:**
- `add_local_dir()` copies local directories into the container image
- Set `PYTHONPATH` to include the directory containing your source
- **IMPORTANT:** `add_local_dir` must be the LAST step in the image build, OR use `copy=True`
- Without `copy=True`, Modal adds files at container startup (faster rebuilds but restricts build order)

### Common System Dependencies

For computer vision workloads:
```python
.apt_install(
    "ffmpeg",
    "libgl1-mesa-glx",
    "libglib2.0-0",
    "libsm6",
    "libxext6",
    "libxrender-dev",
)
```

## Function Decorators

### Functions Defined Inside Other Functions

When defining a Modal function inside another function (not at global scope), you MUST use `serialized=True`:

```python
def run_training(args):
    app = modal.App("my-app")

    @app.function(
        image=get_image(),
        gpu="A10G",
        serialized=True,  # Required for local scope functions
    )
    def my_modal_function():
        pass

    with app.run():
        my_modal_function.remote()
```

**Without `serialized=True`, you'll get:**
```
modal.exception.InvalidError:
The `@app.function` decorator must apply to functions in global scope,
unless `serialized=True` is set.
```

### GPU Configuration

Available GPU types (string-based API):
- `"A10G"` - Testing, inference (24GB, cheapest)
- `"A100-40GB"` - Standard training
- `"A100-80GB"` - Large batch training
- `"H100"` - Fastest training

Multi-GPU format: `"A100-40GB:2"`, `"H100:4"`

Example:
```python
@app.function(
    image=get_image(),
    gpu="A100-40GB",
    volumes={"/vol/myvolume": get_volume()},
    timeout=86400,
    retries=0,  # Always set to 0 unless specified otherwise
)
def train():
    pass
```

## Volumes

### Creating and Accessing Volumes

```python
def get_volume() -> modal.Volume:
    return modal.Volume.from_name("myvolume", create_if_missing=False)
```

### Volume Mount Pattern

```python
@app.function(
    volumes={"/vol/myvolume": get_volume()},
)
def my_function():
    # Access files at /vol/myvolume/...
    pass
```

**Important:** Volume mounts in `@app.function` decorators are evaluated at import time, so dynamic volume names passed via CLI arguments are NOT supported. The volume name must be known when the module is imported.

### Volume Setup (User Action Required)

If a volume doesn't exist, ask the user:
1. What is the correct volume name? (They may have created it with a different name)
2. If no volume exists, they should create it: `modal volume create <volume_name>`

Do NOT automatically create volumes - the user should control their Modal resources.

## Common Errors and Solutions

### 1. Decorator Scope Error

**Error:**
```
The `@app.function` decorator must apply to functions in global scope,
unless `serialized=True` is set.
```

**Solution:** Add `serialized=True` to the decorator.

### 2. Volume.lookup Does Not Exist

**Error:**
```
AttributeError: type object 'Volume' has no attribute 'lookup'
```

**Solution:** Use `modal.Volume.from_name("volume_name", create_if_missing=True)` instead. The `lookup` method does not exist in the Modal API.

### 3. modal.gpu.GPU Type Annotation Error

**Error:**
```
AttributeError: module 'modal.gpu' has no attribute 'GPU'
```

**Solution:** Do NOT use `modal.gpu.GPU` as a type annotation - this base class does not exist. Either omit the return type annotation or use a generic type. GPU classes are accessed directly: `modal.gpu.A10G`, `modal.gpu.A100`, `modal.gpu.H100`, `modal.gpu.T4`.

### 4. Missing Module Error

**Error:**
```
ModuleNotFoundError: No module named 'mymodule'
```

**Solution:** Add local directories to the image with `add_local_dir(..., copy=True)` and set `PYTHONPATH`.

### 5. Build Step After Local Files Error

**Error:**
```
An image tried to run a build step after using `image.add_local_*` to include local files.
```

**Solution:** Either:
- Move `add_local_dir()` to be the last step in the image build
- Add `copy=True` to `add_local_dir()` calls

### 6. Volume Not Found

**Error:** `Volume 'volume_name' not found` or `does not exist`

**Solution:** Ask the user for the correct volume name or have them create it with `modal volume create <name>`.

### 7. Authentication Error

**Error:** Messages containing "auth" or "token"

**Solution:** Run `modal setup` to authenticate.

## Local Entrypoints and CLI Arguments

Modal's `@app.local_entrypoint()` converts Python function parameters to CLI arguments. **All arguments require the `--param-name` flag syntax**, even parameters without defaults that would normally be positional in argparse.

```python
@app.local_entrypoint()
def main(
    input_path: str,           # Required, but still needs --input-path
    output: str = "out.mp4",   # Optional with default
    verbose: bool = False,     # Boolean flag
):
    pass
```

**Usage:**
```bash
# Correct - all args use --flag syntax
modal run script.py --input-path video.mp4 --output result.mp4

# Wrong - positional args don't work
modal run script.py video.mp4  # Error: unexpected extra argument
```

**With `--detach` for background execution:**
```bash
modal run --detach script.py --input-path video.mp4
```

## Best Practices

1. **Always use `serialized=True`** when defining Modal functions inside other functions
2. **Use `copy=True`** with `add_local_dir()` to avoid build order issues
3. **Check all Modal files** when debugging - errors might come from a different file than expected
4. **Test with A10G first** - cheapest GPU for validation
5. **Set appropriate timeouts** - default is often too short for training (use 86400 for 24h)
6. **Always set `retries=0`** unless explicitly requested otherwise - failed GPU jobs should not auto-retry
7. **Use volumes for persistence** - checkpoints, logs, pretrained models
8. **Centralize Modal config** - keep image, volume, and GPU configs in one file
