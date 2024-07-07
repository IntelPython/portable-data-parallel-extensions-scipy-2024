---
title: KDE DPC++ example
description: KDE (kernel density estimation) example using SYCL and DPC++.
date: 2024-07-02
weight: 2
---

Given a sample of \\(n\\) observations \\(x_i\\) drawn from an unknown underlying continuous distribution \\(f(x)\\),
the kernel density estimate of that density function is computed as follows, for some kernel
smoothing parameter \\(h \in \mathbb{R}\\):

$$
    \hat{f}(x) = \frac{1}{n} \sum_{i=1}^{n} \frac{1}{h} K\left(\frac{x - x_i}{h}\right)
$$

An example of NumPy code performing the estimation, for a common choice of kernel function as standard
\\(d\\)-dimensional Gaussian distribution:

<!-- See https://stackoverflow.com/questions/5319754/cross-reference-named-anchor-in-markdown //-->
<a id="kde_numpy" href=""></a>
```python
def kde(poi : np.ndarray, sample : np.ndarray, h : float) -> np.ndarray:
    """Given a sample from underlying continuous distribution and
    a smoothing parameter `h`, evaluate density estimate at each point of
    interest `poi`.
    """
    assert sample.ndim == 2
    assert poi.ndim == 2
    m, d1 = poi.shape
    n, d2 = sample.shape
    assert d1 == d2
    assert h > 0
    dm = np.sum(np.square(poi[:, np.newaxis, ...] - sample[np.newaxis, ...]), axis=-1)
    return np.mean(np.exp(dm/(-2*h*h)), axis=-1)/np.power(np.sqrt(2*np.pi) * h, d1)
```

The code above evaluates \\(f(x)\\) for \\(m\\) values of points of interest \\(y_t\\).

$$
   f(y_t) = \frac{1}{n} \sum_{i=1}^{n} \frac{1}{h} K\left( \frac{1}{h^2} \left\lVert y_t - x_i \right\rVert^{2}  \right), \;\;\;  \forall 0 \leq t \le m
$$

Evaluating such an expression can be done in parallel. Evaluation can be done independently for each \\(t\\).
Furthermore, summation over \\(i\\) can be partitioned among work-items, each summing \\(n_{wi}\\) distinct terms.
Such work partitioning would generate \\(m \cdot \left\lceil {n}/{n_{wi}}\right\rceil\\) independent tasks.
Each work-item could write its partial sum into a dedicated temporary memory location to avoid race condition
for further summation by another kernel operating in a similar fashion.

```cpp
    parallel_for(
        range<2>(m, ((n + n_wi - 1) / n_wi)),
        [=](sycl::item<2> it) {
            auto t = it.get_id(0);
            auto i_block = it.get_id(1);

            T local_partial_sum = ...;

            partial_sums[t * ((n + n_wi - 1) / n_wi) + i_block] = local_partial_sum;
        }
    );
```

Such an approach, known as tree reduction, is implemented in ``kernel_density_esimation_temps`` function found in
``"steps/kernel_density_estimation_cpp/kde.hpp"``.

Use of temporary allocation can be avoided if each work-item atomically adds the value of the local sum to the
appropriate zero-initialized location in the output array, as in implementation ``kernel_density_estimation_atomic_ref``
in the same header file:

```cpp
    parallel_for(
        range<2>(m, ((n + n_wi - 1) / n_wi)),
        [=](sycl::item<2> it) {
            auto t = it.get_id(0);
            auto i_block = it.get_id(1);

            T local_partial_sum = ...;

            sycl::atomic_ref<...> f_aref(f[t]);
            f_aref += local_partial_sum;
        }
    );
```

Multiple work-items may concurrently updating the same location in global memory would produce the correct result due to
use of ``sycl::atomic_ref`` but at the expense of increased number of attempts, phenomenon known as atomic pressure.
Atomic pressure leads to thread divergence and degrades performance.

To reduce the atomic pressure work-items can be organized into work-groups. Every work-item in a work-group has access
to local shared memory, dedicated on-chip memory, which can be used to cooperatively combine values held by work-items
in the work-group without accessing the global memory. This could be done efficiently by calling group function
``sycl::reduce_over_group``. To be able to call it, we must specify iteration range using ``sycl::nd_range`` rather than
``sycl::range`` as we did earlier.

```cpp
    auto wg = 256; // work-group-size
    auto n_data_per_wg = n_wi * wg;
    auto n_groups = ((n + n_data_per_wg - 1) / n_data_per_Wg);

    range<2> gRange(m, n_groups * wg);
    range<2> lRange(1, wg);

    parallel_for(
        nd_range<2>(gRange, lRange),
        [=](sycl::nd_item<2> it) {
            auto t = it.get_global_id(0);

            T local_partial_sum = ...;

            auto work_group = it.get_group();
            T sum_over_wg = sycl::reduce_over_group(work_group, local_sum, sycl::plus<>());

            if (work_group.leader()) {
                sycl::atomic_ref<...> f_aref(f[t]);
                f_aref += sum_over_wg;
            }
        }
    );
```

Complete implementation can be found in ``kernel_density_estimation_work_group_reduce_and_atomic_ref`` function
in ``"steps/kernel_density_estimation_cpp/kde.hpp"``.

These implementations are called from C++ application ``"steps/kernel_density_estimation_cpp/app.cpp"``, which
samples data uniformly distributed over unit cuboid, and estimates the density using Kernel Density Estimation
and spherically symmetric multivariate Gaussian probability density function as the kernel.

The application can be built using `CMake`, or `Meson`, please refer to [README](steps/kernel_density_estimation_cpp/README.md) document in that folder.
