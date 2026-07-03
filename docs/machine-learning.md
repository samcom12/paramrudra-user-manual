# Machine Learning / Deep Learning

Most popular Python ML/DL libraries are pre-installed on PARAM Rudra as
**GPU-enabled Conda environments**, exposed as modules. The 320 GPU nodes
(2× A100 80 GB each) are ideal for training and inference. See also
[GPU Computing](gpu.md).

## Pre-built environments

Load Miniconda, then activate a framework environment via its module:

```bash
module load miniconda
module avail 2>&1 | grep -iE 'pytorch|tensor|keras|caffe|theano|horovod|rapids'
module load Pytorch          # activates the PyTorch (GPU) environment
```

| Category | Module(s) | Version |
| --- | --- | --- |
| Deep learning | `Tensorflow` / `Tensorflow-gpu` | 2.15.0 |
| | `Pytorch` / `Pytorch-gpu` | 2.2.0 / 2.2.1 |
| | `Keras` / `Keras-gpu` | 3.0.5 |
| | `Theano` / `Theano-gpu` | 1.0.5 |
| | `Caffe` / `Caffe-gpu` | 1.0 |
| Distributed DL | `Horovod` (TF / PyTorch) | 0.28.1 |
| Data science | `Rapids` | 21.06 |

Also available: cuDNN, NumPy, SciPy, scikit-learn.

```bash
module load Pytorch
python -c "import torch; print(torch.__version__, torch.cuda.is_available())"
```

Useful Conda commands:

```bash
conda info --env              # list environments
conda list -n <env>           # packages in an environment
conda deactivate              # leave the active environment
```

## Build your own environment

Two options: `pip install` into an existing environment, or build a fresh Conda
environment (recommended for control and fewer version conflicts).

```bash
module load miniconda
conda create --name myenv          # name it anything
conda activate myenv
conda install numpy pytorch        # or: pip install <pkg>
```

!!! warning "Environments eat disk quota"
    Conda environments can consume a large fraction of your **50 GB `/home`
    quota**. Check with `du -sh ~/.conda`, and remove environments you no longer
    need. For very large stacks, consider building under `/scratch` (subject to
    the [3-month purge](data.md)).

## Submitting a DL job with `sbatch`

Activate the environment inside the job, run, then deactivate:

```bash
#!/bin/bash -x
#SBATCH -N 1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=8
#SBATCH --gres=gpu:1              # request a GPU (drop for CPU-only)
#SBATCH -p gpu                    # gpu partition
#SBATCH -A myproject              # your account
#SBATCH -J dl-train
#SBATCH -t 05:00:00
#SBATCH -o %j.out
#SBATCH -e %j.err

cd $SLURM_SUBMIT_DIR
module purge
module load miniconda
conda activate myenv              # or: module load Pytorch
python train.py
conda deactivate
```

For **multi-node, multi-GPU** distributed training (PyTorch DDP / Horovod across
the InfiniBand fabric), see the
[multi-node GPU example](gpu.md#multi-node-gpu-distributed-jobs) and
[example #7](examples.md#7-multi-node-multi-gpu-pytorch-ddp).

## Launching a Jupyter notebook (SSH tunnel)

Run Jupyter on a compute node and open it in your **local** browser via an SSH
tunnel. Never run it on the login node.

**1. Grab a GPU node interactively**

```bash
salloc --nodes=1 --time=1:00:00 --gres=gpu:1 --partition=gpu -A myproject
squeue --me                       # note the assigned node, e.g. cbgpu0044
ssh cbgpu0044                     # hop onto it
```

**2. Start the notebook on the compute node**

```bash
module load miniconda
jupyter notebook --ip=0.0.0.0 --port=8888 --no-browser
# copy the token printed in the terminal
```

**3. From your local machine, open a tunnel through the login node**

```bash
ssh -p 4422 -t -t <username>@paramrudra.cdacb.in \
    -L 8888:localhost:8888 ssh cbgpu0044 -L 8888:localhost:8888
```

**4. Open the notebook locally**

Visit `http://localhost:8888` in your browser and paste the token.

!!! note "Use the port and node SLURM actually assigned"
    Replace `8888` and `cbgpu0044` with your assigned port/node. If the port is
    taken, pick another (e.g. 8889) consistently across all three steps.

Next: [GPU Computing](gpu.md) · [Applications](applications.md).
