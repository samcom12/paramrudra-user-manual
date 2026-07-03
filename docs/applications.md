# Applications

PARAM Rudra provides a broad catalogue of scientific applications through
**[Spack](spack.md)** (and some via modules). This page lists what's installed
and gives working, adapted job scripts for the most common codes.

!!! warning "Node counts adapted to this system's limits"
    The reference scripts below are adapted from C-DAC application examples. On
    **this** system the `cpu` partition allows **max 1 node/job**; `hm` allows 8;
    `gpu` allows 128 (see [Batch System](batch.md)). Scripts here default to a
    single `cpu` node (48 ranks). To scale wider, use `hm` (≤8 nodes) or the
    `gpu` partition, or ask [support](support.md). Always add your
    `#SBATCH -A <account>`, and confirm the exact Spack hash with
    `spack find -l <package>`.

## Installed applications (selection)

| Domain | Applications |
| --- | --- |
| Molecular dynamics | **GROMACS**, **LAMMPS**, **NAMD** (CPU & GPU), **AMBER24** |
| Quantum chemistry / materials | **CP2K**, **NWChem**, **Quantum ESPRESSO**, ABINIT, **OpenMolcas** |
| Bio-informatics | MUMmer, HMMER, MEME, Schrödinger, PHYLIP, mpiBLAST, ClustalW |
| CFD | **OpenFOAM**, SU2 |
| Weather / ocean / climate | **WRF-ARW** (+WPS, ARWPost), RegCM, MOM, **ROMS** |
| Deep learning | TensorFlow, PyTorch, Keras, Theano, Caffe, cuDNN (see [ML/DL](machine-learning.md)) |
| Visualization | ParaView, VisIt, VMD, GrADS |
| Libraries | NetCDF, PNetCDF, HDF5, FFTW, Boost, Jasper, Tcl, Intel MKL |

Find and load any of them:

```bash
module load spack
. /home/apps/spack/share/spack/setup-env.sh
spack find -l gromacs        # available builds + hashes
spack load gromacs@<ver> /<hash>
```

## GROMACS (molecular dynamics)

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
spack load gromacs           # e.g. gromacs@<ver> /<hash>
spack load openmpi           # or intel-oneapi-mpi
export OMP_NUM_THREADS=1

# Prepare (.tpr) then run
gmx_mpi grompp -f pme.mdp -c conf.gro -p topol.top -o water_pme.tpr
time mpirun -np $SLURM_NTASKS gmx_mpi mdrun -s water_pme.tpr
```

Benchmark data: `water_GMX50_bare` from the GROMACS FTP.

## LAMMPS (molecular dynamics)

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

Input: the standard `in.lj` 3-D Lennard-Jones melt. Larger runs (>1 node) →
use `hm` (≤8 nodes) or ask support.

## NAMD (molecular dynamics)

```bash
#!/bin/bash
#SBATCH --job-name=namd
#SBATCH --account=myproject
#SBATCH --partition=cpu
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=48
#SBATCH --exclusive
#SBATCH --output=job.%J.out
#SBATCH --error=job.%J.err

source /home/apps/spack/share/spack/setup-env.sh
spack load namd
export OMP_NUM_THREADS=1

tar -xvf apoa1.tar.gz && cd apoa1
time mpirun -np $SLURM_NTASKS namd2 apoa1.namd &> output
```

## OpenFOAM (CFD)

```bash
#!/bin/bash
#SBATCH --job-name=openfoam
#SBATCH --account=myproject
#SBATCH --partition=cpu
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=48
#SBATCH --exclusive
#SBATCH --output=job.%J.out
#SBATCH --error=job.%J.err

source /home/apps/spack/share/spack/setup-env.sh
spack load openfoam
spack load intel-oneapi-mpi
export OMP_NUM_THREADS=1

# Decompose to $SLURM_NTASKS subdomains, mesh, then solve in parallel
sed -i "s/numberOfSubdomains .*/numberOfSubdomains $SLURM_NTASKS;/g" system/decomposeParDict
./Allmesh
time mpirun -np $SLURM_NTASKS simpleFoam -parallel
```

## WRF (weather)

```bash
#!/bin/bash
#SBATCH --job-name=wrf
#SBATCH --account=myproject
#SBATCH --partition=hm            # WRF benefits from the hm partition
#SBATCH --nodes=2                 # hm allows up to 8 nodes/job
#SBATCH --ntasks-per-node=48
#SBATCH --exclusive
#SBATCH --output=job.%J.out
#SBATCH --error=job.%J.err

source /home/apps/spack/share/spack/setup-env.sh
spack load intel-oneapi-mpi
spack load intel-oneapi-compilers
export OMP_NUM_THREADS=1
ulimit -s unlimited

module load apps/wrf-4.5.1        # site application module, if provided
cd run
time mpirun -np $SLURM_NTASKS wrf.exe
```

Sample datasets (site-specific): check `/home/apps/hpc_inputs/WRF`.

## NWChem / CP2K / OpenMolcas / AMBER24

These follow the same pattern — set up Spack, `spack load <app>` (plus its MPI),
then launch with `mpirun -np $SLURM_NTASKS`. Examples:

```bash
# NWChem
spack load nwchem; spack load intel-oneapi-mpi
time mpirun -np $SLURM_NTASKS nwchem input.nw

# CP2K
source /home/apps/spack/share/spack/setup-env.sh
spack load cp2k
mpirun -np $SLURM_NTASKS cp2k.popt -i input.inp

# OpenMolcas (often on hm; single-node)
module load miniconda; spack load openmolcas
pymolcas grid.inp > grid.out

# AMBER24 (via application module)
module load apps/amber24
```

!!! tip "Acknowledge NSM in your publications"
    If you publish results obtained on PARAM Rudra, please include the NSM
    acknowledgement — see [Accounts & Acknowledgement](acknowledgement.md).

More generic templates (serial, OpenMP, MPI, hybrid, GPU, arrays) are on the
[Job Script Examples](examples.md) page.
