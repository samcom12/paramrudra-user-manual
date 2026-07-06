# Best Practices & Performance

This page collects practical guidance for getting correct, efficient and
scalable results on PARAM Rudra, followed by focused sections on **compiler
optimization** (GCC, NVIDIA HPC SDK, Intel LLVM), **OpenMP**, **MPI**, and
**hybrid MPI+OpenMP**, each with a worked **case study**.

!!! info "Know your node"
    A PARAM Rudra compute node has **2× Intel Xeon Gold 6240R** (Cascade Lake) =
    **48 physical cores** across **2 NUMA domains** (24 cores each), with
    **AVX-512** vector units. GPU nodes add **2× NVIDIA A100** (`sm_80`). Nodes
    are linked by **InfiniBand NDR**. Tune with these numbers in mind.

## General HPC best practices

- **Never compute on the login node.** Login nodes are for editing, compiling
  and submitting only — jobs run through [SLURM](batch.md). Long processes there
  are terminated.
- **Run in `/scratch`, keep source in `/home`.** Stage inputs to
  `/scratch/$USER`, run there (fast Lustre I/O), then copy results worth keeping
  back to `/home` or off-cluster. `/scratch` is **not backed up** and is
  [purged after 3 months](data.md).
- **Compile and run with the *same* compiler and library versions.** Load
  identical [Spack](spack.md)/modules in your build and your job script. Mixing
  compilers (e.g. `gcc` + `icx`) across modules of one application invites link
  and runtime errors.
- **Right-size your request.** Ask only for the nodes, cores, memory and walltime
  you need — accurate limits let the backfill scheduler start you sooner and
  waste less allocation. Remember the **default walltime is 2 hours** if you
  don't set `--time`.
- **Choose appropriate optimization flags** (below) — and **verify accuracy**;
  aggressive/fast-math flags can change numerical results.
- **Use job arrays** for parameter sweeps instead of submitting many near-identical
  jobs one by one — better for throughput and the scheduler.
- **Capture output.** Keep `--output`/`--error` files; never send them to
  `/dev/null`. They are the first place to look on failure.
- **Estimate disk use** before launching many jobs; mind your 50 GB `/home` and
  200 GB `/scratch` quotas. Avoid spaces in file/directory names.
- **Install only from trusted sources**, and report anything strange (slowdowns,
  missing/corrupt files) to [support](support.md).

Beginner samples to start from live in `/home/apps/Docs/samples/` — copy with
`cp -r /home/apps/Docs/samples/ ~/`.

## Compiler optimization

PARAM Rudra offers three compiler families. Each can build the same code; pick
one toolchain per application and stay consistent.

| Family | C / C++ / Fortran | Load (Spack) |
| --- | --- | --- |
| **GCC** | `gcc` / `g++` / `gfortran` | `spack load gcc@13.4.0` |
| **Intel oneAPI (LLVM)** | `icx` / `icpx` / `ifx` | `spack load intel-oneapi-compilers` |
| **NVIDIA HPC SDK** | `nvc` / `nvc++` / `nvfortran` | `spack load nvhpc` |

!!! note "Intel classic vs Intel LLVM"
    The modern Intel compilers are **LLVM-based**: `icx` (C), `icpx` (C++),
    `ifx` (Fortran). The classic `icc`/`ifort` are deprecated — prefer
    `icx`/`ifx` for new work.

### Optimization-level cheat sheet

