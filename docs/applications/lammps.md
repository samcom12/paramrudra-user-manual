# LAMMPS

## Introduction

**LAMMPS** (Large-scale Atomic/Molecular Massively Parallel Simulator) is a
classical molecular dynamics code used across materials science, physics and
chemistry. Official site: <https://www.lammps.org/>.

### Input

The following example illustrates the running of LAMMPS with the commonly used data set. 

The benchmark input files are pre-installed on the PARAM Rudra system and are available at:

```bash
/home/apps/hpc_inputs/applications/LAMMPS/in.lj.txt
```

Copy the benchmark dataset to your working directory before running the example:

```bash
cp -r /home/apps/hpc_inputs/applications/LAMMPS/in.lj.txt .
cd in.lj.txt
```


!!! note

    The Data Set link for sample data set: [" https://www.lammps.org/bench/:/in.lj.txt"](https://www.lammps.org/bench/:/in.lj.txt)

## Sample SLURM script

A sample Slurm job script for GROMACS is available at:

```bash
/home/apps/hpc_inputs/scripts/lamps.slurm
```

Copy it to your working directory:

or

```bash
#!/bin/bash
#SBATCH --job-name="rfm_job"
#SBATCH --ntasks=192
#SBATCH --ntasks-per-node=48
#SBATCH --output=rfm_job.out
#SBATCH --error=rfm_job.err
#SBATCH --exclusive
#SBATCH –partition=cpu
export SPACK_ROOT=/home/apps/spack
. $SPACK_ROOT/share/spack/setup-env.sh
spack load lammps /4dwl4bk
spack load intel-oneapi-mpi/ptyduik


export OMP_NUM_THREADS=1
time mpirun -np 192 lmp  -in in.lj.txt

```

## Expected Output

Reference output files for verification are available at:

```bash
/home/apps/hpc_inputs/output/lamps.out
```

Users can compare their results with the reference output to verify successful execution.

## Input

The standard `in.lj` 3-D Lennard-Jones melt is a common starting benchmark
(available from the LAMMPS benchmarks page). Point the `-in` flag at your input
deck.

!!! tip "Scaling"
    `cpu` is capped at 1 node on this system. For multi-node runs use `hm`
    (≤8 nodes) or the `gpu` partition (with a GPU-enabled LAMMPS build), or ask
    [support](../support.md). Keep `OMP_NUM_THREADS=1` unless you have tuned a
    hybrid MPI+OpenMP configuration.
