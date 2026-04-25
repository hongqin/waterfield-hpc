# Installation Guide for Waterfield

## Installing Miniforge3 Locally
If you don't have a Python environment manager, install Miniforge3 (a lightweight Conda alternative) in your home directory:

```bash
# Download the installer
wget https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh

# Run the installer (follow prompts, usually install to ~/miniforge3)
bash Miniforge3-Linux-x86_64.sh

# Initialize your shell
source ~/miniforge3/bin/activate
conda init
```

## Environment Setup
Waterfield works best with **Miniforge3** for managing Python environments.

```bash
# Recommendation: Use Python 3.11 for AI stacks
conda create -n evo2 python=3.11
conda activate evo2
```

## GPU-Specific Requirements

### RTX 6000 (Blackwell) Compatibility
The RTX 6000 nodes on Waterfield (using `rtxp6000flex` partitions) feature the **Blackwell** architecture (`sm_120`).

- **Issue:** Older PyTorch builds (e.g., those using CUDA 11.x or older 12.x) will fail with: `RuntimeError: CUDA error: no kernel image is available for execution on the device`.
- **Solution:** You MUST use a modern PyTorch version (e.g., `2.10.0+cu128` or newer) that supports the Blackwell architecture.
- **Verification:**
  ```python
  import torch
  print(torch.cuda.get_device_name(0)) # Should show RTX 6000
  print(torch.cuda.get_device_capability(0)) # Should show (12, 0)
  ```

## Evo2 Stack Installation
Validated installation on Hopper nodes (H100).

### Core Libraries
- `pytorch`
- `transformer-engine-torch` (TE)
- `triton`

### Common Issues & Fixes
1. **Mixed pip/conda torch stack:** If `torch` was installed with pip while TE came from conda, remove pip version and reinstall both via conda together.
2. **Flash Attention:** Ensure the node has appropriate CUDA toolkit version (12.x recommended for H100).

## GPU Validation
Run `nvidia-smi` to ensure GPUs are visible.
Run `python -c "import torch; print(torch.cuda.is_available())"` to verify PyTorch integration.
