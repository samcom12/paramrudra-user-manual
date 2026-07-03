# PARAM Rudra (20 PetaFlop) — User Manual

[![Build & Deploy](https://github.com/samcom12/paramrudra-user-manual/actions/workflows/deploy.yml/badge.svg)](https://github.com/samcom12/paramrudra-user-manual/actions/workflows/deploy.yml)
[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](LICENSE)

A free, community-maintained user manual for **PARAM Rudra**, a ~20 PetaFlop
supercomputer under the National Supercomputing Mission (NSM) at C-DAC. It is
grounded in the official C-DAC PARAM Rudra User Manual plus the live login banner
and SLURM configuration.

📖 **Live site:** https://samcom12.github.io/paramrudra-user-manual/
📄 **PDF:** [Download the full manual](https://samcom12.github.io/paramrudra-user-manual/assets/PARAM-Rudra-20PF-User-Manual.pdf) (auto-generated each build)

## What's inside

- **Getting Access** — SSH on port `4422`, 2FA (Google Authenticator), file transfer
- **System Configuration** — 2,906 nodes (2× Xeon 6240R; A100 GPUs; 768 GB HM), InfiniBand NDR, Lustre
- **Environment / Modules & Conda** — shell, modules, Miniconda, `/home` & `/scratch` (50/200 GB)
- **Spack Packages** — the primary package manager (`spack load`, environments, hashes)
- **Building Software** — Intel/GNU/CUDA/MKL/OpenACC toolchains with real versions
- **Batch System (SLURM)** — partitions (`cpu`/`hm`/`gpu`), QoS, dependencies, arrays, PBS→SLURM
- **GPU Computing** — A100 `sm_80`, GRES, multi-GPU, multi-node DDP, MPS
- **Machine Learning / DL** — pre-built PyTorch/TensorFlow envs, Jupyter via SSH tunnel
- **Applications** — GROMACS, LAMMPS, NAMD, OpenFOAM, WRF, CP2K, NWChem via Spack
- **Job Script Examples** — 10 ready-to-copy templates
- **Debugging** — gdb basics + job-failure diagnosis
- **Data Management** — Lustre striping, `/scratch` 3-month purge, quotas, transfers
- **Accounts & Acknowledgement** — NSM account process, CPU-hour accounting, NSM citation
- **Policies & FAQ**

## Built with

[MkDocs](https://www.mkdocs.org/) + [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/),
published automatically to **GitHub Pages** via GitHub Actions.

## Local preview

Requires Python 3.9+.

```bash
pip install -r requirements.txt
mkdocs serve          # live preview at http://127.0.0.1:8000
mkdocs build          # static site into ./site
```

## C-DAC / NSM logos

The co-branding strip uses the official logo files
(`docs/assets/logo-cdac.png`, `docs/assets/logo-nsm.png`), referenced from
`docs/index.md` and `docs/acknowledgement.md`. The C-DAC and NSM logos are
registered trademarks; use them in accordance with C-DAC/NSM branding
guidelines.

## Contributing

Corrections and additions are welcome — open an issue or a pull request. Every
page on the live site has an ✏️ edit link to its source.

> **Note:** This is a **community/user-maintained** guide, grounded in the live
> PARAM Rudra login banner and SLURM configuration. It is **not** an official
> C-DAC/NSM publication. Verify site-specific values (hardware, quotas, contact
> details) with the official support desk. Never commit secrets (passwords, SSH
> keys, access tokens) to this repository.

## License

[CC BY 4.0](LICENSE) — free to share and adapt with attribution.
