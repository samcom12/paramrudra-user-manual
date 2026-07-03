# Batch System (SLURM)

PARAM Rudra uses **SLURM** (Simple Linux Utility for Resource Management) to
schedule work fairly across thousands of nodes. You request resources; SLURM
finds nodes and runs your job when they are free.

!!! danger "Compute belongs in a job, never on the login node"
    *"Do not run jobs on the login node, as doing so will result in termination
    without prior notice."* Use `sbatch`, `srun` or `salloc` — described below.

## Essential commands

| Command | Purpose |
| --- | --- |
| `sbatch script.slurm` | Submit a batch job script |
| `srun <cmd>` | Run a command / launch MPI ranks on allocated nodes |
| `salloc <opts>` | Get an interactive allocation |
| `squeue -u $USER` | Show *your* queued/running jobs |
| `sinfo` | Partition and node status |
| `scancel <jobid>` | Cancel a job |
| `scontrol show job <jobid>` | Full detail of a job |
| `sacct -j <jobid>` | Accounting/history for a finished job |

## Partitions and limits

| Partition | Max wall time | Max nodes / job | Hardware |
| --- | --- | --- | --- |
| `cpu` *(default)* | 4 days (`4-00:00:00`) | 1 | CPU `cbcn*` |
| `hm` | 4 days (`4-00:00:00`) | 8 | High-mem `cbhm*` |
| `gpu` | 6 days (`6-00:00:00`) | 128 | GPU `cbgpu*` |

```bash
sinfo -s                          # quick partition summary
scontrol show partition gpu       # authoritative limits, live
```

!!! note "The `cpu` partition caps a job at **1 node**"
    On this system the `cpu` partition allows a maximum of **1 node per job**.
    Multi-node scaling is done on `gpu` (up to 128 nodes) or `hm` (up to 8
    nodes). Confirm current limits with `scontrol show partition <name>` before
    designing a large run, and contact [support](support.md) if you need a
    multi-node CPU reservation.

## QoS and scheduling policy

| Policy | Value |
| --- | --- |
| Max simultaneous jobs / user | **10** |
| Default max walltime / job | 4 days (`4-00:00:00`) |
| **Default walltime if unspecified** | **2 hours** — always set `--time` explicitly |
| Scheduling | SLURM **backfill** (accurate `--time` improves your turnaround) |

Indicative node × time trade-offs under fair-use policy: an 8-node job for
4 days, a 16-node job for 2 days, or a 32-node job for 1 day (subject to the
per-partition node limits above). Need longer/bigger? Raise a ticket — handled
case-by-case.

!!! tip "Set an accurate walltime"
    If you omit `--time`, your job gets only **2 hours** and will be killed when
    it expires. Requesting *less* time also helps backfill schedule you sooner.

## Your account (`-A`) is mandatory

Every job must charge a project/allocation account:

```bash
#SBATCH -A <your_account>
```

Find your account(s):

```bash
sacctmgr show assoc user=$USER format=account,partition,qos -p
```

## Anatomy of a batch script

Create `job.slurm`:

```bash
#!/bin/bash
#SBATCH --job-name=myjob            # name shown in squeue
#SBATCH --account=myproject         # REQUIRED accounting code (-A)
#SBATCH --partition=cpu             # cpu | hm | gpu
#SBATCH --nodes=1                   # number of nodes
#SBATCH --ntasks-per-node=48        # MPI ranks per node (match core count)
#SBATCH --cpus-per-task=1           # threads per rank (OpenMP)
#SBATCH --time=02:00:00             # walltime HH:MM:SS (<= partition limit)
#SBATCH --output=%x-%j.out          # stdout -> jobname-jobid.out
#SBATCH --error=%x-%j.err           # stderr (omit to merge into .out)

set -euo pipefail

# 1) Reproducible environment
module purge
module load gcc/12 openmpi/4.1.5

# 2) Thread control for OpenMP / hybrid codes
export OMP_NUM_THREADS=${SLURM_CPUS_PER_TASK:-1}

# 3) Work in scratch
cd /scratch/$USER/myproject/run01

# 4) Launch through srun (uses the SLURM allocation)
srun ./app_mpi
```

Submit and watch:

```bash
sbatch job.slurm
squeue -u $USER
tail -f myjob-<jobid>.out
```

## Common `#SBATCH` directives

| Directive | Meaning |
| --- | --- |
| `-J`, `--job-name` | Job name |
| `-A`, `--account` | **Accounting/project code (required)** |
| `-p`, `--partition` | Target partition (`cpu`/`hm`/`gpu`) |
| `-N`, `--nodes` | Number of nodes |
| `--ntasks`, `-n` | Total MPI tasks |
| `--ntasks-per-node` | MPI tasks per node |
| `-c`, `--cpus-per-task` | CPU cores (threads) per task |
| `--gres=gpu:N` | Request N GPUs per node (GPU jobs) |
| `-t`, `--time` | Wall-clock limit (`D-HH:MM:SS` or `HH:MM:SS`) |
| `--mem` / `--mem-per-cpu` | Memory request |
| `-o` / `-e` | stdout / stderr files (`%x`=name, `%j`=jobid, `%N`=node) |
| `--exclusive` | Do not share the node with other jobs |
| `--mail-type` / `--mail-user` | Email on `BEGIN,END,FAIL` |
| `--dependency` | Chain jobs (see below) |
| `--array` | Submit a job array |

