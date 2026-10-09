# OpenHPC Cluster Deployment Guide

## Rocky Linux 9.8 "Blue Onyx" + Warewulf 4 + Slurm

---

## 1. Introduction

This guide explains how to build a small High Performance Computing (HPC) cluster using completely open-source software.

The cluster uses:

* **Rocky Linux 9.8 "Blue Onyx"** as the operating system
* **OpenHPC 3.5** as the HPC software stack
* **Warewulf 4.7** for compute-node provisioning
* **Slurm** for workload and resource management
* **Open MPI / MPICH** for parallel computing
* **Prometheus + Grafana** for monitoring
* **NFS** for shared storage
* **DHCP + TFTP/iPXE** for network booting

The basic architecture is:

```text
                         INTERNET / LAN
                              |
                              |
                       +------+------+
                       |   ADMIN     |
                       |    NODE     |
                       | Rocky 9.8   |
                       |             |
                       | Warewulf    |
                       | Slurmctld   |
                       | DHCP        |
                       | NFS         |
                       | Monitoring  |
                       +------+------+
                              |
                    Cluster / PXE Network
                              |
                    +---------+---------+
                    |   Cisco 3560-CX   |
                    |      Switch       |
                    +--+------+------+--+
                       |      |      |
                       |      |      |
                    +--+--+ +--+--+ +--+--+
                    |Node1| |Node2| |Node3| ...
                    +-----+ +-----+ +-----+
```

The administration node is responsible for managing the entire cluster. Compute nodes do not need to have a complete operating-system installation on their local drives. Warewulf can provision their operating environment over the network.

---

# 2. Hardware

## 2.1 Minimum requirements

You need:

* At least 2 computers

  * 1 administration/head node
  * 1 or more compute nodes
* Gigabit Ethernet switch
* Ethernet cables
* Ethernet NICs
* Monitor
* Keyboard
* Mouse
* USB drive for Rocky Linux installation
* A will to live

## 2.2 Hardware used in this project

The example system uses:

| Component           | Hardware               |
| ------------------- | ---------------------- |
| Administration node | ThinkCentre E73        |
| Compute nodes       | 4× ThinkCentre E73     |
| Switch              | Cisco Catalyst 3560-CX |
| Ethernet            | CAT 5e / CAT 6         |
| Additional NIC      | TP-Link TG-3468        |

The same procedure can be adapted for larger clusters.

---

# 3. Network Design

Before installing anything, design the network.

For this guide, use a dedicated cluster network:

```text
Network:       10.10.10.0/24
Gateway:       10.10.10.1
Admin node:    10.10.10.1
Compute node 1:10.10.10.101
Compute node 2:10.10.10.102
Compute node 3:10.10.10.103
Compute node 4:10.10.10.104
```

The administration node should ideally have **two network interfaces**:

```text
             Normal LAN
                |
                |
          +-----+-----+
          |  Admin    |
          |   Node    |
          +-----+-----+
                |
          Cluster LAN
                |
             Switch
           /  / \  \
        CN1 CN2 CN3 CN4
```

One interface connects the administration node to the normal network/internet.

The second interface connects it to the isolated HPC network.

For example:

```text
eno1 = Internet / normal LAN
eno2 = HPC cluster network
```

Check your interface names with:

```bash
ip link
```

or:

```bash
nmcli device status
```

Do **not** blindly assume your interfaces are named `eno1` and `eno2`.

---

# 4. Install Rocky Linux 9.8

Install Rocky Linux 9.8 on the administration node.

A minimal installation is sufficient.

During installation:

* Set hostname to `admin`
* Configure the cluster NIC
* Give the cluster interface a static IP
* Install SSH server
* Create an administrator account

After installation:

```bash
sudo dnf update -y
```

Verify the operating system:

```bash
cat /etc/os-release
```

You should see Rocky Linux 9.x information.

Check the kernel:

```bash
uname -r
```

Check the hostname:

