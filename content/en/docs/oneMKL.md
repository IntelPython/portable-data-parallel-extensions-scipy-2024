---
title: oneMKL Python extension
description: A Python extension written using oneMKL interfaces.
date: 2024-07-02
weight: 4
---

Since `dpctl.tensor.usm_ndarray` is a Python object carrying a USM allocation, it is possible to write extensions which wrap `oneAPI Math Kernel Library Interfaces` ([oneMKL Interfaces](https://github.com/oneapi-src/oneMKL)) routines and then call them on the USM data underlying the `usm_ndarray` container from Python.

For an example routine from the `oneMKL` documentation, take [`geqrf`](https://spec.oneapi.io/versions/latest/elements/oneMKL/source/domains/lapack/geqrf.html#geqrf-usm-version):
```cpp
namespace oneapi::mkl::lapack {
  cl::sycl::event geqrf(cl::sycl::queue &queue,
                        std::int64_t m,
                        std::int64_t n,
                        T *a,
                        std::int64_t lda,
                        T *tau,
                        T *scratchpad,
                        std::int64_t scratchpad_size,
                        const std::vector<cl::sycl::event> &events = {})
}
```

The `pybind11` castings discussed in the previous section enable us to write a simple wrapper function for this routine with `dpctl::tensor::usm_ndarray` inputs and outputs, so long as we take the same precautions to avoid deadlocks. As a result, we can write the extension in much the same way as the `kde_sycl_ext` extension in the previous chapter.

An example of a Python extension "mkl_interface_ext" that uses `oneMKL` calls to implement a QR decomposition can be found in "steps/mkl_interface" folder (see [README](steps/mkl_interface/README.md)).
