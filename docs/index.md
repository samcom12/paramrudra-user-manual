# PARAM Rudra — 20 PetaFlop System User Manual

Welcome to the user documentation for **PARAM Rudra**, a ~20 PetaFlop
supercomputer operated under the **National Supercomputing Mission (NSM)** by
**C-DAC**. This guide walks you through everything from your first login to
running large-scale CPU, high-memory and GPU jobs through the SLURM batch
system.

!!! tip "New here? Start with these three pages"
    1. [Getting Access](access.md) — connect over SSH (port **4422**).
    2. [Environment](environment.md) — modules, shell and your `/home` & `/scratch` directories.
    3. [Batch System (SLURM)](batch.md) — never run compute on the login node; submit jobs instead.

## System at a glance

| Resource | Count | Node name prefix | Partition |
| --- | --- | --- | --- |
| **Total compute nodes** | **2,906** | — | — |
| CPU-only nodes | 2,266 | `cbcn####` | `cpu` |
| GPU-accelerated nodes | 320 | `cbgpu####` | `gpu` |
| High-memory nodes | 320 | `cbhm####` | `hm` |
| Interconnect | High-speed **InfiniBand (IB)** | — | — |
| Login nodes | `login01`, `login02`, `login03`, … | — | — |

<div class="grid cards" markdown>

- :material-login: **[Getting Access](access.md)**

    SSH key setup, first login on port 4422, and login-node etiquette.

- :material-server-network: **[System Configuration](configuration.md)**

    Node types, partitions, interconnect and storage layout.

- :material-package-variant: **[Software Modules](modules.md)**

    Find and load compilers, libraries and applications with `module`.

- :material-hammer-wrench: **[Building Software](building.md)**

    Compilers, MPI wrappers, CUDA, CMake/Make, Python and Conda.

- :material-calendar-clock: **[Batch System (SLURM)](batch.md)**

    Partitions, `sbatch` scripts, `srun`, dependencies and interactive jobs.

- :material-expansion-card: **[GPU Computing](gpu.md)**

    Requesting GPUs, CUDA MPS, multi-GPU and multi-node runs.

</div>

## The golden rules

!!! danger "Do not run jobs on the login node"
    Running compute-heavy processes on a login node degrades service for
    everyone and **will result in your process being terminated without prior
    notice** (repeat offences may cost you account access). Always submit work
    through SLURM — see the [Batch System](batch.md) page.

!!! warning "`/scratch` is purged"
    Files in `/scratch` that have **not been accessed in the last 3 months are
    permanently deleted**. `/scratch` is fast working space, **not** long-term
    storage. Back up important results elsewhere — see [Data Management](data.md).

## Quick reference card

```bash
# Connect (replace samirs with your username)
ssh samirs@paramrudra.cdacb.in -p 4422

# Environment modules
module avail                 # list available modules
module load <name>           # load a module
module list                  # show loaded modules

# SLURM essentials
sinfo                        # partition / node status
squeue -u $USER              # your jobs
sbatch job.slurm             # submit a batch job
salloc ...                   # interactive allocation
scancel <jobid>              # cancel a job

# Always specify your accounting/project code
#SBATCH -A <your_account>
```

---

!!! note "About this manual"
    This is a **community/user-maintained** guide, structured after the
    excellent [JUPITER documentation](https://apps.fz-juelich.de/jsc/hps/jupiter/index.html)
    at Jülich Supercomputing Centre and grounded in the live PARAM Rudra login
    banner and SLURM configuration. Where a value is site-specific and may
    change (exact per-node core counts, GPU model, quotas), the page tells you
    the command to confirm it on the system. Corrections are welcome via
    [pull request](https://github.com/samcom12/paramrudra-user-manual).