```bash
hostnamectl
```

Set the hostname if required:

```bash
sudo hostnamectl set-hostname admin
```

---

# 5. Configure the Cluster Network

Identify the cluster interface:

```bash
nmcli device status
```

For this example, assume the cluster interface is `eno2`.

Create a connection:

```bash
sudo nmcli connection add \
  type ethernet \
  ifname eno2 \
  con-name cluster \
  ipv4.method manual \
  ipv4.addresses 10.10.10.1/24
```

Bring it online:

```bash
sudo nmcli connection up cluster
```

Verify:

```bash
ip addr show eno2
```

You should see:

```text
inet 10.10.10.1/24
```

The cluster interface should normally **not** have an internet gateway configured.

---

# 6. Configure Hostname Resolution

Create a hosts file containing the cluster nodes:

```bash
sudo nano /etc/hosts
```

Add:

```text
10.10.10.1    admin
10.10.10.101  node01
10.10.10.102  node02
10.10.10.103  node03
10.10.10.104  node04
```

Test:

```bash
ping -c 3 admin
```

The compute nodes will receive their network configuration through DHCP later.

---

# 7. Configure the Cisco Switch

Connect the administration node and compute nodes to the Cisco switch.

Enter configuration mode:

```text
enable
configure terminal
```

Configure the cluster ports as access ports.

For example, if ports Gi1/0/1 through Gi1/0/5 are used:

```text
interface range GigabitEthernet1/0/1 - 5
 switchport mode access
 spanning-tree portfast
 no shutdown
exit
```

Save the configuration:

```text
end
write memory
```

Check:

```text
show interfaces status
```

And:

```text
show spanning-tree interface GigabitEthernet1/0/1
```

### Important

`spanning-tree portfast` should be used on ports connected to **end devices**, such as your administration node and compute nodes.

Do not blindly enable PortFast on switch-to-switch links.

---

# 8. Install the OpenHPC Repository

OpenHPC provides RPM repositories for supported enterprise Linux distributions. OpenHPC 3.x targets RHEL 9-compatible systems, including Rocky Linux.

Install the OpenHPC release package:

```bash
sudo dnf install -y https://github.com/openhpc/ohpc/releases/download/v3.5.GA/ohpc-release-3.5-1.el9.x86_64.rpm
```

Then refresh the repositories:

```bash
sudo dnf clean all
sudo dnf makecache
```

Check that OpenHPC repositories are visible:

```bash
dnf repolist
```

You should see an OpenHPC repository.

---

# 9. Install Basic OpenHPC Components

Install the OpenHPC filesystem package:

```bash
sudo dnf install -y ohpc-filesystem
```

Install development tools:

```bash
sudo dnf groupinstall -y "Development Tools"
```

Install commonly required utilities:

```bash
sudo dnf install -y \
  wget \
  curl \
  git \
  vim \
  nano \
  rsync \
  chrony \
  tar \
  bzip2 \
  unzip \
  pciutils \
  lshw
```

Enable time synchronization:

```bash
sudo systemctl enable --now chronyd
```

Check:

```bash
timedatectl
```

---

# 10. Install Warewulf

OpenHPC 3.5 includes Warewulf 4.7.0.

Install Warewulf:

```bash
sudo dnf install -y warewulf-ohpc
```

Check the version:

```bash
wwctl version
```

You should see a Warewulf 4.x version.

---

# 11. Configure Warewulf

Open the configuration file:

```bash
sudo nano /etc/warewulf/warewulf.conf
```

A basic configuration should contain the cluster network information.

For example:

```yaml
ipaddr: 10.10.10.1
netmask: 255.255.255.0

network:
  device: eno2
  network: 10.10.10.0
  netmask: 255.255.255.0
  gateway: 0.0.0.0
```

The exact configuration generated by the OpenHPC/Warewulf package may differ between releases, so preserve existing sections and modify the network values rather than replacing the entire file.

Validate:

