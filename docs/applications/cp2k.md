# Cp2k

## Introduction

CP2K is a quantum chemistry and solid state physics software package that can perform atomistic simulations of solid state, liquid, molecular, periodic, material, crystal, and biological systems.

The official website for CP2K: <https://www.cp2k.org>


### Input

The following example illustrates the running of Cp2k with the commonly used data set. 

The benchmark input files are pre-installed on the PARAM Rudra system and are available at:

```bash
/home/apps/hpc_inputs/applications/CP2K/H2O.inp
```

Copy the benchmark dataset to your working directory before running the example:

```bash
cp -r /home/apps/hpc_inputs/applications/CP2K/H2O.inp .
cd Motorbike_bench_template.tar.gz
```


## Sample SLURM script

A sample Slurm job script for NWChem is available at:

```bash
/home/apps/hpc_inputs/scripts/cp2k.slurm
```

Copy it to your working directory:

or

```bash
#!/bin/bash
#SBATCH --job-name="cp2k_job"
#SBATCH --ntasks=20
#SBATCH --ntasks-per-node=20
#SBATCH --partition=gpu
#SBATCH --gres=gpu:1           # Request 1 GPU device
#SBATCH --output=cp2k.out
#SBATCH --error=cp2k.err


# 1. Initialize Spack
. /home/apps/spack/share/spack/setup-env.sh


# 2. Load CP2K
spack load cp2k/i55m5p3


# 3. Detect which parallel executable is available (cp2k.psmp or cp2k.popt)
if command -v cp2k.psmp &>/dev/null; then
    EXE="cp2k.psmp"
elif command -v cp2k.popt &>/dev/null; then
    EXE="cp2k.popt"
else
    EXE="cp2k"
fi


# Run the job using mpirun (required for Open MPI builds without PMI)
time \
mpirun -np 20 $EXE -i input.inp
```


## Expected Output

Reference output files for verification are available at:

```bash
/home/apps/hpc_inputs/output/cp2k.out
```