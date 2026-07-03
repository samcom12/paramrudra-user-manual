# Job Script Examples

Copy-and-adapt SLURM scripts for common workloads. Replace `myproject` with your
[account](batch.md#your-account--a-is-mandatory) and set core/GPU counts to match
the [hardware you confirmed](configuration.md#node-types-and-per-node-hardware).

!!! note "Save these on the cluster"
    Put job scripts in `$HOME` (with your code) and run from `/scratch`. Submit
    with `sbatch script.slurm`.

!!! tip "Loading toolchains"
    These templates use `module load` for brevity. On PARAM Rudra the primary
    package manager is **[Spack](spack.md)** — in practice you'll often replace
    the `module load` lines with:
    ```bash
    module load spack
    . /home/apps/spack/share/spack/setup-env.sh
    spack load intel-oneapi-compilers /<hash>
    spack load intel-oneapi-mpi        /<hash>
    ```
    For ready-made application scripts (GROMACS, LAMMPS, WRF …) see
    [Applications](applications.md).

## 1. Serial / single-core job (`cpu`)

```bash
#!/bin/bash
#SBATCH --job-name=serial
#SBATCH --account=myproject
#SBATCH --partition=cpu
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=1
#SBATCH --time=01:00:00
#SBATCH --output=%x-%j.out

set -euo pipefail
module purge
module load gcc/12

cd /scratch/$USER/serial_run
./my_serial_app input.dat
```

## 2. OpenMP (shared-memory, single node)

```bash
#!/bin/bash
#SBATCH --job-name=openmp
#SBATCH --account=myproject
#SBATCH --partition=cpu
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=48        # threads = cores on the node
#SBATCH --time=02:00:00
#SBATCH --output=%x-%j.out

set -euo pipefail
module purge
module load gcc/12

export OMP_NUM_THREADS=${SLURM_CPUS_PER_TASK}
export OMP_PLACES=cores
export OMP_PROC_BIND=close

cd /scratch/$USER/omp_run
srun ./my_openmp_app
```

## 3. MPI single node (`cpu` partition caps at 1 node)

```bash
#!/bin/bash
#SBATCH --job-name=mpi1node
#SBATCH --account=myproject
#SBATCH --partition=cpu
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=48      # ranks = cores on the node
#SBATCH --cpus-per-task=1
#SBATCH --time=04:00:00
#SBATCH --output=%x-%j.out

set -euo pipefail
module purge
module load gcc/12 openmpi/4.1.5

cd /scratch/$USER/mpi_run
srun --cpu-bind=cores ./my_mpi_app
```

!!! tip "Need many CPU nodes?"
    The `cpu` partition allows **1 node per job**. For large multi-node MPI on
    CPUs, discuss options with [support](support.md); multi-node scaling is
    otherwise available on `gpu` (≤128 nodes) and `hm` (≤8 nodes).

## 4. Hybrid MPI + OpenMP

```bash
#!/bin/bash
#SBATCH --job-name=hybrid
#SBATCH --account=myproject
#SBATCH --partition=cpu
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=4       # 4 MPI ranks
#SBATCH --cpus-per-task=12        # 12 threads each -> 48 cores
#SBATCH --time=06:00:00
#SBATCH --output=%x-%j.out

set -euo pipefail
module purge
module load gcc/12 openmpi/4.1.5

export OMP_NUM_THREADS=${SLURM_CPUS_PER_TASK}
export OMP_PROC_BIND=close
export OMP_PLACES=cores

cd /scratch/$USER/hybrid_run
srun --cpu-bind=cores ./my_hybrid_app
```

## 5. High-memory job (`hm`, up to 8 nodes)

```bash
#!/bin/bash
#SBATCH --job-name=bigmem
#SBATCH --account=myproject
#SBATCH --partition=hm
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=32
#SBATCH --mem=0                   # request all memory on the node
#SBATCH --time=1-00:00:00
#SBATCH --output=%x-%j.out

set -euo pipefail
module purge
module load gcc/12

export OMP_NUM_THREADS=${SLURM_CPUS_PER_TASK}
cd /scratch/$USER/bigmem_run
srun ./memory_hungry_app large_input
```

## 6. Single-GPU job (`gpu`)

```bash
#!/bin/bash
#SBATCH --job-name=gpu1
#SBATCH --account=myproject
#SBATCH --partition=gpu
#SBATCH --nodes=1
#SBATCH --gres=gpu:1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=8
#SBATCH --time=02:00:00
#SBATCH --output=%x-%j.out

set -euo pipefail
module purge
module load cuda

nvidia-smi
cd /scratch/$USER/gpu_run
srun ./my_gpu_app
```

## 7. Multi-node, multi-GPU (PyTorch DDP)

```bash
#!/bin/bash
#SBATCH --job-name=ddp
#SBATCH --account=myproject
#SBATCH --partition=gpu
#SBATCH --nodes=4
#SBATCH --gres=gpu:4
#SBATCH --ntasks-per-node=4       # one task per GPU
#SBATCH --cpus-per-task=8
#SBATCH --time=12:00:00
#SBATCH --output=%x-%j.out

set -euo pipefail
module purge
module load cuda
conda activate myenv

export MASTER_ADDR=$(scontrol show hostnames "$SLURM_JOB_NODELIST" | head -n1)
export MASTER_PORT=29500
export NCCL_IB_HCA=mlx5           # confirm with support / `ibstat`

srun python train_ddp.py
```

## 8. Job array (parameter sweep)

```bash
#!/bin/bash
#SBATCH --job-name=sweep
#SBATCH --account=myproject
#SBATCH --partition=cpu
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=4
#SBATCH --time=00:30:00
#SBATCH --array=1-50%10           # 50 cases, 10 at a time
#SBATCH --output=sweep-%A_%a.out

set -euo pipefail
module purge
module load gcc/12

PARAM=$(sed -n "${SLURM_ARRAY_TASK_ID}p" params.txt)
cd /scratch/$USER/sweep
srun ./my_app --param "$PARAM"
```

## 9. Chained workflow (dependencies)

```bash
#!/bin/bash
# submit_pipeline.sh  — run: bash submit_pipeline.sh
set -euo pipefail
jid1=$(sbatch --parsable preprocess.slurm)
jid2=$(sbatch --parsable --dependency=afterok:$jid1 simulate.slurm)
jid3=$(sbatch --parsable --dependency=afterok:$jid2 postprocess.slurm)
echo "Submitted pipeline: $jid1 -> $jid2 -> $jid3"
```

## 10. Interactive session

```bash
# CPU
salloc -A myproject -p cpu -N 1 -t 01:00:00
srun --pty bash

# GPU
salloc -A myproject -p gpu -N 1 --gres=gpu:1 -c 8 -t 00:30:00
srun --pty bash
```

Continue to [Data Management](data.md) for staging inputs and protecting your
results.
