# System Configuration

PARAM Rudra is a ~20 PetaFlop heterogeneous cluster combining CPU-only,
high-memory and GPU-accelerated nodes on a high-speed InfiniBand fabric.

## Node inventory

| Node class | Count | Prefix | Partition | Typical use |
| --- | --- | --- | --- | --- |
| CPU-only | 2,266 | `cbcn####` | `cpu` | MPI/OpenMP simulation, throughput jobs |
| GPU-accelerated | 320 | `cbgpu####` | `gpu` | CUDA, AI/ML training & inference, GPU-HPC |
| High-memory | 320 | `cbhm####` | `hm` | Large-memory pre/post-processing, genomics, in-memory analytics |
| **Total** | **2,906** | — | — | — |

!!! info "Node naming convention"
    Node hostnames encode their class: `cb` (C-DAC Bengaluru) + class
    (`cn` = compute node, `gpu` = GPU node, `hm` = high-memory) + a 4-digit
    index, e.g. `cbcn0001`, `cbgpu0044`, `cbhm0193`.

## Interconnect

All nodes are connected over a **high-speed InfiniBand (IB)** network providing
low-latency, high-bandwidth communication for MPI and for parallel storage
access. This is what allows tightly-coupled jobs to scale across many nodes.

## Storage layout

| Path | Purpose | Persistence | Notes |
| --- | --- | --- | --- |
| `/home/<user>` | Source code, scripts, small inputs, configs | Long-lived, **quota-limited** | Backed up per site policy — still keep your own copies. |
| `/scratch/<user>` | High-performance working space for running jobs | **Purged**: files not accessed in 3 months are deleted | Not backed up. Stage data here, run, then move results off. |

See [Data Management](data.md) for quotas, the purge policy and good practice.

## Per-node hardware

!!! note "Confirm exact specs on the node"
    Exact CPU model, core count, memory and GPU model are site-specific and may
    differ per hardware batch. Always confirm on an **allocated compute node**
    (not the login node) with the commands below rather than assuming.

Check CPU layout:

```bash
lscpu                 # sockets, cores per socket, threads, model name
nproc                 # logical CPUs visible to this job
free -h               # total / available memory
numactl --hardware    # NUMA nodes and memory per NUMA domain
```

Check GPU layout (on a `gpu` node inside a job):

```bash
nvidia-smi                        # GPU model, count, memory, driver
nvidia-smi topo -m                # GPU/NIC topology (NVLink, PCIe)
nvidia-smi --query-gpu=name,memory.total --format=csv
```

A convenient one-liner to record a node's profile at the top of a job:

```bash
echo "Node: $(hostname)"; lscpu | grep -E 'Model name|^CPU\(s\)|Socket|NUMA node\(s\)'; free -h | head -2
```

## Partitions (queues)

Six partitions are configured. The three primary user partitions and their hard
limits (from the live SLURM configuration) are:

| Partition | Max wall time | Max nodes / job | Target hardware |
| --- | --- | --- | --- |
| `cpu` *(default)* | `4-00:00:00` (4 days) | 1 | CPU-only `cbcn*` nodes |
| `hm` | `4-00:00:00` (4 days) | 8 | High-memory `cbhm*` nodes |
| `gpu` | `6-00:00:00` (6 days) | 128 | GPU `cbgpu*` nodes |

Additional partitions (e.g. benchmarking pools such as `hpl02`) may appear in
`sinfo` and are generally reserved for system/administrative use.

!!! tip "See the authoritative limits live"
    Partition definitions can change. Query them directly:
    ```bash
    sinfo -s                                   # summary: partitions and node states
    scontrol show partition cpu                # full limits for a partition
    sinfo -o "%P %l %D %m %c"                   # partition, timelimit, nodes, mem, cores
    ```

Continue to the [Environment](environment.md) page to set up your shell and
modules, or jump to the [Batch System](batch.md) to start submitting jobs.