```bash
wwctl container list
```

---

# 12. Configure DHCP

Warewulf can manage the DHCP/PXE configuration used to provision compute nodes.

First install the required DHCP components:

```bash
sudo dnf install -y dhcp-server
```

The DHCP service must operate on the **cluster interface**, not your normal LAN interface.

Configure firewalld appropriately.

For a dedicated isolated cluster network, first determine your active zone:

```bash
sudo firewall-cmd --get-active-zones
```

Add the cluster interface to an appropriate zone:

```bash
sudo firewall-cmd --permanent \
  --zone=trusted \
  --add-interface=eno2

sudo firewall-cmd --reload
```

Verify:

```bash
sudo firewall-cmd --zone=trusted --list-interfaces
```

---

# 13. Create the Compute Node Image

Warewulf provisions compute nodes using a node image/container.

Create a Rocky Linux container:

```bash
sudo wwctl container import \
  docker://rockylinux:9 \
  rocky-9
```

Check:

```bash
sudo wwctl container list
```

You should see:

```text
rocky-9
```

Enter the container:

```bash
sudo wwctl container exec rocky-9 -- /bin/bash
```

Inside the image, update the packages:

```bash
dnf update -y
```

Install basic packages:

```bash
dnf install -y \
  bash \
  coreutils \
  iproute \
  iputils \
  passwd \
  procps-ng \
  hostname \
  openssh-clients \
  openssh-server \
  chrony \
  sudo
```

Exit:

```bash
exit
```

---

# 14. Install OpenHPC Software in the Compute Image

The compute nodes need the OpenHPC repository as well.

Add the repository inside the image:

```bash
sudo wwctl container exec rocky-9 -- \
  dnf install -y \
  https://github.com/openhpc/ohpc/releases/download/v3.5.GA/ohpc-release-3.5-1.el9.x86_64.rpm
```

Update:

```bash
sudo wwctl container exec rocky-9 -- dnf update -y
```

Install the OpenHPC filesystem:

```bash
sudo wwctl container exec rocky-9 -- \
  dnf install -y ohpc-filesystem
```

Install the Slurm compute-side packages:

```bash
sudo wwctl container exec rocky-9 -- \
  dnf install -y slurm-ohpc
```

Install MPI software.

For example:

```bash
sudo wwctl container exec rocky-9 -- \
  dnf install -y openmpi5-gnu15-ohpc
```

The exact package names can be checked with:

```bash
dnf search openmpi
```

or:

```bash
dnf search mpich
```

---

# 15. Configure the Compute Image

Configure hostname handling:

```bash
sudo wwctl container exec rocky-9 -- \
  systemctl enable sshd
```

Enable chrony:

```bash
sudo wwctl container exec rocky-9 -- \
  systemctl enable chronyd
```

Create the Slurm directories:

```bash
sudo wwctl container exec rocky-9 -- \
  mkdir -p /var/spool/slurmd
```

Set permissions:

```bash
sudo wwctl container exec rocky-9 -- \
  chmod 755 /var/spool/slurmd
```

---

# 16. Build the Compute Image

Build the Warewulf image:

```bash
sudo wwctl container build rocky-9
```

Check:

```bash
sudo wwctl container list
```

You should now have a usable Rocky Linux compute-node image.

---

# 17. Create Compute Nodes

Add the first node:

```bash
sudo wwctl node add node01 \
  --ipaddr 10.10.10.101 \
  --hwaddr XX:XX:XX:XX:XX:XX
```

Replace:

```text
XX:XX:XX:XX:XX:XX
```

with the actual MAC address of node01.

Find the MAC address on the compute machine:

```bash
ip link
```

or:

```bash
nmcli device show
```

Repeat for the other nodes:

