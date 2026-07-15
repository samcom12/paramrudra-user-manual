# WRF

**WRF** (Weather Research and Forecasting model) is a high-resolution
atmospheric model widely used for research and operational forecasting, with
flexible physics options. It benefits from the high-memory partition.


### Input

The following example illustrates the running of WRF with the commonly used data set. 

The benchmark input files are pre-installed on the PARAM Rudra system and are available at:

```bash
/home/apps/hpc_inputs/applications/WRF/wrf_run.tar
```

Copy the benchmark dataset to your working directory before running the example:

```bash
cp -r /home/apps/hpc_inputs/applications/WRF/wrf_run.tar .
cd wrf_run.tar
```

## Sample SLURM script

A sample Slurm job script for NWChem is available at:

```bash
/home/apps/hpc_inputs/scripts/wrf.slurm
```

Copy it to your working directory:

or

## Job script

```bash
#!/bin/bash
#SBATCH --job-name="rfm_job"
#SBATCH --ntasks=96
#SBATCH --output=rfm_job.out
#SBATCH --error=rfm_job.err
#SBATCH --exclusive
#SBATCH --partition=cpu
export SPACK_ROOT=/home/apps/spack
. $SPACK_ROOT/share/spack/setup-env.sh
spack load wrf@4.7.1
export OMP_NUM_THREADS=1
#wget https://www2.mmm.ucar.edu/wrf/users/benchmark/v422/v42_bench_conus2.5km.tar.gz
ulimit -s unlimited
tar -xvf /home/apps/hpc_inputs/WRF/wrf_run.tar
cd run
time mpirun -np 96 wrf.exe      
```


## Expected Output

Reference output files for verification are available at:

```bash
/home/apps/hpc_inputs/output/wrf.out
```

## Notes

- `ulimit -s unlimited` is important — WRF uses a large stack.
- WRF is typically built via Spack, e.g. with MPI + NetCDF + HDF5 support. Check
  `spack find -l wrf` for installed builds.
- The related pre-/post-processors **WPS** and **ARWPost** are also available.
- Check `/home/apps/hpc_inputs/WRF` (site-specific) for sample datasets, and the
  official [WRF tutorials](https://www2.mmm.ucar.edu/wrf/users/) for input
  preparation.
