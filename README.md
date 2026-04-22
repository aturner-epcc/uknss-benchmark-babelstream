# UK-NNSS BabelStream benchmark

This repository contains information on the Babelstream benchmark for the
UK NNSS procurement.

> [!IMPORTANT]
> Please do not contact the benchmark or code maintainers directly with any questions. All questions must be submitted via the procurement response mechanism.

## Benchmark Overview
The BabelStream benchmark was developed at the University of Bristol to measure the achievable main memory bandwidth across variety of CPUs and GPUs using simple kernels. These kernels process data that is larger than the largest level of cache so that transfers from main memory are always in play. Dynamically allocated arrays are used to prevent any compile time optimisations. BabelStream provides implementations in multiple programming models for CPUs and GPUs. When used for GPUs, this benchmark does not include the data transfer time for CPU-GPU transfers.

## Software

Git repository: [BabelStream](https://github.com/UoB-HPC/BabelStream)

> [!CAUTION]
> All results submitted should be based on the following tag:
>- [BabelStream v5.0](https://github.com/UoB-HPC/BabelStream/releases/tag/v5.0)

> [!NOTE] 
> This benchmark/repository is closely based on the one used for the [NERSC-10 benchmarks](https://www.nersc.gov/systems/nersc-10/benchmarks/)


## Building the benchmark

Compiling the code involves the following steps:

1. Configure the build
   ```bash
   cmake -B build -S . -DMODEL=<model> <CMAKE_OPTIONS>
   ```
   where `<model>` should be substituted with one of the programming models
   implemented in the current version of BabelStream. Current options for `<model>` are:
   > omp; ocl; std; std20; hip; cuda; kokkos; sycl; sycl2020; acc; raja; 
     tbb; thrust
  
   Additional CMake variables may be needed for some programming models.
   For example,

   |  Configuration | Flags              |
   |----------------|--------------------|
   | OpenMP         | `-DMODEL=omp`        |
   | OpenMP-offload | `-DMODEL=omp -DCMAKE_CXX_COMPILER=nvc++ \` <br> `-DOFFLOAD=ON -DOFFLOAD_FLAGS="-mp=gpu -gpu=cc90 \` <br> `-Minfo"` |
   | CUDA           | `-DMODEL=cuda -DCMAKE_CXX_COMPILER=nvc++ \ <br> -DCMAKE_CUDA_COMPILER=nvcc -DCUDA_ARCH=sm_90` |

2. Perform the build
   ```bash
   cmake --build build
   ```


### Pre-approved code modifications

Bidders are permitted to modify the benchmark in the following ways.

**Programming Pragmas**

- The bidder may choose any of the programming models implemented in BabelStream.
- The bidder may modify the programming (e.g. OpenMP, OpenACC) pragmas in the benchmark as required  to permit execution on the proposed system, provided: 
   - All modified sources and build scripts must be made available under the same licence as the BabelStream software
   - Any modified code used for the response must continue to be a valid program (compliant to the standard being proposed in the bidder's response).

**Memory Allocation**

- For accelerators, arrays should only be allocated on device's global memory, any pre-staging of data or use of user controlled cache is not allowed.
- The sizes of the allocated arrays must be 4x larger than the largest level of cache. Array sizes can be modified by changing the variable `ARRAY_SIZE` on `line 55` of `./src/main.cpp` in BabelStream benchmark source code. 

**Concurrency & Affinity**

- The bidder may change the kernel launch configurations, type of memory management (e.g. CUDA managed memory, separate host and device pointers etc.).

Any modifications must be fully documented (e.g., as a pull request, diff or patch file)
and reported with the benchmark results.



## Running the benchmark

The BabelStream executable, `<model>-stream`, can be found in the `build` directory.
The following arguments will typically be used to modify its runtime behaviour:

- `--arraysize SIZE` - the size of the arrays to use for the tests. The sizes
  of the allocated arrays in BabelStream must be 4x larger than the largest
  level of cache.
- `device INDEX` - the index of the accelerator device to use (for accelerator 
  memory tests). This option can be used to ensure all accelerator devices on 
  a node are tested.

### Benchmark execution

The benchmark can be used to test both CPU and GPU memory bandwidth.

- CPU memory bandwidth:
  + All CPU cores must be running BabelStream in parallel via OpenMP threads
    or another parallel model implemented in BabelStream.
  + The size of the allocated arrays in BabelStream must be
    4x larger than the largest level of cache. This can be set at run time
    using the `--arraysize` option to BabelStream.
  + A minimum of 100 iterations (BabelStream default) must be used for the test.

- GPU memory bandwidth:
  + Arrays should only be allocated on device's global memory,
    any pre-staging of data or use of user controlled cache is not allowed.
  + Performance of all GPU/GCD on each node should be tested. The `--devices`
    option to BabelStream may be used to target specific GPU/GCD on a 
    node.
  + A minimum of 100 iterations (BabelStream default) must be used for the test.
  + The size of the allocated arrays in BabelStream must be
    4x larger than the largest level of cache. This can be set at run time
    using the `--arraysize` option to BabelStream .

Example job submission scripts from testing on the [IsambardAI](https://docs.isambard.ac.uk/specs/#system-specifications-isambard-ai-phase-2) system are available below:

- [IsambardAI CPU bandwidth](run_cpu_isambardai.sh)
- [IsambardAI GPU bandwidth](run_cuda_isambardai.sh)

## Results

### Reference data

Reference data from IsambardAI.

**CPU**
```
Init: 0.143378 s (=186051.867106 MBytes/sec)
Read: 0.485833 s (=54907.321568 MBytes/sec)
Function    MBytes/sec  Min (sec)   Max         Average     
Copy        1751638.946 0.01015     0.03203     0.01082     
Mul         1665577.965 0.01068     0.03715     0.01118     
Add         1703432.068 0.01566     0.01654     0.01612     
Triad       1730906.365 0.01541     0.03137     0.01633     
Dot         1577779.397 0.01127     0.01659     0.01161
```

**GPU**
```
Init: 0.353618 s (=75436.650631 MBytes/sec)
Read: 0.006928 s (=3850606.492995 MBytes/sec)
Function    MBytes/sec  Min (sec)   Max         Average     
Copy        3216728.170 0.00553     0.00553     0.00553     
Mul         3210095.526 0.00554     0.00555     0.00554     
Add         3528340.308 0.00756     0.00758     0.00757     
Triad       3534593.282 0.00755     0.00756     0.00755     
Dot         3731714.822 0.00477     0.00481     0.00478     
```

Full output for the above runs is available here:
- [IsambardAI CPU memory performance](example_output/BabelStream-CPU-2345261.out)
- [IsambardAI GPU memory performance](example_output/BabelStream-CUDA-2344898.out)

## License

This benchmark description and associated files are released under the MIT license.
