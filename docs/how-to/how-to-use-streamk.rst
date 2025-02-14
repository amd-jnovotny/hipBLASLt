.. meta::
   :description: How to use Stream-K with the hipBLASLt library for GEMM operations
   :keywords: hipBLASLt, ROCm, library, API, tool

.. _streamk:

********************************
Using Stream-K with hipBLASLt
********************************

hipBLASLt supports the Stream-K library, which reduces library sizes for a wide range of General Matrix-Matrix Multiplication (GEMM) shapes and sizes.
It also provides more consistent performance, which might be better in some cases.
Stream-K partitions an equal share of the aggregate inner-loop iterations among physical processing elements,
which provides a near-perfect utilization of computing resources.
For more information about Stream-K, see
`Stream-K: Work-centric Parallel Decomposition for Dense Matrix-Matrix Multiplication on the GPU <https://arxiv.org/abs/2301.03598>`_.

Configuring the kernel selection strategy
=========================================

The ``TENSILE_SOLUTION_SELECTION_METHOD`` environment variable controls the hipBLASLt kernel selection strategy for GEMM operations.
Set this variable to ``2`` to enable the Stream-K library or leave it set to ``0`` to continue to use the default settings.

*  ``TENSILE_SOLUTION_SELECTION_METHOD=0`` (Default)

   *  Kernels are selected from the standard tuned libraries.
   *  The heuristic best kernel is selected from the standard tuning grid.
   *  User-driven tuning (tunable ops) only accesses kernels from the standard grid and free-size libraries.
   *  This option does NOT use any Stream-K kernels.

*  ``TENSILE_SOLUTION_SELECTION_METHOD=2`` (Stream-K)

   *  This enables the optional Stream-K library to use a GEMM scheduling algorithm that results in consistently good peak GEMM performance with far fewer tuned kernels.
   *  The heuristic best kernel is selected from the Stream-K library.
   *  User-driven tuning (tunable ops) considers all kernels from the standard grid, the free-size library, and the Stream-K library.

Configuring the kernel selection strategy
=========================================

You can control The Stream-K kernel launch behavior using the environment variables listed in the following table.
By default, Stream-K uses a model to predict the optimal grid size to use when launching a GEMM kernel at runtime.
However, you can choose how many workgroups to launch a GEMM kernel with using the Stream-K settings below:

.. csv-table::
   :header: "Environment Variable","Description"
   :widths: 30, 100

   "``TENSILE_STREAMK_DYNAMIC_GRID``","Set this variable to ``3`` to use the default setting. With this setting, the program automatically picks the number of work groups to launch for optimal performance. Set this variable to ``0`` to disable the dynamic grid and always use all available compute units."
   "``TENSILE_STREAMK_FIXED_GRID``","This variable overrides the default grid size and launches Stream-K GEMM kernels using the specified number of workgroups."
   "``TENSILE_STREAMK_MAX_CUS``","This variable sets the maximum number of compute units to use for Stream-K kernels. By default, Stream-K kernels are allowed to use all compute units on the device, but this setting lets you limit the number of units that can be used."

Recommendations for using Stream-K
=========================================

Stream-K is especially advantageous in certain situations. Follow these guidelines when choosing a kernel selection strategy,
based on your application and the desired performance.

*  **Wide range of GEMM sizes**: Stream-K is a better choice for applications that handle a variety of GEMM shapes and sizes.
*  **Non-uniform dimensions**: Stream-K is particularly beneficial for GEMMs where one dimension is significantly larger than the others.
*  **Consistent performance**: Stream-K provides more consistent peak performance than the default selection
   method by evenly distributing work across the available compute units.

Managing Stream-K resource use
------------------------------

Follow these guidelines to optimize how Stream-K uses resources:

*  **Promoting concurrency**: Use ``TENSILE_STREAMK_FIXED_GRID`` to limit the number of workgroups. This prevents GEMM from monopolizing the
   GPU resources and allows other kernels to run concurrently.

   The following example limits the GEMM kernels to 64 workgroups:

   .. code-block:: bash
   
      export TENSILE_STREAMK_FIXED_GRID=64

*  **Limiting compute units**: Use ``TENSILE_STREAMK_MAX_CUS`` to restrict the number of compute units the Stream-K kernels can use.
  
   This example limits the GEMM kernels to 32 compute units:
      
   .. code-block:: bash

      export TENSILE_STREAMK_MAX_CUS=32