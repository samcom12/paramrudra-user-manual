# PARAM Rudra (20 PetaFlop) — User Manual

[![Build & Deploy](https://github.com/samcom12/paramrudra-user-manual/actions/workflows/deploy.yml/badge.svg)](https://github.com/samcom12/paramrudra-user-manual/actions/workflows/deploy.yml)
[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](LICENSE)

A free, community-maintained user manual for **PARAM Rudra**, a ~20 PetaFlop
supercomputer under the National Supercomputing Mission (NSM) at C-DAC. The
documentation is structured after the excellent
[JUPITER user documentation](https://apps.fz-juelich.de/jsc/hps/jupiter/index.html)
at Jülich Supercomputing Centre.

📖 **Live site:** https://samcom12.github.io/paramrudra-user-manual/

## What's inside

- **Getting Access** — SSH on port `4422`, key setup, file transfer
- **System Configuration** — 2,906 nodes (CPU / GPU / high-memory), InfiniBand, storage
- **Environment & Modules** — shell, Lmod modules, Conda, `/home` & `/scratch`
- **Building Software** — compilers, MPI, CUDA, CMake/Make, Python, containers
- **Batch System (SLURM)** — partitions (`cpu`/`hm`/`gpu`), scripts, dependencies, arrays
- **GPU Computing** — GRES, multi-GPU, multi-node DDP, MPS
- **Job Script Examples** — 10 ready-to-copy templates
- **Data Management** — `/scratch` 3-month purge, quotas, efficient transfers
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