```bash
sudo wwctl node add node02 \
  --ipaddr 10.10.10.102 \
  --hwaddr XX:XX:XX:XX:XX:XX

sudo wwctl node add node03 \
  --ipaddr 10.10.10.103 \
  --hwaddr XX:XX:XX:XX:XX:XX

sudo wwctl node add node04 \
  --ipaddr 10.10.10.104 \
  --hwaddr XX:XX:XX:XX:XX:XX
```

Check:

```bash
sudo wwctl node list
```

---

# 18. Assign the Rocky Image

Assign the image to each node:

```bash
sudo wwctl node set node01 --image rocky-9
sudo wwctl node set node02 --image rocky-9
sudo wwctl node set node03 --image rocky-9
sudo wwctl node set node04 --image rocky-9
```

Check:

```bash
sudo wwctl node list
```

---

# 19. Configure Warewulf

Generate the required configuration:

```bash
sudo wwctl configure --all
```

Build overlays:

```bash
sudo wwctl overlay build
```

Restart Warewulf:

```bash
sudo systemctl enable --now warewulfd
```

Check:

```bash
sudo systemctl status warewulfd
```

---

# 20. PXE Boot the Compute Nodes

Enter the BIOS/UEFI settings on each ThinkCentre.

Enable:

```text
PXE Boot
Network Boot
UEFI Network Stack
```

The exact names depend on the firmware.

Set network boot before the local disk if necessary.

Connect the node to the cluster switch.

Power it on.

The expected sequence is:

```text
BIOS/UEFI
   ↓
Network initialization
   ↓
DHCP request
   ↓
PXE/iPXE
   ↓
Warewulf
   ↓
Rocky Linux image
   ↓
Compute node
```

Watch the administration node:

```bash
sudo journalctl -fu warewulfd
```

And:

```bash
sudo journalctl -f
```

If DHCP is running independently:

```bash
sudo journalctl -fu dhcpd
```

---

# 21. Verify the Nodes

After the node boots, check:

```bash
ping -c 3 10.10.10.101
```

Then:

```bash
ssh root@10.10.10.101
```

Verify the OS:

```bash
cat /etc/os-release
```

Verify networking:

```bash
ip addr
```

Verify the CPU:

```bash
lscpu
```

Verify memory:

```bash
free -h
```

Verify Warewulf:

```bash
hostname
```

The node should identify itself as:

```text
node01
```

---

# 22. Install and Configure Slurm

Slurm is responsible for scheduling jobs across the compute nodes.

OpenHPC 3.5 currently ships with the Slurm 25.11 series.

On the administration node:

```bash
sudo dnf install -y \
  slurm-ohpc \
  slurm-slurmctld-ohpc
```

On the compute image:

```bash
sudo wwctl container exec rocky-9 -- \
  dnf install -y slurm-ohpc slurm-slurmd-ohpc
```

Create the spool directory:

```bash
sudo wwctl container exec rocky-9 -- \
  mkdir -p /var/spool/slurmd
```

---

# 23. Configure Munge

Munge provides authentication for Slurm.

Install it on the administration node:

```bash
sudo dnf install -y munge munge-libs
```

Enable it:

```bash
sudo systemctl enable --now munge
```

Check:

```bash
sudo systemctl status munge
```

Test:

```bash
munge -n | unmunge
```

The result should indicate successful decoding.

The **same Munge key** must be present on every node.

Copy the key into the Warewulf image:

```bash
sudo mkdir -p /etc/munge
sudo cp /etc/munge/munge.key /etc/warewulf/containers/rocky-9/etc/munge/
```

Depending on the Warewulf configuration and image storage path, use the appropriate image-overlay/resource mechanism instead of directly copying into the container filesystem if your installation stores images elsewhere.

Ensure correct permissions:

```bash
sudo chmod 400 /etc/munge/munge.key
sudo chown munge:munge /etc/munge/munge.key
```

---

# 24. Create slurm.conf

Slurm's main configuration file is:

```text
/etc/slurm/slurm.conf
```

Create it:

```bash
sudo nano /etc/slurm/slurm.conf
```

A simple four-node configuration can look like:

