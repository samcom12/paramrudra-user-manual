# Getting Access

This page explains how to obtain an account and connect to **PARAM Rudra** over
SSH.

## Prerequisites

- An **approved user account** on PARAM Rudra (issued by the C-DAC / NSM HPC
  team after your project or institutional request is granted).
- An **SSH client**:
    - **Linux / macOS** — OpenSSH is pre-installed (`ssh`).
    - **Windows** — OpenSSH (built into Windows 10/11, usable from PowerShell or
      Git Bash) or [PuTTY](https://www.putty.org/).
- Network reachability to `paramrudra.cdacb.in` on **TCP port 4422**. If you are
  off the institutional network you may need the approved VPN.

## Connection details

| Item | Value |
| --- | --- |
| Hostname | `paramrudra.cdacb.in` |
| SSH port | **4422** (non-default — you must pass `-p 4422`) |
| Login nodes | `login01`, `login02`, `login03`, … (assigned on connect) |
| Username | Your C-DAC-issued user id (examples below use `samirs`) |

## First login

=== "Linux / macOS / Git Bash"

    ```bash
    ssh samirs@paramrudra.cdacb.in -p 4422
    ```

=== "Windows PowerShell"

    ```powershell
    ssh samirs@paramrudra.cdacb.in -p 4422
    ```

=== "PuTTY"

    1. **Host Name:** `paramrudra.cdacb.in`
    2. **Port:** `4422`
    3. **Connection type:** SSH
    4. (Optional) Load your private key under *Connection → SSH → Auth →
       Credentials*.
    5. Click **Open** and log in with your username.

On success you will land on one of the login nodes (for example `login03`) and
see the system banner with node counts, partition limits and usage policies.

!!! success "You are on a login node"
    Login nodes are for **editing files, compiling, managing data and submitting
    jobs** — not for running computations. See [Policies](policies.md).

## Set up SSH keys (recommended)

Key-based authentication is more secure and convenient than typing a password
each time.

### 1. Generate a key pair (on your local machine)

Use a modern Ed25519 key protected by a strong passphrase:

```bash
ssh-keygen -a 100 -t ed25519 -C "you@example.org" -f ~/.ssh/id_ed25519_rudra
```

This creates:

- `~/.ssh/id_ed25519_rudra` — your **private** key (never share it).
- `~/.ssh/id_ed25519_rudra.pub` — your **public** key (safe to copy to servers).

### 2. Install the public key on PARAM Rudra

```bash
ssh-copy-id -i ~/.ssh/id_ed25519_rudra.pub -p 4422 samirs@paramrudra.cdacb.in
```

If `ssh-copy-id` is unavailable (e.g. on Windows), append the public key
manually:

```bash
cat ~/.ssh/id_ed25519_rudra.pub | ssh -p 4422 samirs@paramrudra.cdacb.in \
  "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

### 3. Add a convenient host alias

Create/edit `~/.ssh/config` on your local machine:

```ssh-config
Host rudra
    HostName paramrudra.cdacb.in
    Port 4422
    User samirs
    IdentityFile ~/.ssh/id_ed25519_rudra
    IdentitiesOnly yes
    # Reuse one connection for many sessions (faster, fewer prompts)
    ControlMaster auto
    ControlPath ~/.ssh/cm-%r@%h:%p
    ControlPersist 10m
    ServerAliveInterval 60
```

Now you can simply run:

```bash
ssh rudra
```

!!! tip "Protect your private key"
    - Always set a passphrase; unlock it once per session with `ssh-agent`:
      ```bash
      eval "$(ssh-agent -s)" && ssh-add ~/.ssh/id_ed25519_rudra
      ```
    - Keep permissions tight: `chmod 700 ~/.ssh` and `chmod 600 ~/.ssh/id_ed25519_rudra`.
    - Never copy a private key onto the cluster or into a git repository.

## Copying files in and out

Basic transfers (see [Data Management](data.md) for large/parallel transfers):

```bash
# Upload a file to your home directory
scp -P 4422 ./input.dat samirs@paramrudra.cdacb.in:/home/samirs/

# Download results
scp -P 4422 samirs@paramrudra.cdacb.in:/home/samirs/run/out.nc ./

# Sync a directory efficiently (resumable, only changed files)
rsync -avP -e "ssh -p 4422" ./project/ samirs@paramrudra.cdacb.in:/scratch/samirs/project/
```

!!! note "`scp` uses `-P` (capital), `ssh` uses `-p` (lowercase)"
    A very common source of confusion. For `scp`/`sftp` the port flag is
    **`-P 4422`**; for `ssh`/`rsync -e ssh` it is **`-p 4422`**.

## Troubleshooting login

| Symptom | Likely cause / fix |
| --- | --- |
| `Connection timed out` | Port 4422 blocked by your firewall/VPN, or wrong network. Confirm you can reach the host and that VPN is connected. |
| `Permission denied (publickey,password)` | Wrong username, key not installed, or key permissions too open. Re-check steps 1–3 above. |
| `Too many authentication failures` | SSH is offering too many keys. Add `IdentitiesOnly yes` (as in the config above) or use `-o IdentitiesOnly=yes`. |
| Host key changed warning | Only expected after a legitimate rebuild. If unsure, **contact support** before removing the old key — do not blindly `ssh-keygen -R`. |
| Password rejected repeatedly | Account may be locked or expired. Contact the [support desk](support.md). |

Next: set up your working [Environment](environment.md).