!!! tip "Match tasks to cores"
    Set `--ntasks-per-node` to the node's physical core count and
    `--cpus-per-task` for OpenMP threads such that
    `ntasks-per-node × cpus-per-task ≤ cores/node`. Confirm the count with
    `lscpu` on the target partition (see [Configuration](configuration.md)).

## `srun` vs `mpirun`

Inside an allocation, prefer **`srun`** to launch parallel tasks — it inherits
the allocation, handles binding, and integrates with SLURM accounting:

```bash
srun --cpu-bind=cores ./app_mpi
```

If a code insists on `mpirun`, ensure it uses the SLURM-provided host/task
information (most MPI builds detect SLURM automatically).

## Interactive jobs

For debugging, compiling heavy code, or exploratory work, grab an interactive
allocation instead of using the login node:

```bash
# One CPU node for 1 hour
salloc -A myproject -p cpu -N 1 -t 01:00:00

# Then run interactively within the allocation:
srun --pty bash            # shell on the compute node
# ... work ...
exit                       # releases the allocation
```

GPU interactive session:

```bash
salloc -A myproject -p gpu -N 1 --gres=gpu:1 -t 00:30:00
srun --pty bash
nvidia-smi
```

## Job dependencies (workflows)

Chain jobs so one starts only after another finishes:

```bash
jid1=$(sbatch --parsable step1.slurm)
jid2=$(sbatch --parsable --dependency=afterok:$jid1 step2.slurm)
sbatch --dependency=afterok:$jid2 step3.slurm
```

Useful dependency types: `afterok` (prior succeeded), `afterany` (prior ended),
`afternotok` (prior failed), `singleton` (serialise by job name).

## Job arrays (parameter sweeps)

Run many similar tasks with one script:

```bash
#SBATCH --array=1-100%10        # 100 tasks, at most 10 running at once
# ...
INPUT=$(sed -n "${SLURM_ARRAY_TASK_ID}p" inputs.list)
srun ./app "$INPUT"
```

Outputs can use `%A` (array job id) and `%a` (task id): `-o run-%A_%a.out`.

## Monitoring and control

```bash
squeue -u $USER                       # your queue
squeue -j <jobid> -o "%.18i %.9P %.8j %.8T %.10M %.6D %R"
scontrol show job <jobid>             # why is it pending? (see 'Reason')
sstat -j <jobid> --format=JobID,MaxRSS,AveCPU   # live resource use (running)
sacct -j <jobid> --format=JobID,JobName,State,Elapsed,MaxRSS,ExitCode  # after
scancel <jobid>                       # cancel
scancel -u $USER                      # cancel all your jobs
```

Common **pending reasons**: `Priority` / `Resources` (waiting your turn),
`QOSMaxWallDurationPerJobLimit` or `PartitionTimeLimit` (walltime too long),
`AssocMaxJobsLimit` (too many jobs), `ReqNodeNotAvail` (nodes drained/down).

### Holding, releasing and modifying jobs

```bash
scontrol hold <jobid>                       # pause a pending job (JobHeldUser)
scontrol release <jobid>                    # release a held job
scontrol update jobid=<jobid> set TimeLimit=4-00:00:00   # change an attribute
scancel <jobid>                             # cancel
```

### Reservations

If resources have been reserved for your account, use them:

```bash
scontrol show reservation                   # find reservations for your account
sbatch --reservation=<name> job.slurm       # submit into a reservation
```

## Coming from PBS/Torque?

| Concept | PBS/Torque | SLURM |
| --- | --- | --- |
| Directive | `#PBS` | `#SBATCH` |
| Job id | `$PBS_JOBID` | `$SLURM_JOBID` |
| Submit dir | `$PBS_O_WORKDIR` | `$SLURM_SUBMIT_DIR` |
| Node list | `$PBS_NODEFILE` | `$SLURM_JOB_NODELIST` |
| Job name | `-N name` | `--job-name=name` / `-J name` |
| Node count | `-l nodes=N` | `--nodes=N` / `-N N` |
| Cores/node | `-l ppn=N` | `--ntasks-per-node=N` |
| Memory | `-l mem=MB` | `--mem=MB` / `--mem-per-cpu=MB` |
| Walltime | `-l walltime=hh:mm:ss` | `--time=hh:mm:ss` |
| Job array | `-t <spec>` | `--array=<spec>` / `-a` |
| Delay start | `-a <time>` | `--begin=<time>` |

## Estimating start time

```bash
squeue -u $USER --start          # estimated start of pending jobs
sbatch --test-only job.slurm     # would-run check without submitting
```

Next: [GPU Computing](gpu.md) or ready-to-copy [Job Script Examples](examples.md).
