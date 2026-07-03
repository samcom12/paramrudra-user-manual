# OpenFOAM

**OpenFOAM** is an open-source C++ CFD toolbox. Official site:
<https://www.openfoam.com/>.

## CPU job script

```bash
#!/bin/bash
#SBATCH --job-name=openfoam
#SBATCH --account=myproject
#SBATCH --partition=cpu
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=48
#SBATCH --time=04:00:00
#SBATCH --exclusive
#SBATCH --output=job.%J.out
#SBATCH --error=job.%J.err

source /home/apps/spack/share/spack/setup-env.sh
spack load openfoam
spack load intel-oneapi-mpi
export OMP_NUM_THREADS=1

# Set decomposition to match the number of MPI ranks, mesh, then solve
sed -i "s/numberOfSubdomains .*/numberOfSubdomains $SLURM_NTASKS;/g" system/decomposeParDict
./Allmesh
time mpirun -np $SLURM_NTASKS simpleFoam -parallel
```

## Notes

- The `decomposePar` step must produce exactly `$SLURM_NTASKS` subdomains; the
  `sed` line above keeps them in sync.
- The Motorbike tutorial (`Motorbike_bench_template`) is a common benchmark case.

!!! tip "Scaling"
    `cpu` is capped at 1 node here (48 subdomains). For larger decompositions use
    `hm` (≤8 nodes), or contact [support](../support.md).
