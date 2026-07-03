# NAMD

**NAMD** is a parallel molecular dynamics code designed for high-performance
simulation of large biomolecular systems. It runs on both CPUs and GPUs.
Official site: <https://www.ks.uiuc.edu/Research/namd/>.

## CPU job script

```bash
#!/bin/bash
#SBATCH --job-name=namd
#SBATCH --account=myproject
#SBATCH --partition=cpu
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=48
#SBATCH --time=04:00:00
#SBATCH --exclusive
#SBATCH --output=job.%J.out
#SBATCH --error=job.%J.err

source /home/apps/spack/share/spack/setup-env.sh
spack load namd
export OMP_NUM_THREADS=1

tar -xvf apoa1.tar.gz && cd apoa1
time mpirun -np $SLURM_NTASKS namd2 apoa1.namd &> output
```

## Input

The **ApoA1** benchmark is a common test case (available from the NAMD
utilities page).

!!! tip "GPU runs"
    A CUDA-enabled NAMD build can use the A100 GPUs — submit to the `gpu`
    partition with `--gres=gpu:1` (or `:2`). See [GPU Computing](../gpu.md).