| Goal | GCC | Intel LLVM (`icx`/`ifx`) | NVIDIA (`nvc`/`nvfortran`) |
| --- | --- | --- | --- |
| Safe optimization | `-O2` | `-O2` | `-O2` |
| Aggressive | `-O3` | `-O3` | `-O3` |
| Max (may change FP results) | `-Ofast` | `-Ofast` / `-fp-model=fast` | `-fast` |
| **Tune for this CPU** (Cascade Lake) | `-march=cascadelake -mtune=cascadelake` | `-xCASCADELAKE` (or `-xHost`) | `-tp=cascadelake` (or `-tp=host`) |
| Interprocedural / LTO | `-flto` | `-ipo` | `-Mipa=fast` |
| OpenMP | `-fopenmp` | `-qopenmp` | `-mp` |
| Fast math | `-ffast-math` | `-fp-model=fast` | (in `-fast`) |
| Vectorization report | `-fopt-info-vec[-missed]` | `-qopt-report` or `-Rpass=loop-vectorize` | `-Minfo=vect` (or `-Minfo=all`) |

!!! tip "AVX-512 on Cascade Lake"
    These CPUs have AVX-512. GCC sometimes defaults to 256-bit vectors for
    frequency reasons — try `-mprefer-vector-width=512` and **measure** (AVX-512
    can down-clock; the win depends on the kernel). Intel `-xCASCADELAKE` and
    NVHPC `-tp=cascadelake` already target AVX-512.

!!! warning "`-march=native` / `-xHost` matches the *build* machine"
    If you compile on a login node and run on a compute node of a different CPU
    generation, native tuning can emit illegal instructions. On PARAM Rudra the
    login and compute CPUs are the same family, but for portable binaries prefer
    an explicit `-march=cascadelake` / `-xCASCADELAKE` / `-tp=cascadelake`.

### Recommended starting flags

=== "GCC"

    ```bash
    spack load gcc@13.4.0
    gcc -O3 -march=cascadelake -mtune=cascadelake -funroll-loops \
        -fopenmp -fopt-info-vec kernel.c -o kernel
    ```

=== "Intel LLVM"

    ```bash
    spack load intel-oneapi-compilers
    icx -O3 -xCASCADELAKE -qopenmp -qopt-report=3 kernel.c -o kernel
    # add -qmkl to link Intel MKL BLAS/LAPACK/FFT
    ```

=== "NVIDIA HPC SDK"

    ```bash
    spack load nvhpc
    nvc -O3 -tp=cascadelake -Minfo=all -mp kernel.c -o kernel
    # GPU offload: -acc -gpu=cc80  (A100) — see below
    ```

### Case study 1 — flag progression on a compute kernel

Take a hot numerical kernel (e.g. a dense matrix multiply or a stencil). Build it
at increasing optimization and **measure** each step so you keep only what helps:

```bash
for flags in "-O0" "-O2" "-O3" "-O3 -march=cascadelake" \
             "-O3 -march=cascadelake -funroll-loops" "-Ofast -march=cascadelake"; do
  gcc $flags kernel.c -o k && echo -n "$flags : " && ./k   # kernel prints its own timing
done
```

**What to observe / interpret** (representative, *illustrative* trend — always
measure your own):

| Flags | Typical effect |
| --- | --- |
| `-O0` | Baseline, no optimization (debug only) |
| `-O2` | Big first jump — inlining, basic vectorization |
| `-O3` | More aggressive loop transforms/vectorization |
| `+ -march=cascadelake` | Enables AVX-512 tuned codegen — often another step up |
| `+ -funroll-loops` | Helps some loops, neutral/negative for others — measure |
| `-Ofast` | Fastest, **but relaxes IEEE FP** — validate results |

Confirm vectorization actually happened:

```bash
gcc -O3 -march=cascadelake -fopt-info-vec-missed kernel.c -o k   # GCC
icx -O3 -xCASCADELAKE -qopt-report=3 kernel.c -o k               # Intel: *.optrpt
nvc -O3 -tp=cascadelake -Minfo=vect kernel.c -o k                # NVHPC
```

!!! danger "Always validate numerical accuracy after `-Ofast`/`-fast`"
    Fast-math reorders/relaxes floating-point operations. Compare output against
    an `-O2` (IEEE-safe) build on a known case before trusting production runs.

## OpenMP (shared memory, one node)

OpenMP parallelises across the cores of a **single node** using threads that
share memory. On PARAM Rudra you have up to **48 threads/node**.

