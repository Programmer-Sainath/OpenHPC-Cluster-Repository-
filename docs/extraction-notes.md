# Extraction notes

Source: `Admin node Complete config(Warewulfd, Slurm and Munge), Added compute nodes+Config.txt` (4,714 terminal-log lines).

The goal of this branch is to preserve the useful technical configuration while making it readable to external viewers.

## What was separated

- Warewulf server configuration
- Warewulf node definitions
- Rocky Linux 9 image/profile configuration
- Warewulf overlays
- Slurm controller configuration
- slurmd compute-node configuration
- Munge setup
- NFS exports
- firewalld rules
- package/service setup
- cluster network layout
- verification commands

## What was not copied

- Passwords and authentication keys
- Generated `.img` / `.img.gz` Warewulf artifacts
- Large package-manager output
- Repetitive command output
- Failed commands when a later working command exists

The raw build log remains the evidence/history; these files are the cleaned reference layer on top of it.
