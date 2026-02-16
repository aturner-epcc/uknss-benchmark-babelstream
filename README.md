# UK-NSS BabelStream benchmark

**Note:** This benchmark/repository is closely based on the one used for the [NERSC-10 benchmarks](https://www.nersc.gov/systems/nersc-10/benchmarks/)

The BabelStream benchmark was developed at the University of Bristol to measure the achievable main memory bandwidth across variety of CPUs and GPUs using simple kernels. These kernels process data that is larger than the largest level of cache so that transfers from main memory are always in play. Dynamically allocated arrays are used to prevent any compile time optimisations. BabelStream provides implementations in multiple programming models for CPUs and GPUs. When used for GPUs, this benchmark does not include the data transfer time for CPU-GPU transfers.

## Status

Stable

## Maintainers

- @aturner-epcc ([https://github.com/aturner-epcc](https://github.com/aturner-epcc))

## Overview

### Software

- [BabelStream](https://github.com/UoB-HPC/BabelStream)

### Architectures

- CPU: x86, Arm
- GPU: NVIDIA, AMD, Intel

### Languages and programming models

- Programming languages: C++
- Parallel models: OpenMP
- Accelerator offload models: CUDA, HIP, OpenACC, Kokkos, SYCL, RAJA, OpenCL, TBB

## Building the benchmark

**Important:** All results submitted should be based on the following version:

- [BabelStream v?.?]()

Any modifications made to the source code and build/installation files must be 
shared as part of the offerer submission.

## Permitted modifications

Offerors are permitted to modify the benchmark in the following ways.

**Programming Pragmas**

- The Offeror may choose any of the programming models implemented in BabelStream.
- The Offeror may modify the programming (e.g. OpenMP, OpenACC) pragmas in the benchmark as required  to permit execution on the proposed system, provided: 
   - All modified sources and build scripts must be included in the RFP response.
   - Any modified code used for the response must continue to be a valid program (compliant to the standard being proposed in the Offeror's response).

**Memory Allocation**

- For accelerators, arrays should only be allocated on device's global memory, any pre-staging of data or use of user controlled cache is not allowed.
- The sizes of the allocated arrays must be 4x larger than the largest level of cache. Array sizes can be modified by changing the variable `ARRAY_SIZE` on `line 55` of `./src/main.cpp` in BabelStream benchmark source code. 

**Concurrency & Affinity**

- The Offeror may change the kernel launch configurations, type of memory management (e.g. CUDA managed memory, separate host and device pointers etc.).

### Manual build

The Babelstream source code can be obtained from:
https://github.com/UoB-HPC/BabelStream.git using:

```bash
git clone https://github.com/UoB-HPC/BabelStream.git .
```

Move into the repo directory and checkout v5.0:

```bash
cd BabelStream
git checkout v5.0
```

The series of commands to configure and build BabelStream is
```bash
mkdir build
cd build
cmake -DMODEL=<model> <CMAKE_OPTIONS> ../
make
```
where `<model>` should be substituted with one of
the programming models implemented in the current version of BabelStream<br>
( omp; ocl; std; std20; hip; cuda; kokkos;
  sycl; sycl2020; acc; raja; tbb; thrust )
  

Additional CMake variables may be needed for some programming models.
For example,
<table><tr><th> OpenMP </th><th> OpenMP-offload </th><th> CUDA </th><tr>
<tr><td>

```bash
cmake \
-DMODEL=omp \
../ 
```

</td><td>

```bash
cmake \
-DMODEL=omp \
-DCMAKE_CXX_COMPILER=nvc++ \
-DOFFLOAD=ON \
-DOFFLOAD_FLAGS="-mp=gpu -gpu=cc80 -Minfo" \
../ 
```

</td><td>

```bash
cmake \
-DMODEL=cuda \
-DCMAKE_CXX_COMPILER=nvc++ \
-DCMAKE_CUDA_COMPILER=nvcc \
-DCUDA_ARCH=sm_80 \
../ 
```

</td></tr></table>

## Running the benchmark

### Required tests

The bidder is required to run the following tests

- CPU memory bandwidth:
  + Single node runs across all compute nodes concurrently
  + All CPU cores must be running BabelStream in parallel via OpenMP threads
    or another parallel model implemented in BabelStream
  + The sizes of the allocated arrays in BabelStream must be
    4x larger than the largest level of cache (see below for how
    to set this value)
  + The difference between the maximum measured per-node Triad memory
    bandwidth and the minimum measured per-node Triad memory bandwidth
    must be equal to or less than 5% of the mean measured per-node
    Triad memory bandwidth

- GPU memory bandwidth:
  + Arrays should only be allocated on device's global memory,
  any pre-staging of data or use of user controlled cache is not allowed
  + Single GPU/GCD runs across all GPU/GCD  concurrently
  + The sizes of the allocated arrays in BabelStream must be
    4x larger than the largest level of cache (see below for how
    to set this value)
  + The difference between the maximum measured per-GPU/GCD Triad memory
    bandwidth and the minimum measured per-GPU/GCD Triad memory bandwidth
    must be equal to or less than 5% of the mean measured per-GPU/GCD
    Triad memory bandwidth

### Benchmark execution

The BabelStream executable, `<model>-stream`, can be found in the `build` directory
and can be run without additional arguments, for example:

```bash
#OpenMP execution on a CPU
> export OMP_NUM_THREADS=4
> ./omp-stream 

#OpenMP-Offload exection on a GPU
> ./omp-stream 
```

All the kernels are validated at the end of their execution;
no explicit validation test is needed.

We supply example job submission scripts:

- [IsambardAI CPU bandwidth]()
- [IsambardAI GPU bandwidth]()

## Reporting results


The primary figure of merit (FoM) is the Triad rate (MB/s).

The bidder should provide:

- Details of any changes made to the BabelStream source code
  and modifications to any build files (e.g. configure scripts, makefiles)
- Details of the build process for the BabelStream software 
  for both the host (CPU) and device (GPU) versions
- Details on how the tests were run, including any batch job submission
  scripts and options provided to BabelStream at runtime
- All data printed to STDOUT by the BabelStream software

## Example performance data

Example performance data from IsambardAI.

## License

This benchmark description and associated files are released under the MIT license.
