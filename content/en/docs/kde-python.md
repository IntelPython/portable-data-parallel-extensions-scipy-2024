---
title: KDE Python extension
description: KDE (kernel density estimation) Python extension example.
date: 2024-07-02
weight: 3
---

Since SYCL builds on C++, we are going to use `pybind11` project to generate Python extension.
We also need Python objects to carry USM allocations of input and output data, such as `dpctl` ([Data Parallel Control](https://github.com/IntelPython/dpctl.git) Python package). The `dpctl` package also provides Python objects corresponding to DPC++ runtime objects:

| Python object         | SYCL C++ object   |
| --------------------- | ----------------- |
| ``dpctl.SyclQueue``   | ``sycl::queue``   |
| ``dpctl.SyclDevice``  | ``sycl::device``  |
| ``dpctl.SyclContext`` | ``sycl::context`` |
| ``dpctl.SyclEvent``   | ``sycl::event``   |

`dpctl` provides integration with `pybind11` supporting castings between `dpctl` Python objects and corresponding C++ SYCL classes listed in the table above. Furthermore, the integration provides C++ class ``dpctl::tensor::usm_ndarray`` which derives from ``pybind11::object``.
It stores `dpctl.tensor.usm_ndarray` object and provides methods to query its attributes, such as data pointer, dimensionality, shape, strides
and elemental type information.

For illustration purpose, here is a sample extension source code:

```cpp
#include <pybind11/pybind11.h>
#include <pybind11/stl.h>
#include <sycl/sycl.hpp>
#include "dpctl4pybind11.hpp"
#include <vector>

sycl::event
py_foo(dpctl::tensor::usm_ndarray inp, dpctl::tensor::usm_ndarray out, const std::vector<sycl::event> &deps) {
    // validation steps skipped

    // Execution queue is the queue associated with input arrays
    // these are expected to be the same and checked during validation
    sycl::queue exec_q = inp.get_queue();

    const std::int64_t *inp_ptr = inp.get_data<std::int64_t>();
    std::int64_t *out_ptr = out.get_data<std::int64_t>();

    // submit tasks for execution and obtain the event signaling
    // the status of execution of the set of tasks
    sycl::event e_impl =  impl_fn(exec_q, inp_ptr, out_ptr, depends);

    return e_impl;
}

PYBIND11_MODULE(_ext, m) {
    m.def("foo", &py_foo);
}
```

On the Python side, the function would be called as follows

```python
import dpctl.tensor as dpt
import _ext

# Allocate input and output arrays on the
# default-selected device
inp = dpt.arange(100, dtype=dpt.int64)
out = dpt.empty_like(inp)
ev = _ext.foo(inp, out, [])

# ...
```

Since execution is offloaded to a device, it is our responsibility to ensure that USM data being worked on
is not deallocated until after offloaded tasks complete the execution. The simplest way to assure of this is
to wait on the event with `ev.wait()` on the Python side, or with `e_impl.wait()` on the C++ side.

Alternatively, one can assure the lifetime of arrays asynchronously, by using `sycl::handler::host_task` to
schedule code execution on the host ordering it after kernel execution completion:

```cpp
// increment reference counts of input and output arrays

sycl::event ht_ev =
    exec_q.submit([&](sycl::handler &cgh) {
        // execute host_task once e_impl signals completion
        cgh.depends_on(e_impl);

        cgh.host_task([=]() {
            // we must acquire GIL to be able to safely
            // manipulate reference counts of Python objects
            pybind11::gil_scoped_acquire guard;

            // decrement ref-counts
        });
    });
```

Since the host task may execute in a thread different from that of the Python interpreter (the main thread), care must be taken
to avoid deadlocks: synchronization operation that call `ht_ev.wait()` from the main thread must release GIL to afford the body
of the host task a chance at execution.

Of course, if USM memory is not managed by Python, it may be possible to avoid using GIL altogether.

An example of Python extension `"kde_sycl_ext"` that exposes kernel density estimation code from previous
section can be found in `"steps/sycl_python_extension"` folder (see [README](steps/sycl_python_extension/README.md)).

The folder contains comparison between `dpctl`-based implementation of the KDE implementation following the NumPy
implementation [above](#kde_numpy) and the dedicated C++ code:

```
KDE for n_sample = 1000000, n_est = 17, n_dim = 7, h = 0.05
Result agreed.
kde_dpctl took 0.3404452269896865 seconds
kde_ext[mode=0] 0.02209925901843235 seconds
kde_ext[mode=1] 0.02560457994695753 seconds
kde_ext[mode=2] 0.02815118699800223 seconds
kde_numpy 0.7227164240321144 seconds
```

This sample run was obtained on a laptop with 11th Gen Intel(R) Core(TM) i7-1185G7 CPU @ 3.00GHz, 32 GB of RAM, and the integrated Intel(R) Iris(R) Xe GPU, with stock NumPy 1.26.4, and development build of dpctl 0.17 built with oneAPI DPC++ 2024.1.0.
