# Data Transfer between Systems

## Hostnames
- **Waterfield:** `waterfield.hpc.odu.edu` (or `waterfield-login-1`)
- **Wahab:** `wahab.hpc.odu.edu` (common ODU system)

## Rsync Patterns

### Sync from Waterfield to Local
```bash
rsync -avz --progress johnsmith@waterfield.hpc.odu.edu:~/remote_path/ local_path/
```

### Sync between Waterfield and Wahab
(Run from the system where you have SSH keys set up to reach the other)
```bash
rsync -avz --progress ~/path_on_current_host/ user@other_host:~/path_on_target/
```

## macOS Note
When running `rsync` from a macOS laptop (silverair) using bash 3.2, **avoid multi-line commands with backslashes**. Collapse the command into a single line to prevent errors when pasting.
