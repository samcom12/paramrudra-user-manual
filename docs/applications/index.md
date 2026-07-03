# Applications

PARAM Rudra provides a broad catalogue of scientific applications through
**[Spack](../spack.md)** (and some via modules). Pick a code from the list for a
ready-to-run, adapted job script.

<div class="grid cards" markdown>

- :material-molecule: **[GROMACS](gromacs.md)** — molecular dynamics
- :material-atom: **[LAMMPS](lammps.md)** — atomic/molecular MD
- :material-dna: **[NAMD](namd.md)** — biomolecular MD
- :material-air-filter: **[OpenFOAM](openfoam.md)** — CFD
- :material-weather-partly-cloudy: **[WRF](wrf.md)** — weather / climate
- :material-flask: **[Quantum chemistry & more](quantum-chemistry.md)** — CP2K, NWChem, OpenMolcas, AMBER24

</div>

!!! warning "Node counts adapted to this system's limits"
    The scripts on these pages are adapted from C-DAC application examples for
    **this** system, where the `cpu` partition allows **max 1 node/job**; `hm`
    allows 8; `gpu` allows 128 (see [Batch System](../batch.md)). They default to
    a single `cpu` node (48 ranks); scale on `hm`/`gpu` or ask
    [support](../support.md). Always add `#SBATCH -A <account>` and confirm the
    exact Spack hash with `spack find -l <package>`.

## Loading applications with Spack

```bash
module load spack
. /home/apps/spack/share/spack/setup-env.sh
spack find -l gromacs           # available builds + hashes
spack load gromacs@<ver> /<hash>
```

## Installed applications (selection)

| Domain | Applications |
| --- | --- |
| Molecular dynamics | **GROMACS**, **LAMMPS**, **NAMD** (CPU & GPU), **AMBER24** |
| Quantum chemistry / materials | **CP2K**, **NWChem**, **Quantum ESPRESSO**, ABINIT, **OpenMolcas** |
| Bio-informatics | MUMmer, HMMER, MEME, Schrödinger, PHYLIP, mpiBLAST, ClustalW |
| CFD | **OpenFOAM**, SU2 |
| Weather / ocean / climate | **WRF-ARW** (+WPS, ARWPost), RegCM, MOM, **ROMS** |
| Deep learning | TensorFlow, PyTorch, Keras, Theano, Caffe, cuDNN (see [ML/DL](../machine-learning.md)) |
| Visualization | ParaView, VisIt, VMD, GrADS |
| Libraries | NetCDF, PNetCDF, HDF5, FFTW, Boost, Jasper, Tcl, Intel MKL |

!!! tip "Acknowledge NSM in your publications"
    If you publish results obtained on PARAM Rudra, please include the NSM
    acknowledgement — see [Accounts & Acknowledgement](../acknowledgement.md).

Generic templates (serial, OpenMP, MPI, hybrid, GPU, arrays) are on the
[Job Script Examples](../examples.md) page.
