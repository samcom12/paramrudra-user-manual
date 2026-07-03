# Quantum Chemistry & Others

Setup pattern for all of these: initialise [Spack](../spack.md), load the
application (and its MPI), then launch with `mpirun -np $SLURM_NTASKS` (or `srun`).
Add `#SBATCH -A <account>` and confirm builds with `spack find -l <app>`.

```bash
source /home/apps/spack/share/spack/setup-env.sh
```

## NWChem

Ab-initio computational chemistry (quantum chemistry + molecular dynamics).
Site: <https://www.nwchem-sw.org/>.

```bash
#SBATCH --partition=cpu
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=48
spack load nwchem
spack load intel-oneapi-mpi
time mpirun -np $SLURM_NTASKS nwchem input.nw
```

## CP2K

Quantum chemistry and solid-state physics — atomistic simulations of solids,
liquids, molecular and biological systems. Site: <https://www.cp2k.org>.

```bash
#SBATCH --partition=cpu
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=48
spack load cp2k
mpirun -np $SLURM_NTASKS cp2k.popt -i input.inp
```

## OpenMolcas

Multiconfigurational quantum chemistry. Site: <https://www.molcas.org/>.

```bash
#SBATCH --partition=hm
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=12
#SBATCH -t 24:00:00
module load miniconda
spack load openmolcas
pymolcas grid.inp > grid.out
```

## AMBER24

Biomolecular molecular dynamics (proteins, DNA, RNA), with CPU and GPU engines
(`pmemd`, `pmemd.cuda`). Site: <https://ambermd.org/>.

```bash
#SBATCH --partition=cpu
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=48
export OMP_NUM_THREADS=1
module load apps/amber24            # site application module
# CPU: mpirun -np $SLURM_NTASKS pmemd.MPI  -O -i mdin -p prmtop -c inpcrd -o mdout
# GPU: submit to the gpu partition and use pmemd.cuda
```

!!! note "ROMS, Quantum ESPRESSO, ABINIT, SU2, RegCM, MOM"
    These follow the same Spack pattern — `spack load <app>` (+ MPI) and launch
    with `mpirun`/`srun`. Use `spack find -l <app>` to see installed versions and
    hashes.