```text
ClusterName=openhpc
SlurmctldHost=admin

SlurmUser=slurm

AuthType=auth/munge
ProctrackType=proctrack/cgroup
ReturnToService=2

SlurmctldPidFile=/var/run/slurmctld.pid
SlurmdPidFile=/var/run/slurmd.pid

SlurmdSpoolDir=/var/spool/slurmd
StateSaveLocation=/var/spool/slurmctld

SlurmctldPort=6817
SlurmdPort=6818

SelectType=select/cons_tres
SelectTypeParameters=CR_Core

SchedulerType=sched/backfill

NodeName=node[01-04] \
  CPUs=4 \
  RealMemory=7000 \
  State=UNKNOWN

PartitionName=compute \
  Nodes=node[01-04] \
  Default=YES \
  MaxTime=INFINITE \
  State=UP
```

**Important:** change `CPUs` and `RealMemory` to match your actual ThinkCentre hardware.

Check:

```bash
lscpu
free -m
```

---

# 25. Create Slurm Directories

On the administration node:

```bash
sudo mkdir -p /var/spool/slurmctld
sudo mkdir -p /var/log/slurm
```

Set ownership:

```bash
sudo chown slurm:slurm /var/spool/slurmctld
sudo chown slurm:slurm /var/log/slurm
```

---

# 26. Distribute Slurm Configuration

Place the Slurm configuration into the compute image.

For example:

```bash
sudo mkdir -p /etc/warewulf/overlays/slurm
```

Copy:

```bash
sudo cp /etc/slurm/slurm.conf \
  /etc/warewulf/overlays/slurm/
```

The exact overlay implementation can vary with the Warewulf release. The important principle is that every compute node must receive the same:

```text
/etc/slurm/slurm.conf
```

and the same:

```text
/etc/munge/munge.key
```

Rebuild the image/overlay after making changes.

---

# 27. Configure Slurm Services

Administration node:

```bash
sudo systemctl enable --now slurmctld
```

Check:

```bash
sudo systemctl status slurmctld
```

Compute nodes:

```bash
sudo systemctl enable slurmd
```

Because the nodes are provisioned by Warewulf, this should be incorporated into the compute image.

---

# 28. Check the Slurm Cluster

On the administration node:

```bash
sinfo
```

Expected output will resemble:

```text
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
compute*     up   infinite      4   idle node[01-04]
```

Check detailed node information:

```bash
scontrol show nodes
```

Check partitions:

```bash
scontrol show partition
```

If nodes appear as:

```text
DOWN
```

or:

```text
INVAL
```

check:

```bash
journalctl -u slurmctld
```

and on the compute node:

```bash
journalctl -u slurmd
```

---

# 29. Run Your First HPC Job

Create a test job:

```bash
nano test.sh
```

Use:

```bash
#!/bin/bash

echo "Running on:"
hostname

echo "CPU information:"
lscpu | grep "Model name"

echo "Date:"
date
```

Make it executable:

```bash
chmod +x test.sh
```

Submit it:

```bash
sbatch test.sh
```

Check the queue:

```bash
squeue
```

After completion:

```bash
ls
```

You should see a file such as:

```text
slurm-1.out
```

Read it:

```bash
cat slurm-1.out
```

---

# 30. Test Multiple Nodes

Create:

```bash
nano multi.sh
```

Use:

```bash
#!/bin/bash

echo "Running on $(hostname)"
sleep 10
```

Request four nodes:

```bash
sbatch --nodes=4 --ntasks=4 multi.sh
```

Check:

```bash
squeue
```

---

# 31. Install MPI

HPC applications generally require a message-passing implementation.

OpenHPC provides MPI implementations such as Open MPI and MPICH.

For Open MPI:

```bash
sudo dnf search openmpi
```

Install the appropriate OpenHPC Open MPI package:

```bash
sudo dnf install -y openmpi5-gnu15-ohpc
```

