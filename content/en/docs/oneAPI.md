---
title: About oneAPI
linkTitle: About oneAPI
description: A brief overview of oneAPI and the programming model
date: 2024-07-02
weight: 1
---

The Unified Acceleration Foundation ([UXL](https://uxlfoundation.org/)) under the umbrella of Linux Foundation is driving an open standard accelerator software ecosystem that includes compilers and performance libraries. This software ecosystem standardizes programming of different types of accelerators, such as multi-core CPUs, GPUs, some FPGAs, etc. from different vendors.

Intel's oneAPI DPC++ compiler is an implementation of the [SYCL-2020 standard](https://registry.khronos.org/SYCL/specs/sycl-2020/html/sycl-2020.html) that is a part of UXL foundation's overall language design standardization for accelerator programming in C++. The compiler is being developed in https://github.com/intel/llvm, and supports offloading to Intel(R) GPUs, NVidia(R) GPUs, and AMD GPUs.

The oneAPI DPC++ compiler can be installed on Linux and Windows as part of [oneAPI Base Toolkit](https://www.intel.com/content/www/us/en/developer/tools/oneapi/base-toolkit.html) and requires a system compiler to be installed, such as GCC on Linux and MSVC on Windows. Since DPC++ builds
on the conventional toolchain, Python extensions built with DPC++ are compatible with the Python stack out of the box. DPC++ also requires that OpenCL and Level Zero drivers are installed. For NVidia(R) or AMD devices, appropriate drivers must be installed.

To offload to devices, host code and device code are compiled separately, then combined by the linker into a single fat binary.

Support for offloading to GPUs from vendors other than Intel can be enabled by additionally installing plugins from CodePlay:

- [oneAPI for NVidia(R) GPUs](https://developer.codeplay.com/products/oneapi/nvidia/home/)
- [oneAPI for AMD GPUs](https://developer.codeplay.com/products/oneapi/amd/home/)

Using these plugins and the appropriate compiler options, the fat binary for a DPC++ application will also include the necessary sections for offloading a kernel to NVidia(R) or AMD GPUs.

A list of devices available to DPC++ can be obtained using ``sycl-ls``.

```bash
sycl-ls
[opencl:acc:0] Intel(R) FPGA Emulation Platform for OpenCL(TM), Intel(R) FPGA Emulation Device OpenCL 1.2  [2024.17.3.0.08_160000]
[opencl:cpu:1] Intel(R) OpenCL, 12th Gen Intel(R) Core(TM) i9-12900 OpenCL 3.0 (Build 0) [2024.17.3.0.08_160000]
[ext_oneapi_cuda:gpu:0] NVIDIA CUDA BACKEND, NVIDIA GeForce GT 1030 6.1 [CUDA 12.2]
```

A list of SYCL projects can be found on https://sycl.tech/projects
