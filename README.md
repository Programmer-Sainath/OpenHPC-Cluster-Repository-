# HPC Cluster Configuration Reference

This branch is a **technical reference / showcase of the configuration used while building the cluster**.

It is not connected to GitHub automation and is not intended to deploy the cluster automatically.

The configuration was extracted from the actual terminal build log. Files are grouped by component so that another builder can inspect the Warewulf, Slurm, Munge, NFS, node, firewall, networking and system configuration independently.

## Structure

- `configs/warewulf/` — Warewulf server, image/profile, node and overlay configuration
- `configs/slurm/` — Slurm controller and compute-node configuration
- `configs/munge/` — Munge setup; secret key excluded
- `configs/nfs/` — NFS exports and setup
- `configs/firewall/` — firewalld rules
- `configs/system/` — packages and services
- `configs/networking/` — cluster addressing
- `configs/verification/` — verification commands

## Important

These are reference artifacts from one physical cluster. IP addresses, interface names, MAC addresses, package versions and hardware-specific settings may need to be changed for another cluster.

The original terminal log contained failed commands and troubleshooting attempts. Where a command was corrected later, the reference file uses the successful/final form and notes the correction rather than presenting the failed command as valid configuration.

No real authentication keys or passwords are included.
