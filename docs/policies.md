# Policies

These policies come directly from the PARAM Rudra login banner and standard NSM
/ C-DAC HPC practice. Following them keeps the system fair and available for
everyone — and keeps your account in good standing.

## 1. No compute on login nodes

!!! danger "Zero tolerance"
    *"Do not run jobs on the login node, as doing so will result in termination
    without prior notice."*

Login nodes (`login01`, `login02`, `login03`, …) are **shared** and meant only
for:

- Editing files, light scripting
- Compiling / building software (heavy builds → use an interactive job)
- Managing data and submitting jobs

Anything that consumes significant CPU, memory or runs for a long time must go
through **SLURM** (`sbatch` / `salloc` / `srun`). See the
[Batch System](batch.md).

**How to tell:** if `hostname` shows a `login*` node, do **not** launch
simulations, training runs, big `make -j`, or large data crunching there.

## 2. `/scratch` purge — 3 months

!!! warning "Automatic deletion"
    Files in `/scratch` **not accessed in the last 3 months are permanently
    deleted.** `/scratch` is temporary working space, not storage.

- No recovery after purge.
- The clock is based on **access time**, not just existence.
- Move important results **off the cluster** promptly ([Data Management](data.md)).

## 3. Back up your own data

!!! warning "From the banner"
    *"Users are advised to regularly back up their data in `/home` & `/scratch`
    directory."*

Treat the cluster as compute infrastructure, not an archive. Keep authoritative
copies of code (in version control) and results (in institutional storage).

## 4. Use your accounting code

Every job must charge a valid project/allocation with `#SBATCH -A <account>`.
Jobs without a valid account will be rejected or fail. This enables fair-share
scheduling and usage reporting.

## 5. Respect partition limits

| Partition | Max wall time | Max nodes / job |
| --- | --- | --- |
| `cpu` | 4 days | 1 |
| `hm` | 4 days | 8 |
| `gpu` | 6 days | 128 |

- Request only the resources and walltime you need — over-requesting wastes the
  allocation and lengthens your own queue wait.
- Do not attempt to circumvent limits by launching background daemons or
  chaining processes to evade the scheduler.

## 6. Fair-share and good citizenship

- Don't flood the queue with thousands of jobs; use **job arrays** with a
  concurrency cap (`--array=1-N%K`).
- Avoid abusive filesystem patterns (millions of tiny files, tight polling) —
  they degrade the shared filesystem for all users. See
  [parallel-filesystem best practices](data.md#parallel-filesystem-best-practices).
- Release interactive allocations (`exit`) as soon as you are done.

## 7. Storage quotas

| Filesystem | Soft quota |
| --- | --- |
| `/home/$USER` | **50 GB** |
| `/scratch/$USER` | **200 GB** |

Stage inputs and run jobs in `/scratch`; copy results worth keeping back to
`/home` (and off-cluster). Check usage with `lfs quota -h -u $USER /scratch`.
See [Data Management](data.md).

## 8. Security and account hygiene

- Your account is **personal** — never share your password; you are responsible
  for all actions from your account.
- **2FA (Google Authenticator OTP) + CAPTCHA** is mandatory at every login; keep
  your Authenticator device secure.
- Do **not** grant other users permission to your home directory — it can expose
  your files.
- Never store passwords, SSH private keys or access tokens on the cluster or in a
  git repository.
- Only install software from **reliable, safe sources** (ransomware is a real
  risk). Avoid spaces in file/directory names.
- Report suspected compromise or anything strange (slowdowns, missing/corrupted
  files) to [support](support.md) immediately.
- Comply with your institution's and NSM's acceptable-use and data-handling
  rules (including for any sensitive/regulated data).

## 9. Policy updates

!!! note "From the banner"
    *"Any updates regarding policy modifications will be posted here."*

Read the **login banner** each time you connect — it is the authoritative,
up-to-date source for node counts, partition limits and policy changes. This
manual mirrors it but the live banner wins if they ever differ.

---

Questions about a policy? See [FAQ & Support](support.md).