The package name can be checked first because compiler/MPI combinations can change between OpenHPC releases.

Verify:

```bash
module avail
```

Load the MPI environment:

```bash
module load gnu15
module load openmpi5
```

Check:

```bash
which mpicc
```

and:

```bash
mpirun --version
```

---

# 32. Compile an MPI Program

Create:

```bash
nano hello.c
```

Enter:

```c
#include <mpi.h>
#include <stdio.h>

int main(int argc, char **argv)
{
    int rank, size;

    MPI_Init(&argc, &argv);

    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);

    printf("Hello from rank %d of %d on %s\n",
           rank, size, getenv("HOSTNAME"));

    MPI_Finalize();

    return 0;
}
```

Compile:

```bash
mpicc hello.c -o hello
```

Run:

```bash
mpirun -np 4 ./hello
```

For a production Slurm environment, launch MPI applications through Slurm:

```bash
srun -N 4 -n 4 ./hello
```

---

# 33. Shared Storage with NFS

A cluster is much easier to use when compute nodes share a common filesystem.

Install NFS:

```bash
sudo dnf install -y nfs-utils
```

Create a shared directory:

```bash
sudo mkdir -p /export/home
```

Set permissions:

```bash
sudo chmod 755 /export/home
```

Configure:

```bash
sudo nano /etc/exports
```

Add:

```text
/export/home 10.10.10.0/24(rw,sync,no_subtree_check)
```

Export it:

```bash
sudo exportfs -rav
```

Enable NFS:

```bash
sudo systemctl enable --now nfs-server
```

Check:

```bash
sudo exportfs -v
```

---

# 34. Mount the Shared Directory

On a compute node:

```bash
sudo mkdir -p /home
```

Test the mount:

```bash
sudo mount 10.10.10.1:/export/home /home
```

Check:

```bash
df -h
```

You should see:

```text
10.10.10.1:/export/home
```

For a real cluster deployment, configure this through the Warewulf image/overlay so that the mount is automatically available when nodes boot.

---

# 35. Monitoring

A functional cluster is not enough; you should also be able to monitor it.

A completely open-source monitoring stack can use:

```text
Node Exporter
      ↓
Prometheus
      ↓
Grafana
```

Architecture:

```text
+----------+      +------------+
| Compute  | ---> |            |
| Node 01  |      |            |
+----------+      | Prometheus |
                  |            |
+----------+      |            |
| Compute  | ---> |            |
| Node 02  |      +------+-----+
+----------+             |
                         |
                    +----+----+
                    | Grafana |
                    +---------+
```

---

# 36. Install Prometheus

Install Prometheus on the administration node.

If the Rocky repositories provide the required package:

```bash
sudo dnf install -y prometheus
```

Check:

```bash
prometheus --version
```

Enable:

```bash
sudo systemctl enable --now prometheus
```

Check:

```bash
sudo systemctl status prometheus
```

---

# 37. Install Node Exporter

Node Exporter collects CPU, memory, filesystem and other system metrics.

Install:

```bash
sudo dnf install -y node_exporter
```

Enable:

```bash
sudo systemctl enable --now node_exporter
```

Check:

```bash
curl http://localhost:9100/metrics
```

You should receive Prometheus-formatted metrics.

Install Node Exporter into the compute image as well:

```bash
sudo wwctl container exec rocky-9 -- \
  dnf install -y node_exporter
```

Enable it:

```bash
sudo wwctl container exec rocky-9 -- \
  systemctl enable node_exporter
```

---

# 38. Configure Prometheus

Edit:

```bash
sudo nano /etc/prometheus/prometheus.yml
```

Add your cluster nodes:

```yaml
scrape_configs:

  - job_name: "cluster-nodes"

    static_configs:
      - targets:
          - "10.10.10.101:9100"
          - "10.10.10.102:9100"
          - "10.10.10.103:9100"
          - "10.10.10.104:9100"
```

Restart:

```bash
sudo systemctl restart prometheus
```

