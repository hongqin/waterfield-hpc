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

## Partition Choice Tip
- Use `h100flex` for H100 (Hopper) nodes.
- Use `rtxp6000flex` for RTX 6000 (Blackwell) nodes.
  - **Warning:** Ensure your PyTorch environment is updated to support Blackwell (`sm_120`), otherwise jobs will fail with CUDA kernel errors.

## Monitoring Jobs
- `squeue -u $USER`: Check status of your jobs.
- `scancel <job_id>`: Cancel a job.
- `scontrol show job <job_id>`: Get detailed info about a job.
- `nvidia-smi`: Run inside a job (interactive or batch) to check GPU usage.
