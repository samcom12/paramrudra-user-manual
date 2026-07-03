# WRF

**WRF** (Weather Research and Forecasting model) is a high-resolution
atmospheric model widely used for research and operational forecasting, with
flexible physics options. It benefits from the high-memory partition.

## Job script

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
ulimit -s unlimited               # WRF needs a large stack

module load apps/wrf-4.5.1        # site application module, if provided
cd run
time mpirun -np $SLURM_NTASKS wrf.exe
```

## Notes

- `ulimit -s unlimited` is important — WRF uses a large stack.
- WRF is typically built via Spack, e.g. with MPI + NetCDF + HDF5 support. Check
  `spack find -l wrf` for installed builds.
- The related pre-/post-processors **WPS** and **ARWPost** are also available.
- Check `/home/apps/hpc_inputs/WRF` (site-specific) for sample datasets, and the
  official [WRF tutorials](https://www2.mmm.ucar.edu/wrf/users/) for input
  preparation.
