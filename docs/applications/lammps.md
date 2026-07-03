# LAMMPS

**LAMMPS** (Large-scale Atomic/Molecular Massively Parallel Simulator) is a
classical molecular dynamics code used across materials science, physics and
chemistry. Official site: <https://www.lammps.org/>.

## CPU job script

```bash
#!/bin/bash
#SBATCH --job-name=lammps
#SBATCH --account=myproject
#SBATCH --partition=cpu
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=48
#SBATCH --time=04:00:00
#SBATCH --exclusive
#SBATCH --output=job.%J.out
#SBATCH --error=job.%J.err

source /home/apps/spack/share/spack/setup-env.sh
spack load lammps
spack load intel-oneapi-mpi
export OMP_NUM_THREADS=1

time mpirun -np $SLURM_NTASKS lmp -in in.lj
```

## Input

The standard `in.lj` 3-D Lennard-Jones melt is a common starting benchmark
(available from the LAMMPS benchmarks page). Point the `-in` flag at your input
deck.

!!! tip "Scaling"
    `cpu` is capped at 1 node on this system. For multi-node runs use `hm`
    (≤8 nodes) or the `gpu` partition (with a GPU-enabled LAMMPS build), or ask
    [support](../support.md). Keep `OMP_NUM_THREADS=1` unless you have tuned a
    hybrid MPI+OpenMP configuration.
