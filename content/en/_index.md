---
title: Portable Data-Parallel Python Extensions with oneAPI
---

{{<blocks/cover title="" image_anchor="top" height="max" color="primary">}}
  <div class="mx-auto">
    <div class="col-lg-12">
      <p class="display-6 mt-5 mb-3" style="font-weight: bold;">Portable Data-Parallel Python Extensions with oneAPI</p>
      <p></p>
      <p class="display-7 pt-0 mb-3">
      <a href="https://github.com/ndgrigorian" style="color: #50aaf4; font-weight: bold;" target="_blank" rel="noopener">Nikita Grigorian</a> and
      <a href="https://github.com/oleksandr-pavlyk" style="color: #50aaf4; font-weight: bold;" target="_blank" rel="noopener">Oleksandr Pavlyk</a>
      </p>
      <p class="display-7 pl-auto mb-0" style="text-align: center">Sources and details for the Portable Data-Parallel Python Extensions with oneAPI poster at the SciPy 2024 conference. This poster presents ongoing work to enable writing, building, and implementing portable extensions for data-parallel computation in Python.
      </p>
      <p></p>
  </div>
    <div class="lead text-center">
      <div class="mx-auto mb-5">
        <a class="btn btn-lg btn-secondary me-3 mb-4" href="https://github.com/google/docsy-example">
          First<i class="fa-solid fa-question ms-2 "></i>
        </a>
        <a class="btn btn-lg btn-secondary me-3 mb-4" href="https://github.com/google/docsy-example">
          Demonstration<i class="fab fa-github ms-2 "></i>
        </a>
        <a class="btn btn-lg btn-secondary me-3 mb-4" href="https://github.com/google/docsy-example">
          About<i class="fa-solid fa-address-card ms-2 "></i>
        </a>
      </div>
    </div>
  </div>
{{< /blocks/cover >}}

{{% blocks/section color="secondary" %}}
<p class="display-6 mb-3" style="text-align: center; font-weight: bold;">About the authors</p>
<p class="text-center">
This poster was made by and is being presented on behalf of the <a href="https://github.com/IntelPython">Python team</a> at  <a href="http://www.intel.com">Intel</a>.
</p>
<div class="row justify-content-center mx-auto pt-2">
{{< cardpane >}}
  {{< card header=`<p style="font-weight: bold; text-align: center">dpctl</p>`
           footer=`<i class="fab fa-github ms-2 "></i> [Github](https://github.com/IntelPython/dpctl) <i class="fa fa-book ms-2"></i> [Docs](https://intelpython.github.io/dpctl)` >}}
  <p class="text-center">
  A package with Python bindings for DPC++ runtime classes and an <a href="https://data-apis.org/array-api/latest/API_specification/index.html">Array API Specification</a> conformant tensor submodule built on pure SYCL for portability.
  </p>
  {{< /card >}}
  {{< card header=`<p style="font-weight: bold; text-align: center;">numba-dpex</p>`
           footer=`<i class="fab fa-github ms-2 "></i> [Github](https://github.com/IntelPython/numba-dpex) <i class="fa fa-book ms-2"></i> [Docs](https://intelpython.github.io/numba-dpex/latest/index.html)` >}}
  <p class="text-center">
  An extension to the Numba JIT compiler which provides a SYCL-like API for kernel programming and an extension to Numba's parallelizer to generate and offload kernels to data-parallel devices.
  </p>
  {{< /card >}}

  {{< card header=`<p style="font-weight: bold; text-align: center;">dpnp</p>`
           footer=`<i class="fab fa-github ms-2"></i> [Github](https://github.com/IntelPython/dpnp) <i class="fa fa-book ms-2"></i> [Docs](https://intelpython.github.io/dpnp/)` >}}
  <p class="text-center">
  An oneAPI- and oneMKL-powered array library built as a drop-in replacement for Numpy that can be executed on any data-parallel device.
  </p>
  {{< /card >}}
{{< /cardpane >}}
</div>

{.h1 .text-center}
{{% /blocks/section %}}
