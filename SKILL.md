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

## Core Configurations
- **Partitions:** `h100flex-1`, `h100flex-2`, `gpu`.
- **GPUs:** NVIDIA H100 (Hopper).
- **Python Environment:** Prefers Miniforge3/Conda.
