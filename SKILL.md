---
name: waterfield-hpc
description: Specialized workflows and configurations for the Waterfield HPC cluster at ODU. Use for GPU job submission (H100), Slurm templates, environment setup, and data transfer between Waterfield, Wahab, and local systems.
---

# Waterfield HPC Skill

This skill provides procedural knowledge for working effectively on the Waterfield HPC cluster, focusing on GPU workloads and data management.

## Workflows

### 1. Interactive GPU Access
To request an interactive session on an H100 node:
```bash
salloc --partition=h100flex-1 --gres=gpu:1 --time=01:00:00
```
Use `h100flex-2` if `h100flex-1` is full.

### 2. Batch Job Submission
Submit batch jobs using `sbatch`. See [references/slurm-templates.md](references/slurm-templates.md) for templates.

### 3. Environment Setup (Evo2 Stack)
For installing complex AI stacks like Evo2, refer to [references/installation-guide.md](references/installation-guide.md).

### 4. Data Transfer
Sync data between Waterfield and other systems (Wahab, local laptop) using `rsync`. See [references/data-transfer.md](references/data-transfer.md).

### 5. Wall Time Limits and Checkpointing

**Maximum wall time on GPU partitions is 3 days (72 hours):**

| Partition | Max Wall Time |
|---|---|
| `rtxp6000flex-1` | 3-00:00:00 |
| `h100flex-1` | 3-00:00:00 |
| `h100flex-8` | 3-00:00:00 |
| `h200flex-8` | 3-00:00:00 |
| `b200flex-8` | 3-00:00:00 |

Set wall time in SLURM with `#SBATCH --time=3-00:00:00` (max). To request longer wall times, email **rcc@odu.edu** with the partition, requested time, and justification.

**Checkpointing workaround for jobs exceeding 72 hours:**

When a workload needs more than 72 hours, implement checkpointing — periodically save intermediate state and resubmit from the last checkpoint.

```bash
#!/bin/bash
#SBATCH -J checkpoint-example
#SBATCH -p h100flex-1
#SBATCH --gres=gpu:1
#SBATCH --time=3-00:00:00

module load python3

CHECKPOINT_DIR="$HOME/checkpoints/$SLURM_JOB_NAME"
mkdir -p "$CHECKPOINT_DIR"

# Pass checkpoint dir to the script; the script should:
#   1. Check for existing checkpoint and resume if found
#   2. Periodically save state (e.g., every N batches)
crun -p ~/envs/myproject python3 train.py \
    --checkpoint-dir "$CHECKPOINT_DIR" \
    --resume
```

In Python, a minimal checkpoint pattern:

```python
import os, torch

def save_checkpoint(state, checkpoint_dir, filename="checkpoint.pt"):
    torch.save(state, os.path.join(checkpoint_dir, filename))

def load_checkpoint(checkpoint_dir, filename="checkpoint.pt"):
    path = os.path.join(checkpoint_dir, filename)
    if os.path.exists(path):
        return torch.load(path)
    return None
```

For non-training workloads (e.g., embedding extraction), process data in batches and write intermediate `.npz` files so that a resubmitted job can skip already-completed batches.

### 6. Job Monitoring and Hung Job Detection

Monitor running jobs and detect problems early. See [references/job-monitoring.md](references/job-monitoring.md) for full details.

**Quick health check** (jobs + idle partitions + unhealthy nodes):
```bash
squeue -u $USER --format="%.10i %.30j %.10T %.10M %.20R %.15P"
sinfo -t idle,idle~ --format="%.20P %.6D %.6t" | grep -E "flex|gpu"
```

**Check GPU utilization on a running job** (cannot SSH to GPU nodes — they are GCP cloud-burst VMs):
```bash
srun --jobid=<job_id> --overlap nvidia-smi
```

**Detect hung jobs:**
- `CONFIGURING` for >5 min → node failed to boot. Cancel and resubmit.
- `PENDING` with `ReqNodeNotAvail` → partition full. Resubmit on another partition.
- `RUNNING` but output file not modified in >30 min + GPU at 0% → hung process. Cancel and resubmit.

### 7. Cancel and Resubmit to Healthy Nodes

When jobs are stuck on unavailable or unhealthy nodes:

```bash
# 1. Cancel the stuck job
scancel <job_id>

# 2. Find partitions with idle nodes
sinfo -t idle,idle~ --format="%.20P %.6D %.6t" | grep -E "flex|gpu"

# 3. Change partition in sbatch and resubmit
sed -i 's/--partition=OLD/--partition=NEW/' script.sbatch
sbatch script.sbatch
```

**Partition selection guide:**

| Partition | GPU | VRAM | Use when |
|---|---|---|---|
| `rtxp6000flex-1` | RTX 6000 Ada | 48 GB | Model fits in 48 GB; needs PyTorch 2.10+/CUDA 12.8+ (sm_120) |
| `h100flex-1` | H100 | 80 GB | Large models (Evo2-7B full), standard CUDA 12.x |
| `h200flex-8` | 8× H200 | 141 GB each | Multi-GPU or very large models |
| `b200flex-8` | 8× B200 | 192 GB each | Newest, largest VRAM |

**Exclude a bad node:**
```bash
sbatch --exclude=<node_name> script.sbatch
```

### 8. Node Health Check

```bash
# All node states
sinfo --format="%.20P %.5a %.10l %.6D %.6t %.20N"

# Only unhealthy nodes
sinfo -t down,drain,fail --format="%.20P %.6D %.6t %.20N"
```

Node states: `idle`/`idle~` = available, `mix`/`alloc` = busy but healthy, `down`/`drain`/`drng`/`fail` = unhealthy (avoid).

## Core Configurations
- **Partitions:** `h100flex-1`, `h100flex-2`, `h100flex-4`, `h100flex-8`, `rtxp6000flex-1`, `rtxp6000flex-2`, `rtxp6000flex-4`, `rtxp6000flex-8`, `h200flex-8`, `b200flex-8`.
- **GPUs:** NVIDIA H100 (Hopper), RTX 6000 Ada (Blackwell), H200, B200.
- **Python Environment:** Prefers Miniforge3/Conda.
- **GPU node access:** Cannot SSH directly — use `srun --overlap` for diagnostics.
