# Building Software

This page covers compiling and installing your own code: compilers, MPI
wrappers, CUDA, build systems, and interpreted languages.

!!! tip "Compile on a login node; run on compute nodes"
    Compiling is allowed on login nodes. Long builds that pin CPUs for a long
    time are better done in a short [interactive job](batch.md#interactive-jobs).
    Never launch the resulting **compute** run on the login node.

## Compilers

Load a compiler module first (versions from `module avail`):

=== "GNU"

    ```bash
    module load gcc/12
    gcc   -O3 -march=native -o app app.c
    g++   -O3 -std=c++17    -o app app.cpp
    gfortran -O3            -o app app.f90
    ```

=== "Intel oneAPI"

    ```bash
    module load intel        # or intel-oneapi / intel/2024, per `module avail`
    icx  -O3 -xHost -o app app.c        # classic: icc
    ifx  -O3 -xHost -o app app.f90      # classic: ifort
    # Intel MKL for BLAS/LAPACK/FFT:
    icx -O3 app.c -qmkl -o app
    ```

=== "NVIDIA HPC SDK"

    ```bash
    module load nvhpc        # if available
    nvc    -O3 -o app app.c
    nvfortran -O3 -o app app.f90
    nvc -acc -gpu=cc80 -o app app.c     # OpenACC offload example
    ```

Common optimisation flags: `-O3`, `-march=native`/`-xHost` (tune for the build
node's CPU), `-fopenmp` (GNU) / `-qopenmp` (Intel) for OpenMP threading.

!!! note "`-march=native` matches the build node"
    If you build on a login node and run on a compute node with a *different*
    CPU generation, `-march=native` may produce illegal instructions. When in
    doubt, build inside an interactive job on the target partition, or target a
    conservative common ISA.

## MPI

Load an MPI module, then use its compiler wrappers (they add the right include
and link flags automatically):

```bash
module load gcc/12 openmpi/4.1.5     # or intel + intel-mpi

mpicc   -O3 -o app_mpi app.c         # C
mpicxx  -O3 -o app_mpi app.cpp       # C++
mpif90  -O3 -o app_mpi app.f90       # Fortran

# Hybrid MPI + OpenMP
mpicc -O3 -fopenmp -o app_hybrid app.c
```

Run MPI programs **through SLURM** (`srun`), not with a bare `mpirun` on the
login node — see [Batch System](batch.md) and [Examples](examples.md).

```bash
srun ./app_mpi        # inside an allocation, srun launches ranks across nodes
```

## CUDA / GPU builds

On (or targeting) `gpu` nodes:

```bash
module load cuda            # e.g. cuda/12.x per `module avail`
nvcc -O3 -arch=sm_80 -o app_gpu app.cu     # choose -arch for the actual GPU

# MPI + CUDA
module load gcc/12 openmpi cuda
mpicxx -O3 app.cpp -lcudart -o app_mpi_gpu
```

Confirm the correct GPU architecture (`sm_XX`) for the installed GPUs:

```bash
# Inside a GPU job:
nvidia-smi --query-gpu=name,compute_cap --format=csv
```

See [GPU Computing](gpu.md) for running, MPS, and multi-GPU jobs.

## Build systems

=== "Make"

    ```bash
    module load gcc/12
    make -j $(nproc)          # parallel build using all cores on the node
    ```

=== "CMake"

    ```bash
    module load cmake gcc/12 openmpi
    cmake -S . -B build -DCMAKE_BUILD_TYPE=Release \
          -DCMAKE_INSTALL_PREFIX=$HOME/opt/myapp
    cmake --build build -j $(nproc)
    cmake --install build
    ```

Install into `$HOME/opt/...` (or `/scratch` for large builds) and add the result
to your `PATH`/`LD_LIBRARY_PATH`, or expose it via a
[personal module](modules.md#building-your-own-module-advanced).

## Python

Two good options — Conda (default on the system) or `venv`:

=== "Conda (recommended)"

    ```bash
    conda create -n proj python=3.11 numpy scipy mpi4py
    conda activate proj
    python -c "import numpy; print(numpy.__version__)"
    ```

=== "venv + pip"

    ```bash
    module load python 2>/dev/null || true   # or use the conda base python
    python -m venv $HOME/venvs/proj
    source $HOME/venvs/proj/bin/activate
    pip install --upgrade pip
    pip install numpy scipy pandas
    ```

For MPI-parallel Python, install `mpi4py` **against the cluster MPI**:

```bash
module load gcc/12 openmpi
pip install --no-binary=mpi4py mpi4py    # builds against the loaded MPI
```

!!! warning "Mind your `$HOME` quota"
    Python environments can grow to many GB. Check `du -sh ~/.conda ~/venvs` and
    place large environments under `/scratch` if needed (remember the purge
    policy).

## Julia and other languages

```bash
module avail 2>&1 | grep -i julia
module load julia        # if provided
julia -e 'using Pkg; Pkg.status()'
```

Set `JULIA_DEPOT_PATH` to a project location if you want packages outside
`$HOME`.

## Containers (if enabled)

If a container runtime such as **Apptainer/Singularity** is provided:

```bash
module avail 2>&1 | grep -i -E 'apptainer|singularity'
apptainer pull docker://ghcr.io/<org>/<image>:<tag>
srun apptainer exec app.sif ./run.sh     # run inside a job
```

Containers are a clean way to ship reproducible software stacks; confirm
availability and any registry/proxy requirements with [support](support.md).

Next: submit your build to the [Batch System](batch.md).
