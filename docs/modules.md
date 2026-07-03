# Software Modules

PARAM Rudra provides its software stack through **environment modules**. Modules
let you load a precise combination of compiler, MPI and application versions on
demand, and switch between them cleanly.

!!! quote "From the login banner"
    *"We strongly recommend that users use the module file already available on
    the cluster."*

## Everyday commands

| Command | What it does |
| --- | --- |
| `module avail` | List all available modules |
| `module load <name>` | Load a module (optionally version: `name/version`) |
| `module unload <name>` | Unload a single module |
| `module list` | Show currently loaded modules |
| `module purge` | Unload **everything** (clean slate) |
| `module show <name>` | Show what a module sets (paths, env vars, conflicts) |
| `module spider <name>` | Search all modules (Lmod) incl. hidden dependencies |
| `module swap <a> <b>` | Replace module `a` with `b` |

```bash
# Discover
module avail
module avail 2>&1 | grep -i openmpi      # search (module writes to stderr)

# Load a toolchain
module load gcc/12
module load openmpi/4.1.5

# Verify
module list

# Reset when switching projects
module purge
```

!!! tip "`module avail` output goes to stderr"
    Redirect it to filter: `module avail 2>&1 | grep -i <keyword>`.

## Typical software categories

The exact names and versions come from `module avail` on the system, but you can
generally expect:

- **Compilers** — GNU (`gcc`, `gfortran`), Intel oneAPI (`icc`/`icx`, `ifort`/`ifx`),
  possibly NVIDIA HPC SDK (`nvc`, `nvfortran`).
- **MPI** — OpenMPI, Intel MPI, and/or MPICH.
- **GPU** — CUDA toolkit(s), cuDNN, NCCL.
- **Numerical libraries** — Intel MKL, OpenBLAS, FFTW, ScaLAPACK, HDF5, NetCDF.
- **Applications** — common community codes (e.g. GROMACS, LAMMPS, OpenFOAM,
  WRF, QuantumESPRESSO) where licensed/installed.
- **Languages / tools** — Python (Conda), Julia, CMake, Git.

Confirm what is actually installed:

```bash
module avail 2>&1 | sort | less
module spider gromacs        # (Lmod) find versions and how to load
```

## Version pinning and reproducibility

Always load **explicit versions** in job scripts so results are reproducible and
do not silently change when defaults are updated:

```bash
# Good — explicit and reproducible
module purge
module load gcc/12
module load openmpi/4.1.5
module load fftw/3.3.10

# Risky — 'default' version may change over time
module load openmpi
```

Record the environment inside your job output for provenance:

```bash
module list 2>&1        # captured in your SLURM .out file
```

## Module dependencies and conflicts

- Some modules **require** others first (e.g. an MPI-built app needs its MPI and
  compiler loaded). `module show <name>` reveals prerequisites and conflicts.
- Loading two incompatible modules (e.g. two different MPI stacks) can break your
  build or run. When in doubt, `module purge` and load a clean, minimal set.

## Building your own module (advanced)

If you install software yourself under `/home` or `/scratch`, you can expose it
through a personal module tree:

```bash
mkdir -p $HOME/privatemodules
module use $HOME/privatemodules      # add your tree to the search path
```

Place modulefiles there (Tcl or Lua) that set `PATH`, `LD_LIBRARY_PATH`, etc.
For most users, Conda environments (see [Environment](environment.md)) are a
simpler path for self-managed software.

Next: [Building Software](building.md).
