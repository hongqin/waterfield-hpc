# Job Monitoring, Health Checks, and Recovery

## Monitoring Running Jobs

### Quick status overview
```bash
squeue -u $USER --format="%.10i %.30j %.10T %.10M %.20R %.15P"
```

### Detailed job info (start time, time limit, node list, exit code)
```bash
scontrol show job <job_id>
```

### Check GPU utilization on a running job's node
GPU nodes are GCP cloud-burst VMs — you cannot SSH to them directly. Use `srun --overlap`:
```bash
srun --jobid=<job_id> --overlap nvidia-smi
```

### Check if a job is making progress
Look at the output file for recent writes:
```bash
ls -la <output_file>                    # check file modification time
tail -20 <output_file>                  # check last lines of output
stat --format='%Y' <output_file>        # epoch timestamp of last modification
```

## Detecting Hung Jobs

A job may be hung if it matches any of these patterns:

### 1. Stuck in CONFIGURING
Job allocated a node but never starts running. Common when a cloud-burst node fails to boot.
```bash
squeue -u $USER -t CONFIGURING
```
**Action:** Cancel and resubmit on a different partition or exclude the node.

### 2. Stuck in PENDING — common reasons

| Reason | Meaning | Action |
|---|---|---|
| `(ReqNodeNotAvail)` | Requested partition has no idle nodes | Resubmit on a different partition (e.g., rtxp6000flex-1) |
| `(Priority)` | Queued behind higher-priority jobs | Wait, or try a less-loaded partition |
| `(Dependency)` | Waiting on another job (afterok, afterany) | Check if the dependency job is stuck |
| `(Resources)` | Cluster lacks requested resources | Reduce resource request or try another partition |
| `(ReqNodeNotAvail, May be reserved)` | Nodes are reserved for maintenance | Try another partition |

### 3. Running but no output progress
If the output file hasn't been modified in a suspiciously long time (e.g., >30 min for a GPU training job):
```bash
find <output_dir> -name "*.out" -mmin +30 -exec ls -la {} \;
```
Cross-check with GPU utilization — if GPU is at 0% utilization, the job is likely hung:
```bash
srun --jobid=<job_id> --overlap nvidia-smi --query-gpu=utilization.gpu --format=csv,noheader
```

## Checking Node Health

### Partition overview (all partitions)
```bash
sinfo --format="%.20P %.5a %.10l %.6D %.6t %.20N"
```

### Node state codes

| State | Meaning | Healthy? |
|---|---|---|
| `idle` / `idle~` | Available, no jobs (~ = cloud node, not yet booted) | Yes |
| `mix` | Some GPUs/CPUs in use, some available | Yes |
| `alloc` | Fully allocated | Yes (busy) |
| `down` / `down*` | Node is down | No |
| `drain` / `drain#` | Draining (no new jobs, finishing current) | No |
| `drng` / `drng!` | Draining with jobs running | No |
| `fail` | Node has failed | No |

### Find healthy idle GPU nodes on a specific partition
```bash
sinfo -p <partition> -t idle --format="%.20N %.6t %.10e %.20G"
```

### Find all available GPU partitions with idle nodes
```bash
sinfo -t idle,idle~ --format="%.20P %.6D %.6t" | grep -E "flex|gpu"
```

### Check a specific node's health
```bash
scontrol show node <node_name>
```

## Cancelling Jobs

### Cancel a single job
```bash
scancel <job_id>
```

### Cancel multiple jobs
```bash
scancel <job_id_1> <job_id_2> <job_id_3>
```

### Cancel all PENDING jobs (keep RUNNING ones)
```bash
scancel -u $USER -t PENDING
```

### Cancel by job name pattern
```bash
scancel -u $USER --name=<job_name>
```

## Resubmitting to Healthy Nodes

### Step 1: Identify available partitions
```bash
sinfo -t idle,idle~ --format="%.20P %.6D %.6t" | grep -E "flex|gpu"
```

### Step 2: Choose partition based on GPU needs

| Partition | GPU | VRAM | Notes |
|---|---|---|---|
| `rtxp6000flex-1` | RTX 6000 Ada | 48 GB | Requires PyTorch with sm_120 support |
| `rtxp6000flex-2` | 2× RTX 6000 Ada | 48 GB each | Multi-GPU jobs |
| `h100flex-1` | 1× H100 | 80 GB | Best for large models (Evo2-7B) |
| `h100flex-2` | 2× H100 | 80 GB each | Multi-GPU |
| `h100flex-4` | 4× H100 | 80 GB each | Large-scale training |
| `h200flex-8` | 8× H200 | 141 GB each | Largest available |
| `b200flex-8` | 8× B200 | 192 GB each | Newest generation |

### Step 3: Edit sbatch partition and resubmit
```bash
sed -i 's/--partition=h100flex-1/--partition=rtxp6000flex-1/' script.sbatch
sbatch script.sbatch
```

### Step 4: Exclude specific bad nodes (optional)
If a particular node is problematic:
```bash
sbatch --exclude=wf-a3-highgpu-1g-21 script.sbatch
```

## Partition Compatibility Notes

- **RTX 6000 Ada (Blackwell, sm_120):** Requires PyTorch 2.10+ with CUDA 12.8+. Older PyTorch builds will fail with `no kernel image is available`. The `evo2` conda environment may need updating for RTX 6000 — test with a short interactive job first.
- **H100 (Hopper, sm_90):** Compatible with PyTorch 2.0+ and CUDA 12.x. The standard `evo2` environment works here.
- **H200 / B200:** Same compatibility as H100 / RTX 6000 respectively, but with more VRAM.

## Automated Health Check Script

Run this to get a complete cluster health snapshot:
```bash
echo "=== Your Jobs ===" && \
squeue -u $USER --format="%.10i %.30j %.10T %.10M %.20R %.15P" && \
echo "" && \
echo "=== GPU Partitions with Idle Nodes ===" && \
sinfo -t idle,idle~ --format="%.20P %.6D %.6t" | grep -E "flex|gpu" && \
echo "" && \
echo "=== Unhealthy Nodes ===" && \
sinfo -t down,drain,fail --format="%.20P %.6D %.6t %.20N" 2>/dev/null
```