### Compile & run

```bash
# Compile (pick your toolchain's OpenMP flag)
gcc  -O3 -march=cascadelake -fopenmp  app.c -o app     # GCC
icx  -O3 -xCASCADELAKE       -qopenmp  app.c -o app     # Intel LLVM
nvc  -O3 -tp=cascadelake     -mp       app.c -o app     # NVHPC
```

```bash
# In your job script
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=48        # threads = physical cores

export OMP_NUM_THREADS=${SLURM_CPUS_PER_TASK}
export OMP_PROC_BIND=close        # pin threads; 'close' packs them together
export OMP_PLACES=cores           # one thread per physical core
srun ./app
```

### Thread affinity matters

Binding threads to cores prevents the OS migrating them and keeps cache/NUMA
locality. Use `OMP_PLACES=cores` with `OMP_PROC_BIND=close` (compact) or `spread`
(spread across sockets — good for memory-bandwidth-bound codes). Inspect the
actual placement:

```bash
export OMP_DISPLAY_ENV=verbose     # prints the OpenMP runtime settings
export OMP_DISPLAY_AFFINITY=true   # prints where each thread landed
```

### Case study 2 — OpenMP thread scaling and binding

Measure how a memory-bandwidth-bound kernel (e.g. STREAM-like) and a
compute-bound kernel (e.g. DGEMM) scale with threads, and how binding changes it:

```bash
for t in 1 2 4 8 16 24 48; do
  OMP_NUM_THREADS=$t OMP_PROC_BIND=close OMP_PLACES=cores ./app
done
```

**Interpretation:**

- **Compute-bound** (DGEMM) usually scales close to linearly up to 48 cores with
  `close` binding.
- **Bandwidth-bound** (STREAM) saturates memory bandwidth well before 48 threads;
  `OMP_PROC_BIND=spread` (using both sockets/NUMA domains) often beats `close`
  because it uses both memory controllers. **Measure both.**
- Going beyond 48 threads oversubscribes physical cores and **degrades**
  performance — keep `OMP_NUM_THREADS ≤ 48`.

!!! tip "First-touch NUMA"
    Initialise (write) large arrays with the same thread layout you compute with,
    so pages are allocated on the NUMA node that will use them. Combined with
    `OMP_PROC_BIND`, this avoids cross-socket memory traffic.

## MPI (distributed memory, many nodes)

MPI parallelises across **many nodes**, each rank a separate process, exchanging
messages over **InfiniBand NDR**. Available MPI families: **Intel MPI**,
**Open MPI**, **MVAPICH**, **MPICH**.

### Compile & run

```bash
spack load intel-oneapi-compilers intel-oneapi-mpi
mpiicx -O3 -xCASCADELAKE app.c -o app_mpi        # Intel MPI + Intel LLVM
# or: mpicc -O3 -march=cascadelake app.c -o app_mpi   # Intel MPI + GCC / Open MPI
```

Launch through SLURM's `srun` (it inherits the allocation and handles binding):

```bash
#SBATCH --nodes=4
#SBATCH --ntasks-per-node=48       # 48 ranks/node = 192 ranks
srun --mpi=auto --cpu-bind=cores ./app_mpi
```

Useful fabric/pinning knobs:

```bash
# Intel MPI
export I_MPI_FABRICS=shm:ofi       # shared memory intra-node, OFI/IB inter-node
export I_MPI_PIN=1                  # pin ranks to cores
export I_MPI_DEBUG=5               # print pinning/fabric info (raise to diagnose)

# Open MPI (UCX over InfiniBand)
# srun --mpi=pmix ... ; UCX auto-selects IB transports
```

### Case study 3 — strong vs weak scaling

- **Strong scaling** — fix the total problem size, increase ranks; ideal time
  halves as ranks double. Plot efficiency `T(1)/(N·T(N))`.

  ```bash
  for N in 48 96 192 384; do        # 1,2,4,8 nodes × 48
    sbatch --nodes=$((N/48)) --ntasks-per-node=48 run.slurm
  done
  ```