Check:

```bash
sudo systemctl status prometheus
```

---

# 39. Install Grafana

Install Grafana using the official open-source Grafana repository.

First create the repository:

```bash
sudo nano /etc/yum.repos.d/grafana.repo
```

Add:

```ini
[grafana]
name=grafana
baseurl=https://rpm.grafana.com
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://rpm.grafana.com/gpg.key
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
```

Install:

```bash
sudo dnf install -y grafana
```

Enable:

```bash
sudo systemctl enable --now grafana-server
```

Check:

```bash
sudo systemctl status grafana-server
```

---

# 40. Access Grafana

Grafana normally listens on:

```text
http://ADMIN-IP:3000
```

For example:

```text
http://10.10.10.1:3000
```

The first login will prompt you to create/change the administrator credentials.

---

# 41. Add Prometheus to Grafana

In Grafana:

```text
Connections
    ↓
Data sources
    ↓
Add data source
    ↓
Prometheus
```

Set the URL to:

```text
http://localhost:9090
```

Save and test.

Grafana should report that the data source is working.

---

# 42. Create a Cluster Dashboard

Create a new dashboard.

Useful metrics include:

### CPU

```promql
100 - (avg by(instance) (
  rate(node_cpu_seconds_total{mode="idle"}[5m])
) * 100)
```

### Memory

```promql
100 * (
  1 -
  node_memory_MemAvailable_bytes /
  node_memory_MemTotal_bytes
)
```

### Disk space

```promql
100 * (
  1 -
  node_filesystem_avail_bytes /
  node_filesystem_size_bytes
)
```

### Network traffic

```promql
rate(node_network_receive_bytes_total[5m])
```

and:

```promql
rate(node_network_transmit_bytes_total[5m])
```

You can then create a dashboard containing:

```text
+-----------------------------------------------+
|             OPENHPC CLUSTER                   |
+-----------------------------------------------+
| CPU Usage              | Memory Usage         |
| node01 ████████        | node01 █████         |
| node02 █████           | node02 ███████       |
| node03 █████████       | node03 ████          |
+------------------------+----------------------+
| Network Traffic                               |
|                                               |
|       Graph                                   |
|                                               |
+-----------------------------------------------+
| Filesystem Usage       | Node Status          |
| node01 42%             | node01 UP            |
| node02 31%             | node02 UP            |
+-----------------------------------------------+
```

---

# 43. Test the Cluster

At this point test every major component.

## Network

```bash
ping node01
ping node02
ping node03
ping node04
```

## SSH

```bash
ssh node01
ssh node02
```

## Warewulf

```bash
wwctl node list
```

## Slurm

```bash
sinfo
```

## Node status

```bash
scontrol show nodes
```

## Job submission

```bash
sbatch test.sh
```

## MPI

```bash
srun -N 4 -n 4 ./hello
```

## Monitoring

Open:

```text
http://ADMIN-IP:3000
```

---

# 44. Troubleshooting

## Compute node does not PXE boot

Check the physical connection first:

```bash
ip link
```

Check the switch:

```text
show interfaces status
```

Check PortFast:

```text
show spanning-tree interface GigabitEthernet1/0/1
```

Check DHCP:

```bash
sudo journalctl -fu dhcpd
```

Check Warewulf:

```bash
sudo journalctl -fu warewulfd
```

---

## Node receives an IP but does not boot

Check its MAC address:

```bash
ip link
```

Compare it with:

```bash
wwctl node list
```

If the MAC address is wrong, correct the node definition.

---

## Node boots but Slurm says DOWN

Check:

```bash
sinfo
```

Then:

```bash
scontrol show node node01
```

On the compute node:

```bash
systemctl status slurmd
```

Check logs:

```bash
journalctl -u slurmd
```

On the administration node:

```bash
journalctl -u slurmctld
```

---

## Munge authentication failure

Test:

```bash
munge -n | unmunge
```

