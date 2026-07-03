# Getting Access

This page explains how to obtain an account and connect to **PARAM Rudra** over
SSH. Access uses **password authentication protected by Two-Factor Authentication
(2FA)** with Google Authenticator, plus a CAPTCHA.

## 1. Get an account

Accounts are issued through the NSM user-creation portal:

1. Go to **<https://services.nsmindia.in/userportal/account>** (linked from the
   *Access to PARAM Systems* section of [nsmindia.in](https://nsmindia.in)).
2. Register with your **official institutional email**, city and institute.
3. Verify your email, complete the form, and upload the required documents
   (ID proof, User Creation Form, etc.).
4. Your HOD/PI verifies the request; the coordinator selects the cluster; higher
   authority grants final approval.
5. On approval you receive an email with your **user id, a temporary password**,
   and the allocated cluster.

Queries: `nsmsupport@cdac.in`. Full details on the
[Accounts & Acknowledgement](acknowledgement.md) page.

## 2. Install Google Authenticator (2FA)

As part of the security policy, **2FA is mandatory for every login**. Before your
first connection, install **Google Authenticator** on your phone:

- Android — Google Play Store
- iOS — Apple App Store

## Connection details

| Item | Value |
| --- | --- |
| Hostname | `paramrudra.cdacb.in` |
| SSH port | **4422** (non-default — always pass `-p 4422`) |
| Login nodes | 14 nodes; you are assigned one (e.g. `login03`) |
| Auth | Username + **CAPTCHA** + **6-digit OTP** (Google Authenticator) + password |
| Support portal | `https://paramrudra.cdac.in/support` |

## 3. First login

=== "Linux / macOS / PowerShell / cmd"

    ```bash
    ssh <username>@paramrudra.cdacb.in -p 4422
    ```

=== "PuTTY (Windows)"

    1. **Host Name:** `paramrudra.cdacb.in`  **Port:** `4422`  **Type:** SSH
    2. Click **Open** and enter your username.

=== "MobaXterm (Windows)"

    1. **Session → SSH**, remote host `paramrudra.cdacb.in`, specify username.
    2. **Port:** `4422`. Start the session.

The first login sequence:

1. Enter the **CAPTCHA** shown in the terminal.
2. A **QR code** is displayed. In Google Authenticator tap **➕ → Scan a QR code**
   and scan it. A new entry appears (e.g. `Cluster: you@login`).
3. Enter the **6-digit OTP** from the app to complete 2FA setup.
4. Enter your **temporary password**. You are then required to **set a new
   password** of your own.

**Every subsequent login:** username → CAPTCHA → current 6-digit OTP → password.

!!! success "You are on a login node"
    Login nodes are shared gateways for **editing, compiling, data transfer and
    job submission** — never for running computations. See [Policies](policies.md).

## Passwords

- Your new password must be strong (upper + lower case, digits, a few special
  characters).
- Passwords are valid for **90 days**; you are prompted to change on expiry.
- Change it any time with:
  ```bash
  passwd
  ```
- **Forgot your password?** Raise a ticket at
  `https://paramrudra.cdac.in/support`; after email verification the support team
  resets it and emails you a temporary password to change on next login.

## Optional: SSH config convenience

You can still shorten the command with a host alias in `~/.ssh/config` on your
local machine (you will still be prompted for CAPTCHA/OTP/password):

```ssh-config
Host rudra
    HostName paramrudra.cdacb.in
    Port 4422
    User <username>
    ServerAliveInterval 60
```

Then just `ssh rudra`.

!!! note "2FA and SSH keys"
    Because interactive 2FA (OTP + CAPTCHA) is enforced at login, plain SSH
    public-key login is generally **not** the access path here — use your
    password + Authenticator OTP as above. If you have a specific need for
    key-based automation, ask the [support desk](support.md) what is permitted.

## Copying files in and out

Use port **4422** for all transfers (SFTP too — not port 22).

```bash
# Upload (note: scp uses -P, capital)
scp -r -P 4422 ./localdir <username>@paramrudra.cdacb.in:/home/<username>/

# Download results
scp -P 4422 <username>@paramrudra.cdacb.in:/scratch/<username>/out.nc ./

# Efficient, resumable sync (rsync/ssh use -p, lowercase)
rsync -avP -e "ssh -p 4422" ./project/ <username>@paramrudra.cdacb.in:/scratch/<username>/project/
```

GUI tools that work well on Windows:

- **WinSCP** — drag-and-drop file transfer (set the port to **4422**).
- **MobaXterm** — integrated SSH terminal + SFTP browser.

!!! note "`scp` uses `-P`, `ssh`/`rsync` use `-p`"
    A common trip-up: for `scp`/`sftp` the port flag is uppercase **`-P 4422`**;
    for `ssh` and `rsync -e ssh` it is lowercase **`-p 4422`**.

## Troubleshooting login

| Symptom | Likely cause / fix |
| --- | --- |
| `Connection timed out` | Port 4422 blocked by your firewall/VPN, or wrong network. Ensure your network/firewall allows access to the HPC system. |
| OTP rejected | Phone clock drift, or wrong Authenticator entry. Ensure automatic time is on; use the entry created for this cluster. |
| Password rejected on first login | Use the exact temporary password from the welcome email; you will be forced to change it. |
| Locked out / lost 2FA device | Raise a ticket at `https://paramrudra.cdac.in/support`. |
| `Permission denied` repeatedly | Account may be expired (90-day password) — reset via ticket. |

Next: set up your working [Environment](environment.md).
