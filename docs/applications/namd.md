# NAMD

## Introduction

**NAMD** is a parallel molecular dynamics code designed for high-performance
simulation of large biomolecular systems. It runs on both CPUs and GPUs.
Official site: <https://www.ks.uiuc.edu/Research/namd/>.


### Input

The following example illustrates the running of NAMD with the commonly used data set. 

The benchmark input files are pre-installed on the PARAM Rudra system and are available at:

```bash
/home/apps/hpc_inputs/applications/NAMD/apoa1.tar.gz
```

Copy the benchmark dataset to your working directory before running the example:

```bash
cp -r /home/apps/hpc_inputs/applications/NAMD/apoa1.tar.gz .
cd apoa1.tar.gz
```


!!! note

    The benchmark dataset is also available from the official NAMD benchmark repository:

    ```bash 
    wget https://www.ks.uiuc.edu/Research/namd/utilities/apoa1.tar.gz

    ```
## Sample SLURM script

A sample Slurm job script for NAMD is available at:

```bash
/home/apps/hpc_inputs/scripts/namd.slurm
```

Copy it to your working directory:

or

## CPU job script

```bash
#!/bin/bash
#SBATCH --job-name="rfm_job"
#SBATCH --ntasks=48
#SBATCH --ntasks-per-node=48
#SBATCH --output=rfm_job.out
#SBATCH --error=rfm_job.err
#SBATCH --exclusive
#SBATCH --partition=cpu
spack load namd/syvixe4
export OMP_NUM_THREADS=1
tar -xvf apoa1.tar.gz
cd apoa1
time \
mpirun -np 48 namd3 apoa1.namd &> output

```


## Expected Output

Reference output files for verification are available at:

```bash
/home/apps/hpc_inputs/output/namd.out
```

Users can compare their results with the reference output to verify successful execution.

## Input

The **ApoA1** benchmark is a common test case (available from the NAMD
utilities page).

!!! tip "GPU runs"
    A CUDA-enabled NAMD build can use the A100 GPUs — submit to the `gpu`
    partition with `--gres=gpu:1` (or `:2`). See [GPU Computing](../gpu.md).
