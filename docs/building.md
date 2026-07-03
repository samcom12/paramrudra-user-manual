# Building Software

This page covers compiling your own code on PARAM Rudra: compilers, MPI, CUDA,
MKL and OpenACC. Toolchains are provided through **[Spack](spack.md)**.

!!! tip "Compile on the login node; run on compute nodes"
    *Compilation is done on the login node; execution happens on compute nodes
    via SLURM.* Long or heavy builds are better done in a short
    [interactive job](batch.md#interactive-jobs). Compile and run with the **same
    compiler and library versions** to avoid surprises.

## Available compilers

| Toolchain | Command(s) | Versions available |
| --- | --- | --- |
| GNU | `gcc` / `g++` / `gfortran` | 8.5.0, 9.5.0, 10.5.0, 11.5.0, 12.5.0, 13.4.0, 14.3.0, 15.2.0 |
| Intel oneAPI | `icx` / `icpx` / `ifx` (classic: `icc`/`ifort`) | 2022.2.0, 2024.2.1 |
| Intel MPI + GNU | `mpicc` / `mpicxx` / `mpif90` | 2021.18.0 |
| Intel MPI + Intel | `mpiicx` / `mpiicpx` / `mpiifx` | 2021.18.0 |
| CUDA | `nvcc` | 11.8.0, 12.2.2, 13.1.1 |
| NVIDIA HPC SDK | `nvc` / `nvfortran` (OpenACC) | via `nvhpc` |

Confirm exact builds/hashes with `spack find -l gcc`, `spack find -l intel-oneapi-compilers`, etc.

!!! tip "Prefer Intel compilers on this hardware"
    Intel compilers are generally **better optimized** for the Xeon Cascade Lake
    CPUs. Recommended optimization flags:
    **Intel:** `-O3 -xHost`  ·  **GNU:** `-O3`.

## Set up Spack first

```bash
module load spack
. /home/apps/spack/share/spack/setup-env.sh
```

## C / C++ / Fortran

=== "Intel (recommended)"

    ```bash
    spack load gcc@13.4.0
    spack load intel-oneapi-compilers /6asbh6t

    icx  -O3 -xHost prog.c        -o app     # C
    icpx -O3 -xHost prog.cpp      -o app     # C++
    ifx  -O3 -xHost prog.f90      -o app     # Fortran
    ./app
    ```

=== "GNU"

    ```bash
    spack load gcc@13.4.0
    gcc      -O3 prog.c   -o app
    g++      -O3 -std=c++17 prog.cpp -o app
    gfortran -O3 prog.f90 -o app
    ```

## OpenMP (shared memory)

```bash
spack load gcc@13.4.0
spack load intel-oneapi-compilers /6asbh6t

icx -O3 -xHost -qopenmp prog.c -o app_omp   # Intel: -qopenmp  (GNU: -fopenmp)
export OMP_NUM_THREADS=48                    # up to 48 physical cores/node
./app_omp
```

## MPI

```bash
spack load gcc@13.4.0
spack load intel-oneapi-compilers@2024.2.1
spack load intel-oneapi-mpi@2021.18.0

mpiicx  -O3 -xHost prog.c   -o app_mpi       # Intel MPI + Intel compiler
# or: mpicc -O3 prog.c -o app_mpi            # Intel MPI + GNU

# Launch through SLURM (preferred), or Intel's launcher:
srun --mpi=auto -n $SLURM_NTASKS ./app_mpi
# mpirun -n <num_procs> ./app_mpi
```

Useful Intel MPI environment settings seen in production scripts:

```bash
export I_MPI_FALLBACK=disable
export I_MPI_FABRICS=shm:ofa      # InfiniBand; (older: shm:dapl / shm:tmi)
export I_MPI_DEBUG=5              # verbosity (raise for diagnostics)
```

## Intel MKL

```bash
spack load gcc@13.4.0
spack load intel-oneapi-compilers@2024.2.1
spack load intel-oneapi-mkl@2026.0.0

icx -O3 -xHost -mkl prog.c -o app_mkl        # link MKL BLAS/LAPACK/FFT
./app_mkl
```

## CUDA (GPU) builds

The A100 GPUs are **Ampere → `sm_80`**:

```bash
spack load gcc@13.4.0
spack load cuda@12.8.0                        # or 11.8.0 / 12.2.2 / 13.1.1

nvcc -arch=sm_80 prog.cu -o app_gpu           # A100 = sm_80
./app_gpu
```

CUDA + OpenMP, linking cuBLAS:

```bash
spack load gcc@13.4.0
spack load cuda@12.2.2
nvcc -arch=sm_80 -Xcompiler "-fopenmp" -lgomp mm_blas_omp.cu -lcublas -o app
```

!!! note "`-arch` must match the GPU and CUDA/GCC versions"
    `sm_80` targets the A100 and is valid for CUDA 12+. Older CUDA releases only
    work with older GCC — load a matching `gcc` module if you use an older CUDA.

## OpenACC (NVIDIA HPC SDK)

```bash
spack load nvhpc@23.11
spack load cuda@12.2.2

# GPU offload (A100 = cc80):
nvc -acc -fast -Minfo=all -gpu=cc80,managed prog.c -o app_acc
# CPU multicore fallback:
nvc -acc -fast -Minfo=all -mp=multicore prog.c -o app_cpu
./app_acc
```

## Sample programs to start from

C-DAC ships worked examples. Copy them and follow the compile/run notes in each
file's header:

```bash
cp -r /home/apps/Docs/samples/ ~/
```

| File | Content |
| --- | --- |
| `mm.c` | Serial matrix–matrix multiply |
| `mm_omp.c` | OpenMP version |
| `mm_mpi.c` | MPI version |
| `mm_acc.c` | OpenACC version |
| `mm_blas.cu` | CUDA + cuBLAS |
| `mm_mkl.c` | Intel MKL |
| `laplace_acc.c` | OpenACC stencil |

## Build systems

```bash
# Make
spack load gcc@13.4.0
make -j $(nproc)

# CMake (load cmake via spack/module if needed)
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=$HOME/opt/myapp
cmake --build build -j $(nproc)
cmake --install build
```

Next: submit your build to the [Batch System](batch.md), or see
[Job Script Examples](examples.md).
