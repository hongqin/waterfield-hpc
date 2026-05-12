# Slurm Templates for Waterfield

## H100 GPU Batch Job Template
Use this template for running GPU workloads on H100 nodes.

```bash
#!/bin/bash
#SBATCH --job-name=gpu-job
#SBATCH --output=%x-%j.out
#SBATCH --error=%x-%j.err
#SBATCH --partition=h100flex-1
#SBATCH --gres=gpu:1
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=16
#SBATCH --time=04:00:00

set -euo pipefail

# Initialize conda/miniforge
# source ~/miniforge3/bin/activate your_env

# Run your script
python your_script.py
```

## Wall Time Limits

All GPU partitions have a **maximum wall time of 3 days (72 hours)**:

| Partition | Max Wall Time |
|---|---|
| `rtxp6000flex-1` | 3-00:00:00 |
| `h100flex-1` | 3-00:00:00 |
| `h100flex-8` | 3-00:00:00 |
| `h200flex-8` | 3-00:00:00 |
| `b200flex-8` | 3-00:00:00 |

Set with `#SBATCH --time=3-00:00:00` (max). To request longer limits, email **rcc@odu.edu** with the partition, requested time, and justification.

## Checkpointing Template (for jobs exceeding 72 hours)

When a workload needs more than 72 hours, implement checkpointing — save intermediate state and resubmit from the last checkpoint.

```bash
#!/bin/bash
#SBATCH --job-name=checkpoint-job
#SBATCH --output=%x-%j.out
#SBATCH --error=%x-%j.err
#SBATCH --partition=h100flex-1
#SBATCH --gres=gpu:1
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=16
#SBATCH --time=3-00:00:00

set -euo pipefail

CHECKPOINT_DIR="$HOME/checkpoints/$SLURM_JOB_NAME"
mkdir -p "$CHECKPOINT_DIR"

# The script should:
#   1. Check for existing checkpoint and resume if found
#   2. Periodically save state (e.g., every N batches)
# source ~/miniforge3/bin/activate your_env

python train.py \
    --checkpoint-dir "$CHECKPOINT_DIR" \
    --resume
```

Minimal PyTorch checkpoint pattern:

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

For non-training workloads (e.g., embedding extraction), process data in batches and write intermediate `.npz` files so a resubmitted job can skip completed batches.

## Partition Choice Tip
- Use `h100flex` for H100 (Hopper) nodes.
- Use `rtxp6000flex` for RTX 6000 (Blackwell) nodes.
  - **Warning:** Ensure your PyTorch environment is updated to support Blackwell (`sm_120`), otherwise jobs will fail with CUDA kernel errors.

## Monitoring Jobs
- `squeue -u $USER`: Check status of your jobs.
- `scancel <job_id>`: Cancel a job.
- `scontrol show job <job_id>`: Get detailed info about a job.
- `nvidia-smi`: Run inside a job (interactive or batch) to check GPU usage.
