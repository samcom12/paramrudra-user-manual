# OpenFOAM

## Introduction

OpenFOAM is a GPL-opensource C++ CFD-toolbox. This offering is supported
by OpenCFD Ltd, producer and distributor of the OpenFOAM software via
www.openfoam.com, and owner of the OPENFOAM trademark. OpenCFD Ltd has
been developing and releasing OpenFOAM since its debut in 2004.


The official website for OpenFOAM:
<https://www.openfoam.com/>


### Input

The following example illustrates the running of Openfoam with the commonly used data set. 

The benchmark input files are pre-installed on the PARAM Rudra system and are available at:

```bash
/home/apps/hpc_inputs/applications/OPENFOAM/Motorbike_bench_template.tar.gz
```

Copy the benchmark dataset to your working directory before running the example:

```bash
cp -r /home/apps/hpc_inputs/applications/OPENFOAM/Motorbike_bench_template.tar.gz .
cd Motorbike_bench_template.tar.gz
```


!!! note

    The benchmark dataset is also available from the official Openfoam benchmark repository:

    ```bash
    Wget http://openfoamwiki.net/images/6/62/Motorbike_bench_template.tar.gz
    ```


## Sample SLURM script

A sample Slurm job script for NWChem is available at:

```bash
/home/apps/hpc_inputs/scripts/openfoam.slurm
```

Copy it to your working directory:

or

```bash
#!/bin/bash
#SBATCH --job-name="rfm_job"
#SBATCH --ntasks=96
#SBATCH --ntasks-per-node=48
#SBATCH --output=rfm_job.out
#SBATCH --error=rfm_job.err
#SBATCH --exclusive
#SBATCH --partition=cpu


# 1. Initialize Spack
. /home/apps/spack/share/spack/setup-env.sh

# 2. Load ONLY OpenFOAM (Spack will load the correct MPI dependency automatically)
spack load openfoam/q5jbckq

# 3. Source the OpenFOAM environment to set correct PATH and MPI variables
source /home/apps/spack/opt/spack/linux-cascadelake/openfoam-2512-q5jbckqquhnhhyuzyysoiz4rdxlevonq/etc/bashrc
export OMP_NUM_THREADS=1

# Extract the benchmark files
tar --strip-components 2 -xf Motorbike_bench_template.tar.gz bench_template/basecase

# Grant execute permissions
chmod +x Allclean Allmesh Allmesh_serial

# Clean previous run
./Allclean

# Patch for ESI OpenFOAM v2512 Compatibility
sed -i 's/surfaceFeatures/surfaceFeatureExtract/g' Allmesh
sed -i 's/surfaceFeatures/surfaceFeatureExtract/g' Allmesh_serial
if [ -f system/surfaceFeaturesDict ]; then
    mv system/surfaceFeaturesDict system/surfaceFeatureExtractDict
fi

# Configure decomposing and system files
sed -i -- "s/method .*/method          scotch;/g" system/decomposeParDict
sed -i -- "s/numberOfSubdomains .*/numberOfSubdomains 96;/g" system/decomposeParDict
sed -i -- 's/    #include "streamLines"//g' system/controlDict
sed -i -- 's/    #include "wallBoundedStreamLines"//g' system/controlDict
sed -i -- 's|caseDicts|caseDicts/mesh/generation|' system/meshQualityDict

# Execute meshing
./Allmesh

# Run using the correct mpirun configured by OpenFOAM
time \
mpirun -np 96 simpleFoam -parallel
```

## Expected Output

Reference output files for verification are available at:

```bash
/home/apps/hpc_inputs/output/openfoam.out
```

## Notes

- The `decomposePar` step must produce exactly `$SLURM_NTASKS` subdomains; the
  `sed` line above keeps them in sync.
- The Motorbike tutorial (`Motorbike_bench_template`) is a common benchmark case.

!!! tip "Scaling"
    `cpu` is capped at 1 node here (48 subdomains). For larger decompositions use
    `hm` (≤8 nodes), or contact [support](../support.md).
