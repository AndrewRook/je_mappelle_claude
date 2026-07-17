---
name: compiling_gpu_code_on_modal
description: Explains how to set up a Modal.com image for custom GPU operations where compiling may be necessary. In general good advice for running GPU enabled functions
on Modal.com.
---

The default image used by Modal's docs for running functions lacks Nvidia compiler
tools. This is ok for running simple stuff, but if you're going to be compiling
anything complicated you can quickly run into errors. Through trial and error I have
learned that the following image configuration has much higher chances of success:

```
cuda_version = "12.8.1"  # should be no greater than host CUDA version
flavor = "devel"  # includes full CUDA toolkit
operating_sys = "ubuntu24.04"
tag = f"{cuda_version}-{flavor}-{operating_sys}"

image = (
    modal.Image.from_registry(f"nvidia/cuda:{tag}", add_python="3.12")
    .entrypoint([])  # remove verbose logging by base image on entry
    .apt_install(  # install system libraries for graphics handling
        [
            "build-essential",
            "libgl1",
            "libglib2.0-0"
        ]
    )
)
```