- **Weak scaling** — grow the problem *with* the ranks (fixed work/rank); ideal
  time stays flat.

**Interpretation:** strong scaling eventually flattens when per-rank work gets too
small and **communication dominates** (surface-to-volume). If efficiency drops
early, check: rank–core binding (`I_MPI_DEBUG=5`), whether the code is latency- or
bandwidth-bound, and load balance. Confirm the IB fabric is actually used (not
Ethernet) via the MPI debug output.

## Hybrid MPI + OpenMP

Combine MPI *across/within* nodes with OpenMP *within* a rank. The rule on a
48-core node: **ranks-per-node × threads-per-rank = 48**.

```bash
#SBATCH --nodes=4
#SBATCH --ntasks-per-node=2        # 2 ranks/node (one per socket/NUMA domain)
#SBATCH --cpus-per-task=24         # 24 threads/rank -> 2×24 = 48

export OMP_NUM_THREADS=${SLURM_CPUS_PER_TASK}
export OMP_PROC_BIND=close
export OMP_PLACES=cores
export I_MPI_PIN_DOMAIN=socket     # give each rank a whole socket
srun --cpu-bind=cores ./app_hybrid
```

### Case study 4 — finding the best ranks × threads ratio

Sweep the decomposition on one node (all give 48 cores) and pick the fastest:

| Config | ranks/node | threads/rank | Good when |
| --- | --- | --- | --- |
| Pure MPI | 48 | 1 | Communication-light, good MPI scaling |
| **1 rank/socket** | 2 | 24 | **Common sweet spot** — 1 rank per NUMA domain |
| Balanced | 4 | 12 | Moderate comm + threading |
| Thread-heavy | 8 | 6 | Large per-rank memory, few messages |

```bash
for r in 48 8 4 2; do
  t=$((48 / r))
  sbatch --nodes=1 --ntasks-per-node=$r --cpus-per-task=$t run_hybrid.slurm
done
```

**Interpretation:** aligning **one rank per NUMA domain** (2 ranks/node × 24
threads) usually minimises cross-socket traffic while keeping MPI message counts
low. Fewer, larger messages over InfiniBand generally beat many tiny ones. Always
keep `ranks × threads = 48` and bind both ranks and threads.

## GPU offload (A100)

For the GPU nodes, offload with CUDA, OpenACC or OpenMP-target and build for
**`sm_80` / `cc80`**:

```bash
nvc -O3 -tp=cascadelake -acc -gpu=cc80 -Minfo=accel app.c -o app_gpu   # OpenACC
nvc -O3 -mp=gpu -gpu=cc80 -Minfo=mp app.c -o app_omp_gpu               # OpenMP target
nvcc -O3 -arch=sm_80 app.cu -o app_cuda                                # CUDA
```

`-Minfo=accel`/`-Minfo=mp` reports exactly what got offloaded and any data-movement
issues. See [GPU Computing](gpu.md) for running, MPS and multi-GPU jobs.

## Quick checklist

- [ ] Built and running with the **same** compiler/library versions.
- [ ] Tuned flags (`-O3` + `-march/-x/-tp` for Cascade Lake); accuracy validated.
- [ ] Confirmed vectorization (`-fopt-info-vec` / `-qopt-report` / `-Minfo=vect`).
- [ ] OpenMP: `OMP_NUM_THREADS ≤ 48`, threads **bound** (`OMP_PLACES=cores`).
- [ ] MPI: launched via `srun`, ranks **bound**, IB fabric confirmed.
- [ ] Hybrid: `ranks × threads = 48`, one rank per NUMA domain as a starting point.
- [ ] Right-sized `--time`, nodes and memory; output captured.

Related: [Building Software](building.md) · [Batch System](batch.md) ·
[GPU Computing](gpu.md) · [Job Script Examples](examples.md).
