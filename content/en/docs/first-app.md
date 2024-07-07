---
title: First DPC++ app
description: A SYCL and DPC++ "Hello, World!" example.
date: 2024-07-02
weight: 2
---

For an in-depth introduction to SYCL and to accelerators programming please refer to the "[Data Parallel C++](https://link.springer.com/book/10.1007/978-1-4842-9691-2)" open access e-book.

A SYCL application runs on SYCL platform (host, connected to one or more heterogeneous devices). The application is structured in three scopes: application scope, command group scope, and kernel scope. The kernel scope specifies a single kernel function that will be compiled by the device
compiler and executed on the device. The command group scope specifies a unit work which includes the kernel function, preparation of
its arguments and specifying execution ordering information. The application scope specifies all the other code outside of command group scope.
Execution of SYCL application begins in the application scope.

```cpp
// Compile: icpx -fsycl first.cpp -o first
#include <sycl/sycl.hpp>

int main(void) {
    // queue to enqueue work to
    // default-selected device
    sycl::queue q{sycl::default_selector_v};

    // allocation device
    size_t data_size = 256;
    int *data = sycl::malloc_device<int>(data_size, q);

    // submit a task to populate
    // device allocation
    sycl::event e_fill =
        q.fill<int>(data, 42, data_size);   // built-in kernel

    // submit kernel to modify device allocation
    sycl::event e_comp =
        q.submit([&](sycl::handler &cgh) {   // command-group scope
            // order execution after
            // fill task completes
            cgh.depends_on(e_fill);

            sycl::range<1> global_iter_range{data_size};
            cgh.parallel_for(
                global_iter_range,
                [=](sycl::item<1> it) {      // kernel scope
                    int i = it.get_id(0);
                    data[i] += i;
                }
            );
        });

    // copy from device to host
    // order execution after modification task completes
    int *host_data = new int[data_size];

    q.copy<int>(                              // built-in kernel
        data, host_data, data_size, {e_comp}
    ).wait();
    sycl::free(data, q);

    // Output content of the array
    output_array(host_data, data_size);
    delete[] host_data;

    return 0;
}
```

The device where the kernel functions is executed is controlled by a device selector function, ``sycl::default_selector_v``.
The default selector assigns scores to every device recognized by the runtime, and selects the one with the highest score.
The list of devices recognized by the DPC++ runtime can be obtained by running ``sycl-ls`` command.

A user of SYCL application compiled with DPC++ may restrict the set of devices discoverable by the runtime using
``ONEAPI_DEVICE_SELECTOR`` environment variable. For example:

```bash
# execute on GPU
ONEAPI_DEVICE_SELECTOR=*:gpu ./first
# execute on CPU
ONEAPI_DEVICE_SELECTOR=*:cpu ./first
```

By default, DPC++ compiler generates offload code for [SPIR64](https://www.khronos.org/spir/) SYCL target, supported by
Intel GPUs as well as by CPU devices of x86_64 architecture. An attempt to execute SYCL program while
selecting only devices that do not support SPIR language would result in an error.

### Targeting other GPUs

DPC++ supports generation of offload sections for multiple targets. For example, to compile for both SPIR and NVPTX targets (oneAPI for NVidia(R) GPUs is assumed installed):

```bash
icpx -fsycl -Xsycl-targets="nvptx64-nvidia-cuda,spir64-unknown-unknown" first.cpp -o first.out
```

To compile for both SPIR and AMD GCN targets (oneAPI for AMD GPUs is assumed installed):

```bash
icpx -fsycl -Xsycl-targets="amdgcn-amd-amdhsa,spir64-unknown-unknown" first.cpp -o first.out
```

It is possible to pass additional arguments to the specific SYCL target backend. For example, to target specific architecture use:

- ``-Xsycl-target-backend=amdgcn-amd-amdhsa --offload-arch=gfx1030`` for AMD GPUs
- ``-Xsycl-target-backend=nvptx64-nvidia-cuda --cuda-gpu-arch=sm_80`` for NVidia GPUs
