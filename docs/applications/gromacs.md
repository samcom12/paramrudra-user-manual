# GROMACS

## Introduction

GROMACS is a versatile package to perform molecular dynamics, i.e. simulate the Newtonian equations of motion for systems with hundreds to millions of particles, and is a community-driven project. It is primarily designed for biochemical molecules like proteins, lipids, and nucleic acids that have a lot of complicated bonded interactions, but since GROMACS is extremely fast at calculating the nonbonded interactions (that usually dominate simulations) many groups are also using it for research on non-biological systems, e.g. polymers and fluid dynamics.

Official site: <https://www.gromacs.org/>.

### Input 

The following example illustrates the running of GROMACS with the commonly used water_GMX50_bare benchmark data set.  

The benchmark input files are pre-installed on the PARAM Rudra system and are available at:

```bash
/home/apps/hpc_inputs/applications/gromacs/water_GMX50_bare
```

Copy the benchmark dataset to your working directory before running the example:

```bash
cp -r /home/apps/hpc_inputs/applications/gromacs/water_GMX50_bare .
cd water_GMX50_bare
```

Running GROMACS is a two-step process:

- Generating the input file (.tpr)

- Running the generated input file (.tpr)

!!! note

    The benchmark dataset is also available from the official GROMACS benchmark repository:

    ```bash
    wget http://ftp.gromacs.org/pub/benchmarks/water_GMX50_bare.tar.gz
    tar -xzf water_GMX50_bare.tar.gz
    ```
## Running (two steps)

GROMACS runs are a two-step process: generate the run input (`.tpr`) with
`grompp`, then run it with `mdrun`.

## Sample SLURM script

A sample Slurm job script for GROMACS is available at:

```bash
/home/apps/hpc_inputs/scripts/gromacs.slurm
```

Copy it to your working directory:

or

```bash
#!/bin/bash
#SBATCH --job-name="rfm_job"
#SBATCH --ntasks=48
#SBATCH --ntasks-per-node=48
#SBATCH --output=rfm_job.out
#SBATCH --error=rfm_job.err
#SBATCH --exclusive
#SBATCH --partition=cpu


# Load required packages
spack load gromacs/pvlg3o7
spack load openmpi /22inq4o


# Extract benchmark
tar -xvf water_GMX50_bare.tar.gz


# Change to benchmark directory
cd water-cut1.0_GMX50_bare/3072


# Generate input
gmx_mpi grompp -f pme.mdp -c conf.gro -p topol.top -o water_pme.tpr


# Run simulation
time mpirun -np 48 gmx_mpi mdrun -nsteps 5000 -s water_pme.tpr


```

## Expected Output

Reference output files for verification are available at:

```bash
/home/apps/hpc_inputs/output/gromacs.out
```

Users can compare their results with the reference output to verify successful execution.

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
