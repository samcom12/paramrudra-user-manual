# Modules & Conda

PARAM Rudra exposes software through two complementary mechanisms:

- **[Spack](spack.md)** — the *primary* package manager for compilers, MPI,
  libraries and most applications (`spack load ...`).
- **Environment Modules** (`module ...`) — used to enable Spack itself, the
  **Miniconda** Python stack, and pre-built **ML/DL Conda environments**
  (PyTorch, TensorFlow, …) and some applications.

This page covers `module` and Conda. For `spack`, see the
[Spack Packages](spack.md) page.

## Environment Modules

| Command | What it does |
| --- | --- |
| `module avail` | List all available modules |
| `module load <name>` | Load a module (optionally `name/version`) |
| `module unload <name>` | Unload a module |
| `module list` | Show loaded modules |
| `module purge` | Unload everything |
| `module show <name>` | Show what a module sets |

```bash
module avail
module avail 2>&1 | grep -i conda    # module output goes to stderr — use 2>&1
module load spack                    # enable Spack
module load miniconda                # enable the Conda/Python base
module list
```

!!! tip "`module avail` prints to stderr"
    To search it, redirect: `module avail 2>&1 | grep -i <keyword>`.

## Conda / Python (Miniconda)

You typically land in a Conda **base** environment (`(base)` in your prompt).
Load the module explicitly if needed:

```bash
module load miniconda
conda list                # packages in the current environment
conda info --env          # list all environments
```

!!! warning "Don't install into `base`"
    Create your own named environments instead of modifying `base`:
    ```bash
    conda create --name myenv python=3.11
    conda activate myenv
    conda install numpy scipy
    conda deactivate
    ```
    Environments can be large — mind your **50 GB `/home` quota**
    (`du -sh ~/.conda`).

## Pre-built ML/DL Conda environments

PARAM Rudra ships ready-to-use, **GPU-enabled** Conda environments for machine
learning, exposed as modules. Load one with `module load <ENV_NAME>`:

| Category | Environment(s) | Version |
| --- | --- | --- |
| Deep learning | `Tensorflow` / `Tensorflow-gpu` | 2.15.0 |
| | `Pytorch` / `Pytorch-gpu` | 2.2.0 / 2.2.1 |
| | `Keras` / `Keras-gpu` | 3.0.5 |
| | `Theano` / `Theano-gpu` | 1.0.5 |
| | `Caffe` / `Caffe-gpu` | 1.0 |
| Distributed DL | `Horovod` (TensorFlow / PyTorch) | 0.28.1 |
| Data science | `Rapids` | 21.06 |

```bash
module load Pytorch        # activates the PyTorch environment
python -c "import torch; print(torch.__version__, torch.cuda.is_available())"
```

Full ML/DL workflow — building your own environment, `sbatch` templates and
launching Jupyter — is on the [Machine Learning / DL](machine-learning.md) page.

## Reproducibility

Always load **explicit versions** (modules) and **hashes** (Spack) in job scripts
so runs don't silently change when defaults are updated, and record what was
loaded for provenance:

```bash
module list 2>&1          # captured into your SLURM .out file
spack find -l <pkg>       # note the exact build hash
```

Next: [Spack Packages](spack.md) or [Building Software](building.md).