Make sure every machine uses the same:

```text
/etc/munge/munge.key
```

Check permissions:

```bash
ls -l /etc/munge/munge.key
```

The key should not be world-readable.

---

## MPI does not work

Check:

```bash
which mpicc
```

and:

```bash
which mpirun
```

Check loaded modules:

```bash
module list
```

Load the required environment:

```bash
module load gnu15
module load openmpi5
```

---

# 45. Final Cluster Architecture

The completed system should resemble:

```text
                         INTERNET
                            |
                            |
                     +------+------+
                     |   ADMIN     |
                     |    NODE     |
                     | Rocky 9.8   |
                     +------+------+
                            |
                     10.10.10.0/24
                            |
                 +----------+----------+
                 | Cisco 3560-CX      |
                 +----------+----------+
                    |    |    |    |
                    |    |    |    |
                 +--+ +--+ +--+ +--+
                 |01| |02| |03| |04|
                 +--+ +--+ +--+ +--+
                    COMPUTE NODES
```

The administration node provides:

```text
Rocky Linux
     |
     +-- OpenHPC
     |
     +-- Warewulf
     |      |
     |      +-- PXE
     |      +-- Compute images
     |      +-- Node management
     |
     +-- Slurm
     |      |
     |      +-- Job scheduling
     |      +-- Resource allocation
     |
     +-- NFS
     |      |
     |      +-- Shared storage
     |
     +-- Prometheus
     |      |
     |      +-- Metrics collection
     |
     +-- Grafana
            |
            +-- Cluster visualization
```

---

# 46. Basic Commands Reference

### Warewulf

```bash
wwctl node list
wwctl node status
wwctl container list
wwctl overlay list
wwctl overlay build
wwctl configure --all
```

### Slurm

```bash
sinfo
squeue
scontrol show nodes
scontrol show partition
sbatch job.sh
srun command
scancel JOBID
```

### MPI

```bash
mpicc program.c -o program
mpirun -np 4 ./program
srun -N 4 -n 4 ./program
```

### System monitoring

```bash
top
htop
free -h
lscpu
lsblk
df -h
ip addr
ip route
```

### Services

```bash
systemctl status warewulfd
systemctl status slurmctld
systemctl status munge
systemctl status prometheus
systemctl status grafana-server
```

---

# 47. Conclusion

You have now built a complete small-scale HPC cluster using open-source software.

The final software stack is:

| Function           | Software              |
| ------------------ | --------------------- |
| Operating System   | Rocky Linux 9.8       |
| HPC Stack          | OpenHPC 3.5           |
| Provisioning       | Warewulf 4.7          |
| Scheduler          | Slurm                 |
| Authentication     | Munge                 |
| Parallel Computing | Open MPI / MPICH      |
| Shared Storage     | NFS                   |
| Metrics            | Prometheus            |
| Visualization      | Grafana               |
| Network Boot       | PXE/iPXE              |
| Network            | Cisco Ethernet switch |

The most important concept is that these components perform different jobs:

```text
WAREWULF
"What operating system should this node boot?"

SLURM
"Which node should run this job?"

MPI
"How should processes communicate?"

NFS
"Where can the nodes access shared files?"

PROMETHEUS
"What is happening inside the cluster?"

GRAFANA
"How can I visualize what is happening?"
```

Together, these components form a functional HPC environment rather than simply a collection of computers.

---

## Important Version Note

This guide deliberately targets **OpenHPC 3.5 on Rocky Linux 9.8**.

OpenHPC 3.5 was released in June 2026 and is built against RHEL 9.8. It upgraded Warewulf to 4.7.0 and Slurm to the 25.11 series. Therefore, older tutorials that use Warewulf 3, older Slurm versions, or OpenHPC 2.x should not be copied blindly into this installation.

OpenHPC 4.x is a different branch targeting newer operating-system releases, so this guide intentionally stays on the 3.x/Rocky 9.8 stack.



 
