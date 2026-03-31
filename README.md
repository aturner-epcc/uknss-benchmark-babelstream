# UK-NNSS BabelStream benchmark

**Note:** This benchmark/repository is closely based on the one used for the [NERSC-10 benchmarks](https://www.nersc.gov/systems/nersc-10/benchmarks/)

The BabelStream benchmark was developed at the University of Bristol to measure the achievable main memory bandwidth across variety of CPUs and GPUs using simple kernels. These kernels process data that is larger than the largest level of cache so that transfers from main memory are always in play. Dynamically allocated arrays are used to prevent any compile time optimisations. BabelStream provides implementations in multiple programming models for CPUs and GPUs. When used for GPUs, this benchmark does not include the data transfer time for CPU-GPU transfers.

## Software

- [BabelStream](https://github.com/UoB-HPC/BabelStream)

## Building the benchmark

**Important:** All results submitted should be based on the following version:

- [BabelStream v5.0](https://github.com/UoB-HPC/BabelStream/releases/tag/v5.0)

Any modifications made to the source code and build/installation files must be 
shared as part of the bidder submission under the same licence as the BabelStream
software.

## Permitted modifications

Offerors are permitted to modify the benchmark in the following ways.

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
-DOFFLOAD_FLAGS="-mp=gpu -gpu=cc90 -Minfo" \
../ 
```

</td><td>

```bash
cmake \
-DMODEL=cuda \
-DCMAKE_CXX_COMPILER=nvc++ \
-DCMAKE_CUDA_COMPILER=nvcc \
-DCUDA_ARCH=sm_90 \
../ 
```

</td></tr></table>

## Running the benchmark

### Required tests

The bidder is required to run the following tests

- CPU memory bandwidth:
  + Single node runs across all compute nodes.
  + All CPU cores must be running BabelStream in parallel via OpenMP threads
    or another parallel model implemented in BabelStream.
  + The sizes of the allocated arrays in BabelStream must be
    4x larger than the largest level of cache. This can be set at run time
    using the `--arraysize` option to BabelStream.
  + A minimum of 100 iterations (BabelStream default) must be used for the test.
  + The difference between the maximum measured per-node Triad memory
    bandwidth and the minimum measured per-node Triad memory bandwidth
    must be equal to or less than 5% of the mean measured per-node
    Triad memory bandwidth.

- GPU memory bandwidth:
  + Arrays should only be allocated on device's global memory,
    any pre-staging of data or use of user controlled cache is not allowed.
  + Must be run across all compute nodes.
  + Performance of all GPU/GCD on each node should be tested. The `--devices`
    option to BabelStream may be used to target specific GPU/GCD on a 
    node.
  + A minimum of 100 iterations (BabelStream default) must be used for the test.
  + The sizes of the allocated arrays in BabelStream must be
    4x larger than the largest level of cache. This can be set at run time
    using the `--arraysize` option to BabelStream .
  + The difference between the maximum measured per-GPU/GCD Triad memory
    bandwidth and the minimum measured per-GPU/GCD Triad memory bandwidth
    must be equal to or less than 5% of the mean measured per-GPU/GCD
    Triad memory bandwidth.

### Benchmark execution

The BabelStream executable, `<model>-stream`, can be found in the `build` directory.
The following arguments will typically be used to modify its runtime behaviour:

- `--arraysize SIZE` - the size of the arrays to use for the tests. The sizes
  of the allocated arrays in BabelStream must be 4x larger than the largest
  level of cache.
- `device INDEX` - the index of the accelerator device to use (for accelerator 
  memory tests). This option can be used to ensure all accelerator devices on 
  a node are tested.

We supply example job submission scripts from testing on the [IsambardAI](https://docs.isambard.ac.uk/specs/#system-specifications-isambard-ai-phase-2) system:

- [IsambardAI CPU bandwidth](run_cpu_isambardai.sh)
- [IsambardAI GPU bandwidth](run_cuda_isambardai.sh)

## Reporting results

The primary figure of merit (FoM) is the Triad rate (MB/s).

The bidder should provide:

- The minimum, maximum and mean Triad rate across all nodes/devices
  for the two test configurations
- Details of any changes made to the BabelStream source code
  and modifications to any build files (e.g. configure scripts, makefiles)
- Details of the build process for the BabelStream software 
  for both the host (CPU) and device (GPU) versions
- Details on how the tests were run, including any batch job submission
  scripts and options provided to BabelStream at runtime
- All data printed to STDOUT by the BabelStream software for 
  all runs of the benchmark on all nodes

## Example performance data

Example performance data from IsambardAI.

- [IsambardAI CPU memory performance](example_output/BabelStream-CPU-2345261.out)
- [IsambardAI GPU memory performance](example_output/BabelStream-CUDA-2344898.out)

## License

This benchmark description and associated files are released under the MIT license.
