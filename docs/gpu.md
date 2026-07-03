# GPU Computing

PARAM Rudra has **320 GPU-accelerated nodes** (`cbgpu*`) in the `gpu` partition.
Each node carries **2× NVIDIA A100 (80 GB HBM2e, Ampere, `sm_80`)** — 6,912 CUDA
cores per GPU — on top of 2× Intel Xeon Gold 6240R (48 cores). The `gpu`
partition allows up to **128 nodes per job** and a **6-day** wall-time limit —
ideal for large AI/ML training and GPU-accelerated HPC.

!!! info "A100 quick facts"
    Compile CUDA for the A100 with `nvcc -arch=sm_80` (see
    [Building CUDA](building.md#cuda-gpu-builds)). Request up to 2 GPUs per node
    with `--gres=gpu:1` or `--gres=gpu:2`.

## Requesting GPUs

GPUs are requested as a **generic resource (GRES)**:

```bash
#SBATCH --partition=gpu
#SBATCH --gres=gpu:1            # GPUs per node (set to the node's GPU count for full-node)
```

Minimal GPU job script:

```bash
#!/bin/bash
#SBATCH --job-name=gpu-test
#SBATCH --account=myproject
#SBATCH --partition=gpu
#SBATCH --nodes=1
#SBATCH --gres=gpu:1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=8
#SBATCH --time=00:30:00
#SBATCH --output=%x-%j.out

module purge
module load cuda            # version per `module avail`

nvidia-smi                  # show the GPU(s) assigned to this job
srun ./app_gpu
```

## Discover the GPU hardware

Always confirm the exact GPU model and capability inside a job (do not assume):

```bash
nvidia-smi
nvidia-smi --query-gpu=index,name,memory.total,compute_cap --format=csv
nvidia-smi topo -m          # NVLink / PCIe topology between GPUs and NICs
```

Use the reported `compute_cap` to pick the right `nvcc -arch=sm_XX` when
[building CUDA code](building.md#cuda-gpu-builds).

## GPU visibility

SLURM sets `CUDA_VISIBLE_DEVICES` to the GPUs allocated to your job. Your process
sees them as indices `0..N-1`.

```bash
echo $CUDA_VISIBLE_DEVICES        # e.g. 0  or  0,1,2,3
```

!!! warning "Do not hard-code physical GPU ids"
    Let SLURM assign GPUs and read `CUDA_VISIBLE_DEVICES`. Manually overriding it
    can collide with other jobs sharing a node.

## Multi-GPU on one node

Request several GPUs and one task per GPU:

```bash
#SBATCH --nodes=1
#SBATCH --gres=gpu:4
#SBATCH --ntasks-per-node=4
#SBATCH --cpus-per-task=8

srun --gpus-per-task=1 ./app_gpu     # one rank drives one GPU
```

## Multi-node GPU (distributed) jobs

The `gpu` partition scales to **128 nodes**. For frameworks like PyTorch DDP,
use `srun` to launch one process per GPU across nodes:

```bash
#!/bin/bash
#SBATCH --job-name=ddp
#SBATCH --account=myproject
#SBATCH --partition=gpu
#SBATCH --nodes=8
#SBATCH --gres=gpu:4                 # GPUs per node
#SBATCH --ntasks-per-node=4          # one task per GPU
#SBATCH --cpus-per-task=8
#SBATCH --time=12:00:00
#SBATCH --output=%x-%j.out

module purge
module load cuda

# Rendezvous info for torch.distributed
export MASTER_ADDR=$(scontrol show hostnames "$SLURM_JOB_NODELIST" | head -n1)
export MASTER_PORT=29500
export WORLD_SIZE=$(( SLURM_NNODES * SLURM_NTASKS_PER_NODE ))

srun python train_ddp.py             # each rank reads SLURM_PROCID / LOCAL rank
```

For high-performance collective communication across nodes, ensure **NCCL** uses
the InfiniBand fabric:

```bash
export NCCL_IB_HCA=mlx5          # match the site's HCA naming (see `ibstat`)
# export NCCL_DEBUG=INFO         # enable temporarily to diagnose comms
```

Confirm the correct NCCL/IB settings with [support](support.md) — HCA names are
site-specific.

## CUDA MPS (sharing one GPU among ranks)

If several MPI ranks must share a single GPU efficiently, the **Multi-Process
Service (MPS)** can improve throughput:

```bash
export CUDA_MPS_PIPE_DIRECTORY=/tmp/nvidia-mps-$SLURM_JOB_ID
export CUDA_MPS_LOG_DIRECTORY=/tmp/nvidia-mps-log-$SLURM_JOB_ID
nvidia-cuda-mps-control -d          # start the MPS daemon
# ... run your ranks ...
echo quit | nvidia-cuda-mps-control # stop MPS at the end
```

Use MPS only when profiling shows a single kernel underutilises the GPU;
one-rank-per-GPU is simpler and usually best for HPC.

## AI/ML quick start (PyTorch)

```bash
salloc -A myproject -p gpu -N 1 --gres=gpu:1 -c 8 -t 00:30:00
srun --pty bash
module load cuda
conda activate myenv                 # env with a CUDA-enabled PyTorch
python - <<'PY'
import torch
print("CUDA available:", torch.cuda.is_available())
print("Device:", torch.cuda.get_device_name(0) if torch.cuda.is_available() else "CPU")
PY
```

!!! tip "Match PyTorch's CUDA build to the toolkit"
    Install a PyTorch wheel whose CUDA version is compatible with the driver on
    the GPU nodes (`nvidia-smi` shows the driver/CUDA version). Mismatches are
    the most common cause of `CUDA error: no kernel image is available`.

For pre-built ML/DL Conda environments and launching a **Jupyter notebook** on a
GPU node via SSH tunnel, see the [Machine Learning / DL](machine-learning.md)
page. Ready-to-run scripts are on [Job Script Examples](examples.md).
