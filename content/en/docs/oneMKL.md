---
title: oneMKL Python extension
description: A Python extension written using oneMKL interfaces.
date: 2024-07-02
weight: 4
---

Given a matrix \\(A\\), the QR decomposition of \\(A\\) is defined as the decomposition of \\(A\\) into the product of matrices \\(Q\\) and \\(R\\) such that \\(Q\\) is orthonormal and \\(R\\) is upper-triangular.

QR factorization is a common routine in more optimized LAPACK libraries, so rather than write and implement an algorithm ourselves, it would be preferable to find a suitable library routine.

Since `dpctl.tensor.usm_ndarray` is a Python object with an underlying USM allocation, it is possible to write extensions which wrap `oneAPI Math Kernel Library Interfaces` ([oneMKL Interfaces](https://github.com/oneapi-src/oneMKL)) USM routines and then call them on the `dpctl.tensor.usm_ndarray` from Python. These low-level routines can greatly improve the performance of an extension.

`oneMKL Interfaces` can be built to dispatch to a variety of backends including `cuBLAS` and `rocBLAS` (see [oneMKL Interfaces README](https://github.com/oneapi-src/oneMKL?tab=readme-ov-file#oneapi-math-kernel-library-onemkl-interfaces)). The [`portBLAS`](https://github.com/codeplaysoftware/portBLAS) backend is also notable as it is open-source and written in pure SYCL.

`oneMKL` routines are essentially wrappers for the same routine in an underlying backend library, depending on the targeted device. This means that the same code can be used for NVidia, AMD, and Intel devices, making it highly portable.  

Looking to the `oneMKL` documentation on [`geqrf`](https://spec.oneapi.io/versions/latest/elements/oneMKL/source/domains/lapack/geqrf.html#geqrf-usm-version):

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

This general format (``sycl::queue``, arguments, and a vector of ``sycl::event``s) is more or less the same throughout the `oneMKL` USM routines.

The `pybind11` castings discussed in the previous section enable us to write a simple wrapper function for this routine with ``dpctl::tensor::usm_ndarray`` inputs and outputs, so long as we take the same precautions to avoid deadlocks. As a result, we can write the extension in much the same way as the `"kde_sycl_ext"` extension in the previous chapter.

An example of a Python extension `"mkl_interface_ext"` that uses `oneMKL` calls to implement a QR decomposition can be found in [`"steps/mkl_interface"`](https://github.com/IntelPython/example-portable-data-parallel-extensions/tree/main/steps/mkl_interface) folder (see [README](https://github.com/IntelPython/example-portable-data-parallel-extensions/blob/main/steps/mkl_interface/README.md)).

The folder executes the tests found in [`"steps/mkl_interface/tests"`](https://github.com/IntelPython/example-portable-data-parallel-extensions/tree/main/steps/mkl_interface/tests) as well as running a larger benchmark which compares Numpy's `linalg.qr` (for reference) to the extension's implementation:

```bash
$ python run.py
Using device NVIDIA GeForce GT 1030
================================================= test session starts ==================================================
collected 8 items

tests/test_qr.py ........                                                                                        [100%]

================================================== 8 passed in 0.45s ===================================================
QR decomposition for matrix of size = (3000, 3000)
Result agreed.
qr took 0.016026005148887634 seconds
np.linalg.qr took 0.5165981948375702 seconds
```
