# OpenHPC Cluster Infrastructure

> A student-led High Performance Computing (HPC) project built using repurposed desktop hardware, Rocky Linux 9, and the OpenHPC software ecosystem.
>
> The project focuses on cluster infrastructure, node provisioning, workload scheduling, networking, shared storage, and parallel computing.
>
> Developed as a long-term educational resource, it aims to make practical scientific computing and HPC system administration more accessible within a school environment.

---

## Overview

The **OpenHPC Cluster Infrastructure** project documents the design, configuration, and development of a multi-node High Performance Computing cluster. It combines established open-source HPC technologies with repurposed hardware to create a platform for learning distributed computing and supporting computational workloads.

The infrastructure consists of five desktop PCs configured as one admin node and four compute nodes. The admin node provides centralized cluster management, while the compute nodes are intended to execute scheduled parallel and scientific workloads.

The systems comprise a mix of Lenovo ThinkCentre E71, E73, and M71 models. They are interconnected through a TP-Link LS1016G unmanaged Gigabit Ethernet switch.

This repository also serves as a technical reference for the actual cluster configuration, documenting the software stack and component-specific configuration files used during deployment.


## Team

| Member                          | Role and Responsibilities                                                  |
| ------------------------------- | -------------------------------------------------------------------------- |
| **Samuel Durai**                | Project Director/Orginator · Lead Systems Architect · Linux Administration |
| **Shanmukha Sainath Kasireddy** | Project Director · Lead HPC Infrastructure Engineer · Simulations          |
| **Eric Bipin Philip**           | Software · Infrastructure                                                  |
| **Aayush Pal**                  | Simulations · Performance Analysis                                         |
| **Harshit Srinivasan**          | Documentation                                                              |
| **Sanskriti Srinivasan**        | Testing · Verification                                                     |

## Hardware Architecture

The cluster is built from five repurposed Lenovo ThinkCentre desktop PCs, using a dedicated management node and four compute nodes connected through an unmanaged Gigabit Ethernet switch.

### Admin Node

| Component    | Specification                         |
| ------------ | ------------------------------------- |
| Quantity     | 1                                     |
| Processor    | Intel Core i5-4460                    |
| Memory       | 16 GB DDR3                            |
| Storage      | 128 GB SSD                            |
| Network      | Gigabit Ethernet                      |
| Primary Role | Cluster management and administration |

The admin node hosts the cluster management services, including the Slurm controller, Warewulf, NFS, and SSH, as configured for the deployment.

### Compute Nodes

| Component               | Specification                               |
| ----------------------- | ------------------------------------------- |
| Quantity                | 4                                           |
| Processor               | Intel Core i5-4460                          |
| Memory                  | 16 GB DDR3 per node                         |
| Combined Compute Memory | 64 GB DDR3                                  |
| Network                 | Gigabit Ethernet                            |
| Primary Role            | Scheduled parallel and scientific workloads |

### Aggregate Resources

| Resource                | Specification                        |
| ----------------------- | ------------------------------------ |
| Total Nodes             | 5                                    |
| Admin Nodes             | 1                                    |
| Compute Nodes           | 4                                    |
| Desktop Models          | Lenovo ThinkCentre E71, E73, and M71 |
| Total Installed Memory  | 80 GB DDR3                           |
| Combined Compute Memory | 64 GB DDR3                           |
| Network Switch          | TP-Link LS1016G                      |
| Switch Type             | Unmanaged                            |
| Network Infrastructure  | Gigabit Ethernet                     |

*Memory totals represent installed physical RAM across all five nodes, not memory available to a single job.*

## Software Stack

The infrastructure uses open-source tools for provisioning, workload scheduling, authentication, networking, and shared file access.

| Component                 | Technology                       | Purpose                                                      |
| ------------------------- | -------------------------------- | ------------------------------------------------------------ |
| Operating System          | Rocky Linux 9                    | Linux environment for cluster services and compute workloads |
| HPC Software Distribution | OpenHPC                          | HPC software integration and deployment tooling              |
| Node Provisioning         | Warewulf                         | Compute-node image and provisioning management               |
| Workload Manager          | Slurm                            | Job scheduling and allocation of cluster resources           |
| Authentication            | Munge                            | Authentication for Slurm-related services                    |
| Message Passing           | OpenMPI                          | MPI-based parallel computing                                 |
| Shared File Access        | NFS                              | Network-based file sharing                                   |
| Remote Administration     | SSH                              | Secure remote access and administration                      |
| Network Management        | Linux networking and `firewalld` | Interface configuration, addressing, and traffic control     |

*The table describes the roles of the software components; successful operation of each service depends on the deployed configuration and verification.*

## Configuration Reference

The repository contains component-specific configuration files extracted and organized from the cluster's actual terminal build log. These files provide a technical record of the deployment and a reference for builders investigating similar HPC environments.

### Configuration Directory

| Directory               | Contents                                                          |
| ----------------------- | ----------------------------------------------------------------- |
| `configs/warewulf/`     | Server, image, profile, node, and overlay configuration           |
| `configs/slurm/`        | Slurm controller and compute-node configuration                   |
| `configs/munge/`        | Munge setup and authentication configuration; secret key excluded |
| `configs/nfs/`          | NFS setup, exports, and shared-storage configuration              |
| `configs/firewall/`     | `firewalld` rules and firewall configuration                      |
| `configs/networking/`   | Cluster addressing and network-interface configuration            |
| `configs/system/`       | System packages, services, and related settings                   |
| `configs/verification/` | Commands used to inspect and verify cluster components            |

The configuration files are intended for technical inspection, learning, troubleshooting, and adaptation. They are **not a one-command installer or a complete automated deployment system**.

### Configuration Notes

* **Hardware-specific settings:** IP addresses, interface names, MAC addresses, hostnames, and other environment-dependent values may need to be changed for another deployment.
* **Version compatibility:** Package versions, configuration syntax, and service behavior may differ between software releases.
* **Troubleshooting history:** Where identifiable, corrected commands and final configuration forms are documented instead of presenting failed terminal attempts as valid configuration.
* **Security:** Real passwords and authentication secrets are excluded. A new deployment must generate and securely distribute its own Munge key.
* **Verification:** Configuration examples should be reviewed and tested against the target environment before use.

## Simulation Workloads

Scientific applications and simulation-related work are maintained separately to keep the infrastructure documentation focused on cluster deployment and management.

See the [**Simulation Workloads branch**](../../tree/Simulation-Workloads) for the project's computational workloads and related development.

The intended application areas include computational physics, mathematical modelling, numerical methods, and Computational Fluid Dynamics (CFD).

## Development Roadmap

* Complete and validate the deployment of all four compute nodes.
* Verify node provisioning, Slurm scheduling, and MPI execution across compute nodes.
* Run reproducible performance benchmarks and document the results.
* Expand scientific workload testing and performance analysis.
* Improve configuration documentation and troubleshooting guides.
* Evaluate future storage and compute-capacity expansion.

## Acknowledgements

We gratefully acknowledge **DPS-Modern Indian School** for providing the hardware and access to the school's robotics lab for the development of this cluster.

We also acknowledge the **OpenHPC community** and the broader open-source ecosystem for the software, tools, and documentation that support accessible High Performance Computing.

## License

See the [`LICENSE`](LICENSE) file for the terms governing the use, modification, and distribution of this repository.
