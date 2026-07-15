## Introduction

NWChem is an ab initio computational chemistry software package that includes quantum chemical and molecular dynamics functionality. It was designed to run on high-performance parallel supercomputers as well as conventional workstation clusters

The official website for NWChem: [https://www.nwchem-sw.org/](https://www.nwchem-sw.org/)


### Input

The following example illustrates the running of NWChem with the commonly used data set. 

The benchmark input files are pre-installed on the PARAM Rudra system and are available at:

```bash
/home/apps/hpc_inputs/applications/NWCHEM/input2.nw
```

Copy the benchmark dataset to your working directory before running the example:

```bash
cp -r /home/apps/hpc_inputs/applications/NWCHEM/input2.nw .
cd input2.nw
```


!!! note

    The Data Set link for sample data set: [" https://www.nwchem.org/bench/:/input2.nw"](https://nwchemgit.github.io/Sample.html#water-scf-calculation-and-geometry-optimization-in-a-6-31g-basis)

## Sample SLURM script

A sample Slurm job script for NWChem is available at:

```bash
/home/apps/hpc_inputs/scripts/nwchem.slurm
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
#SBATCH --partition=cpu
spack load nwchem/zsq4xv3
spack load intel-oneapi-mpi/ptyduik
time \
mpirun -np 192 nwchem input.nw

```

## Expected Output

Reference output files for verification are available at:

```bash
/home/apps/hpc_inputs/output/nwchem.out
```

Users can compare their results with the reference output to verify successful execution.