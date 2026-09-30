# OpenHPC-Cluster
## A student-built High Performance Computing (HPC) cluster using OpenHPC and Rocky Linux for scientific computing, parallel programming, and computational research.

# Team
## Samuel Durai- Project Founder/Director • Lead Systems Architect
## Shanmukha Sainath Kasireddy-	Project Director • Linux Administration • Simulations
## Eric Bipin Philip-	Software • Infrastructure
## Aayush Pal-	Simulations • Software
## Harshit Srinivasan-	Documentation • Software Implementation
## Sanskriti Srivastava- Systems Architect • Linux Administration
# About

The OpenHPC Cluster is a student-led engineering project focused on designing, building, and maintaining a functional High Performance Computing (HPC) cluster using enterprise-grade open-source technologies.

Developed as a long-term educational resource for our school, the cluster provides students with hands-on experience in Linux system administration, distributed computing, cluster networking, and scientific computing. Beyond serving as a learning platform, the cluster is intended to support computational workloads in Physics, Mathematics, Computational Fluid Dynamics (CFD), and future research-oriented projects.

By combining repurposed hardware with modern HPC software, this project demonstrates that powerful computational infrastructure can be built in an educational environment through collaboration, engineering, and open-source technologies.

# Vision

Our goal is to establish an accessible HPC environment that allows students to move beyond theoretical science and mathematics by performing real computational experiments/simulations.

The cluster is being developed to support:

-Computational Fluid Dynamics (CFD)
-Physics simulations
-Mathematical modelling
-Numerical methods
-Parallel programming
-Scientific computing
-Future machine learning workloads
-Research and educational projects

# Objectives
-Develop this HPC cluster to run complex mathematical and physics simulations.
-Support parallel programming, numerical methods, and scientific computing experiments.
-Provide a platform for benchmarking, performance analysis, and future research projects.

# Software Specifications

### Category                    Technology 
### Operating System-            Rocky Linux 9.8
### HPC Distribution-            OpenHPC
### Resource Manager-            Slurm
### MPI Implementation-          OpenMPI
### Provisioning-                Warewulf
### File Sharing-                NFS
### Remote Access-               SSH
### Networking-                  Gigabit Ethernet 

# Hardware Specifications 

The cluster is currently deployed using repurposed desktop hardware provided by the school, demonstrating how affordable systems can be transformed into a capable High Performance Computing platform.

## Current Deployment 

### Component                   Specification 
### Active Nodes-                5
### Admin Nodes-                 1
### Compute Nodes-               4
### Ethernet Switch-             TP-Link Ethernet Switch LS1016G 16-Port
### Operating System-            Rocky Linux 9.8
### HPC Distribution-            OpenHPC
### Cluster Management-          Slurm
### MPI Implementation-          OpenMPI
### Network-                     Gigabit Ethernet

# Admin Node

## Component         Specification 

### Role-              Cluster Management
### Operating system-  Rocky Linux 9.8
### Processor-         Intel i5-4460
### RAM-               16 GB DDR3
### Services-          Slurm Controller, Warewulf, NFS, SSH, OpenHPC Management Services


## Note- All softwares and Scientific Librabries will installed on the Admin Node ONLY

# Compute Nodes

## Component         Specification 

### Number of Nodes-   4
### Role-              Parallel Computing(Each Node will be assigned a particular task by the admin node depending on the simulation)
### Operating System-  No operating systemc(PXE network Boot)
### Processor-         Intel i5-4460
### RAM-               16 GB DDR3 per node
## Note- Slurm has been setup and configured on all nodes

# Aggregate Resources
## Resource                 Value
### Total Active Nodes-       5
### Total Admin Nodes-        1
### Total Compute Nodes-      4
### Total RAM-                80 GB DDR3
### Network Infrastructure-   Gigabit Ethernet
                        
# Educational Value

This project provides practical experience in:

-High Performance Computing (HPC)
-Parallel Computing
-Distributed Systems
-Linux System Administration
-Cluster Networking
-Scientific Computing
-Performance Optimization
-Research Computing

# Planned Expansion

The cluster has been designed with scalability in mind. As the project progresses, additional compute nodes and a dedicated storage server will be integrated to support larger scientific workloads, improve storage capacity, and enhance overall computational performance.

# Acknowledgements

This project would not have been possible without the approval and support of DPS-Modern Indian School, which provided all the nodes and workplace access in the school's robotics lab for the development of this cluster. We also acknowledge the OpenHPC ecosystem and the broader open-source community, whose software and documentation have made High Performance Computing accessible to students and educational institutions worldwide.

# License

This repository is intended for educational purposes and knowledge sharing.
  








