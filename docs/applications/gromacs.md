# GROMACS

**GROMACS** (GROningen MAchine for Chemical Simulations) is a fast, widely-used
molecular dynamics package for proteins, lipids and nucleic acids, running on
both CPUs and GPUs. Official site: <https://www.gromacs.org/>.

## Running (two steps)

GROMACS runs are a two-step process: generate the run input (`.tpr`) with
`grompp`, then run it with `mdrun`.

## CPU job script

```bash
#!/bin/bash
#SBATCH --job-name=gromacs
#SBATCH --account=myproject
#SBATCH --partition=cpu
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=48
#SBATCH --time=04:00:00
#SBATCH --exclusive
#SBATCH --output=job.%J.out
#SBATCH --error=job.%J.err

source /home/apps/spack/share/spack/setup-env.sh
spack load gromacs            # e.g. gromacs@<ver> /<hash>  (spack find -l gromacs)
spack load openmpi           # or intel-oneapi-mpi
export OMP_NUM_THREADS=1

# 1) Build the .tpr, then 2) run
gmx_mpi grompp -f pme.mdp -c conf.gro -p topol.top -o water_pme.tpr
time mpirun -np $SLURM_NTASKS gmx_mpi mdrun -s water_pme.tpr
```

## GPU acceleration

GROMACS can offload nonbonded/PME work to the A100 GPUs. Use the `gpu` partition
and a CUDA-enabled GROMACS build:

```bash
#SBATCH --partition=gpu
#SBATCH --gres=gpu:1
...
source /home/apps/spack/share/spack/setup-env.sh
spack load gromacs           # a +cuda build
srun gmx_mpi mdrun -s water_pme.tpr -nb gpu -pme gpu
```

Build a CUDA + MPI GROMACS for A100 (`cuda_arch=80`) via
[Spack](../spack.md#installing-a-new-package-into-your-own-space) if a suitable
one isn't already installed.

## Benchmark data

The commonly used `water_GMX50_bare` benchmark set is available from the GROMACS
FTP site.

!!! tip "Scaling"
    `cpu` is capped at 1 node here. For larger runs use `hm` (≤8 nodes) or the
    `gpu` partition, or contact [support](../support.md).
