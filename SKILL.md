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

## Core Configurations
- **Partitions:** `h100flex-1`, `h100flex-2`, `gpu`.
- **GPUs:** NVIDIA H100 (Hopper).
- **Python Environment:** Prefers Miniforge3/Conda.
