# Data Management

Good data hygiene keeps your jobs fast and your results safe. This page explains
the file systems, the `/scratch` purge policy, quotas, and efficient transfers.

## File systems

Storage is a **Lustre** parallel filesystem (10 PiB primary + 10 PiB archival,
~100 GB/s).

| Path | Role | Soft quota | Backed up? | Purge | Best for |
| --- | --- | --- | --- | --- | --- |
| `/home/$USER` | Home — code, scripts, configs | **50 GB** | Per site policy (keep your own copies too) | No | Small, important, permanent files |
| `/scratch/$USER` | High-performance working space | **200 GB** | **No** | **Files not accessed in 3 months are deleted** | Active job I/O, large temporary data |

!!! danger "Two rules that will save you"
    1. **`/scratch` is not storage.** Files untouched for 3 months are
       **permanently deleted**. There is no recovery.
    2. **Back up what you cannot regenerate.** Neither filesystem is a personal
       archive — copy important results to your institutional storage promptly.

## The recommended data lifecycle

```text
  ┌─────────────┐   stage in    ┌──────────────┐   run    ┌──────────────┐
  │  Your code  │ ────────────▶ │  /scratch    │ ───────▶ │  Results in  │
  │  in /home   │   (rsync/scp) │  (fast I/O)  │  (SLURM) │  /scratch    │
  └─────────────┘               └──────────────┘          └──────┬───────┘
                                                                  │ copy out promptly
                                                                  ▼
                                                        ┌───────────────────┐
                                                        │  Institutional /   │
                                                        │  personal archive  │
                                                        └───────────────────┘
```

1. Keep source and job scripts in `/home`.
2. Stage inputs into `/scratch/$USER/<project>/<run>` and run there.
3. As soon as a run finishes, **move the outputs you need off the cluster**.
4. Delete large intermediates from `/scratch` you no longer need.

## Checking usage and quota

```bash
# Space used by your directories
du -sh /home/$USER            # (can be slow on large trees)
du -sh /scratch/$USER/*

# Free space on the filesystems
df -h /home /scratch

# Quota — try in order (depends on the filesystem type)
quota -s 2>/dev/null || lfs quota -h -u $USER /scratch 2>/dev/null
```

If you hit a quota, jobs can fail to write output. Clean up or move data before
resubmitting.

## Efficient transfers

=== "rsync (recommended)"

    Resumable, transfers only changed files, shows progress:

    ```bash
    # Local -> cluster
    rsync -avP -e "ssh -p 4422" ./project/ \
          samirs@paramrudra.cdacb.in:/scratch/samirs/project/

    # Cluster -> local
    rsync -avP -e "ssh -p 4422" \
          samirs@paramrudra.cdacb.in:/scratch/samirs/project/results/ ./results/
    ```

=== "scp"

    ```bash
    scp -P 4422 input.tar.gz samirs@paramrudra.cdacb.in:/scratch/samirs/
    scp -P 4422 samirs@paramrudra.cdacb.in:/scratch/samirs/out.nc ./
    ```

=== "sftp"

    ```bash
    sftp -P 4422 samirs@paramrudra.cdacb.in
    # then: put localfile / get remotefile / ls / cd
    ```

!!! tip "Bundle many small files before transferring"
    Thousands of tiny files transfer slowly and stress the filesystem. Pack them
    first: `tar czf data.tar.gz data/` then transfer the single archive and
    unpack on the other side. This also helps parallel filesystems that dislike
    huge numbers of small files.

## Parallel-filesystem best practices

`/scratch` is a shared parallel filesystem. To be a good neighbour and get good
performance:

- **Avoid many-small-files workloads.** Prefer fewer, larger files; use formats
  like HDF5/NetCDF that pack data.
- **Don't `ls`/`stat` huge directories repeatedly** — metadata operations are
  expensive and slow everyone down.
- **Write from one rank or use parallel I/O (MPI-IO/HDF5)** rather than every
  rank hammering the same directory.
- **Don't poll files in tight loops** from many processes.
- **Keep job I/O in `/scratch`, not `/home`** — `/home` is for code, not
  high-throughput reads/writes.

## Lustre striping (large files)

`/scratch` is Lustre, which spreads (**stripes**) large files across multiple
storage targets (OSTs) for parallel bandwidth. For **very large files** you can
raise the stripe count so each chunk is roughly 200–300 GB:

| File size | Recommended stripe count | Command (run in the target directory) |
| --- | --- | --- |
| 500 GB – 1 TB | 4 | `lfs setstripe -c 4 .` |
| 1 TB – 2 TB | 8 | `lfs setstripe -c 8 .` |

```bash
lfs getstripe <dir>              # check current striping
lfs setstripe -c 4 <dir>         # stripe new files over 4 OSTs
lfs setstripe -c 4 -s 10m <dir>  # also set 1 MB->10 MB stripe size
lfs quota -h -u $USER /scratch   # your Lustre quota/usage
```

- Striping is set **per directory** and applies to **newly created** files;
  subdirectories inherit their parent's setting at creation time.
- Once a file is created, its stripe count **cannot be changed**.
- `-c 0` = system default (usually 1); `-c -1` = stripe over all OSTs.

!!! note "Don't over-stripe small files"
    Striping helps only genuinely large files. For the typical few-GB file the
    default (single stripe) is best — over-striping many small files hurts
    metadata performance for everyone.

## Housekeeping

```bash
# Find your largest directories under scratch
du -h --max-depth=1 /scratch/$USER | sort -h | tail

# Find files not accessed in >80 days (candidates before the 3-month purge)
find /scratch/$USER -atime +80 -type f -printf '%AY-%Am-%Ad  %p\n' 2>/dev/null | head

# Compress finished runs you still want to keep on-cluster short-term
tar czf run01.tar.gz run01/ && rm -rf run01/
```

!!! warning "`atime` and the purge"
    The purge is based on **access time**. Simply having files sit there does not
    protect them — if they are not read for 3 months they are removed. Move
    anything important **off** the cluster rather than relying on touching files.

Next: review the [Policies](policies.md) that govern fair use.
