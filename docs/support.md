# FAQ & Support

## Getting help

If something isn't working, gather the details below before contacting the
support desk — it dramatically speeds up resolution.

When reporting an issue, include:

- Your **username** and the **login node** you used (`hostname`).
- The **job id** (`squeue`/`sacct`) and the **exact command** or job script.
- The **full error message** and the relevant `.out`/`.err` file.
- What you **expected** vs. what **happened**, and any recent changes.

Useful diagnostics to attach:

```bash
scontrol show job <jobid>                 # scheduler view / pending reason
sacct -j <jobid> --format=JobID,State,Elapsed,MaxRSS,ExitCode,NodeList
module list 2>&1                          # loaded environment
```

!!! info "Where to reach support"
    - **Ticketing portal:** <https://paramrudra.cdac.in/support> — sign in with
      your **cluster username and password**, pick a Help Topic, and create a
      ticket. You'll get an acknowledgement email with a ticket number; the
      ticket stays open until the issue is resolved.
    - **Email:** `rudrasupport@cdac.in` (technical support, license renewal,
      feedback — include the page number when giving documentation feedback).
    - **Account creation queries:** `nsmsupport@cdac.in`.

### Create a support ticket

1. Open <https://paramrudra.cdac.in/support> and click **Sign In** (top-right).
2. Log in with your cluster **username and password**.
3. Select a **Help Topic** and click **Create Ticket**.
4. Fill in the issue details and submit — you'll receive an acknowledgement
   email with the ticket number.

## Frequently asked questions

??? question "My job was killed and I saw a warning about the login node. Why?"
    Compute-heavy processes are not allowed on login nodes and are terminated
    without notice. Submit through SLURM (`sbatch`/`salloc`/`srun`). See
    [Policies](policies.md) and the [Batch System](batch.md).

??? question "My files in /scratch disappeared."
    `/scratch` is purged: files **not accessed in 3 months** are permanently
    deleted, with no recovery. Move important data off the cluster promptly. See
    [Data Management](data.md).

??? question "Why is my job stuck in PENDING?"
    Run `squeue -j <jobid> -o "%.18i %.9P %.8T %.10M %R"` and read the **Reason**:
    `Priority`/`Resources` means you're waiting your turn; `PartitionTimeLimit`
    means your walltime exceeds the partition max; `AssocMaxJobsLimit` means too
    many of your jobs are queued; `ReqNodeNotAvail` means requested nodes are
    down/drained. See [Monitoring](batch.md#monitoring-and-control).

??? question "`sbatch` rejects my job about the account."
    Every job needs a valid `#SBATCH -A <account>`. List yours with
    `sacctmgr show assoc user=$USER format=account,partition -p`. See
    [Your account](batch.md#your-account--a-is-mandatory).

??? question "I need more than 1 node on the CPU partition."
    The `cpu` partition caps jobs at **1 node**. Multi-node scaling is available
    on `gpu` (≤128 nodes) and `hm` (≤8 nodes). For large multi-node CPU runs,
    contact the support desk to discuss options.

??? question "How do I know how many cores / how much memory a node has?"
    Confirm on an allocated node (not the login node): `lscpu`, `nproc`,
    `free -h`, `numactl --hardware`. See
    [Per-node hardware](configuration.md#node-types-and-per-node-hardware).

??? question "Which GPU do the nodes have, and what `sm_XX` should I compile for?"
    Inside a GPU job run
    `nvidia-smi --query-gpu=name,compute_cap --format=csv` and use the reported
    compute capability as `nvcc -arch=sm_XX`. See [GPU Computing](gpu.md).

??? question "`module load X` says not found."
    Search first: `module avail 2>&1 | grep -i x` or (Lmod)
    `module spider x`. Module output goes to **stderr**, so remember the `2>&1`.
    See [Software Modules](modules.md).

??? question "`scp` fails but `ssh` works (or vice-versa)."
    Port flags differ: `ssh`/`rsync -e ssh` use lowercase `-p 4422`; `scp`/`sftp`
    use uppercase `-P 4422`. See [Getting Access](access.md#copying-files-in-and-out).

??? question "My CUDA program says 'no kernel image is available for execution'."
    The binary was built for a different GPU architecture than the one it ran on.
    Rebuild with the correct `-arch=sm_XX` for the GPU reported by `nvidia-smi`,
    and make sure your framework's CUDA version matches the driver. See
    [Building CUDA](building.md#cuda-gpu-builds).

??? question "Conda is slow / filled my home quota."
    Don't install into `base`; create named envs and consider placing large ones
    under `/scratch`. Check usage with `du -sh ~/.conda`. See
    [Environment](environment.md#conda-and-python).

## Contributing to this manual

This guide is open and community-maintained. Spot something outdated or wrong?

- Open an issue or pull request at
  [github.com/samcom12/paramrudra-user-manual](https://github.com/samcom12/paramrudra-user-manual).
- Each page has an ✏️ **edit** link (top right) that takes you straight to the
  source on GitHub.

!!! warning "Never include secrets in a contribution"
    Do not paste passwords, SSH private keys, or access tokens into issues, pull
    requests, or documentation. If a credential is ever exposed, **rotate it
    immediately**.
