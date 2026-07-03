# Environment

This page covers your shell, the module system, your file systems, and the
default Conda base environment you land in.

## Your shell

The default login shell is **bash**. When you log in you will typically see a
prompt like:

```text
(base) [samirs@login03 ~]$
```

The `(base)` prefix indicates that a **Conda base environment** is active by
default (see [Conda / Python](#conda-and-python) below).

Customise your environment through the usual bash startup files in `/home`:

- `~/.bashrc` — interactive shell settings, aliases, `module load` lines you
  want every session.
- `~/.bash_profile` — login-shell settings (often just sources `~/.bashrc`).

!!! warning "Keep startup files fast and safe"
    Avoid heavy commands, network calls, or `module load` of large stacks in
    `~/.bashrc` — they run on **every** shell, including inside every job step
    and can slow down or break batch jobs. Prefer loading modules explicitly in
    your job scripts.

## File systems

| Variable / Path | Use it for | Watch out for |
| --- | --- | --- |
| `/home/$USER` (`$HOME`) | Code, scripts, configs, small files | Quota-limited; not for heavy job I/O |
| `/scratch/$USER` | Fast working space for running jobs | **3-month access-time purge**; not backed up |

Recommended pattern:

```bash
# 1. Keep your code and job scripts in $HOME
cd $HOME/myproject

# 2. Stage inputs and run in /scratch (fast parallel filesystem)
mkdir -p /scratch/$USER/myproject/run01
cd /scratch/$USER/myproject/run01

# 3. After the run, move important results to safe long-term storage
```

Check your usage:

```bash
# Disk usage of a directory
du -sh /scratch/$USER/*

# Quota (command may vary by site; try these)
quota -s 2>/dev/null || lfs quota -h -u $USER /scratch 2>/dev/null || df -h $HOME /scratch
```

!!! danger "Back up your data"
    `/scratch` is purged and neither filesystem should be treated as an archive.
    Regularly copy results you cannot regenerate to your own institutional
    storage. See [Data Management](data.md).

## Identify where you are

```bash
hostname          # e.g. login03, cbcn0421, cbgpu0044
echo $SLURM_JOB_ID    # non-empty only inside a job
groups            # your groups / possible accounts
sacctmgr show assoc user=$USER format=account,partition -p 2>/dev/null  # your accounts
```

## Environment modules

Software is provided through **environment modules** so you can select specific
compilers, MPI libraries and applications without conflicts.

```bash
module avail                 # list everything available
module avail 2>&1 | grep -i openmpi   # search (module output goes to stderr)
module load gcc/12           # load a specific version
module list                  # what is currently loaded
module unload gcc/12         # remove one module
module purge                 # remove all modules (clean slate)
module show openmpi          # see what a module sets (paths, vars)
```

The module system is covered in depth on the [Software Modules](modules.md)
page.

!!! tip "`module avail` prints to stderr"
    To search it, redirect stderr into the pipe: `module avail 2>&1 | grep -i cuda`.

## Conda and Python

You land in a Conda **base** environment (`(base)` in your prompt). Do **not**
install packages into `base` — create your own named environments instead:

```bash
# Create a project environment
conda create -n myenv python=3.11 numpy scipy
conda activate myenv

# ... work ...

conda deactivate
```

If you prefer not to start in `base` automatically:

```bash
conda config --set auto_activate_base false   # takes effect next login
```

!!! warning "Do not build Conda environments on the login node under load"
    Large `conda`/`pip` installs are I/O and CPU heavy. For big environments,
    do the install inside an [interactive job](batch.md#interactive-jobs) or a
    short batch job, and consider placing large environments under `/scratch`
    (mindful of the purge policy) rather than filling your `$HOME` quota.

See [Building Software](building.md) for compilers, MPI and CUDA, and for using
`pip`/`venv` and Julia.

## Transferring files

Quick reference (full details on the [Data Management](data.md) page):

```bash
scp -P 4422  file  samirs@paramrudra.cdacb.in:/scratch/samirs/
rsync -avP -e "ssh -p 4422"  ./dir/  samirs@paramrudra.cdacb.in:/scratch/samirs/dir/
```

## Using Git on the cluster

Git is available for pulling your code onto the system:

```bash
module avail 2>&1 | grep -i git    # a newer git may be provided as a module
git clone https://github.com/<you>/<repo>.git
```

!!! danger "Never store credentials or tokens in the repo or in plain files"
    Do not commit SSH private keys, passwords or Personal Access Tokens. Use SSH
    keys or a short-lived credential helper, and keep secrets out of `$HOME`
    files that might be shared or backed up.
