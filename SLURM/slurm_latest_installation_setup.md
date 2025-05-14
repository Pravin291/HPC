

---

````markdown
# ⚙️ SLURM v24.11.5 Installation on AlmaLinux 9.5 (Master + Node Setup)

A step-by-step guide to install and configure the **latest SLURM workload manager (v24.11.5)** on **AlmaLinux 9.5**, with full support for **MUNGE**, **SLURM accounting**, and **QoS management**.

---

## 📋 Table of Contents

- [Prerequisites](#-prerequisites)
- [MUNGE Setup](#-munge-setup)
- [SLURM Installation](#-slurm-installation)
- [SLURM Configuration](#-slurm-configuration)
- [Start SLURM Services](#-start-slurm-services)
- [SLURM Accounting Setup](#-slurm-accounting-setup)
- [User and QoS Management](#-user-and-qos-management)
- [Common Issues & Fixes](#-common-issues--fixes)
- [Useful Commands](#-useful-commands)

---

## ✅ Prerequisites

Run these on **both Master and Node**:

```bash
dnf install -y epel-release
dnf config-manager --set-enabled crb
dnf groupinstall -y "Development Tools"
dnf install -y munge munge-libs munge-devel mariadb-server mariadb-devel \
               pam-devel python3 gcc make rpm-build perl-ExtUtils-MakeMaker \
               perl-DBI perl-Switch openssl-devel
````

---

## 🔐 MUNGE Setup

### On Master:

```bash
create-munge-key -f
chown -R munge:munge /etc/munge
chmod 400 /etc/munge/munge.key
systemctl enable --now munge
```

### On Node:

```bash
scp /etc/munge/munge.key root@node1:/etc/munge/
ssh node1 "chown -R munge:munge /etc/munge && chmod 400 /etc/munge/munge.key && systemctl enable --now munge"
```

---

## 🛠️ SLURM Installation

### Download and Build SLURM (on Master):

```bash
wget https://download.schedmd.com/slurm/slurm-24.11.5.tar.bz2

dnf install -y rpm-build readline-devel ncurses-devel \
    openssl-devel pam-devel libibmad-devel \
    libibumad-devel hwloc-devel numactl-devel \
    perl-ExtUtils-MakeMaker python3-devel gtk2-devel \
    libssh2-devel rrdtool-devel lua-devel libedit-devel

rpmbuild -ta slurm-24.11.5.tar.bz2
cd ~/rpmbuild/RPMS/x86_64/
dnf install -y slurm-*.rpm
```

### Install RPMs on Node:

```bash
scp ~/rpmbuild/RPMS/x86_64/slurm-*.rpm root@node1:/tmp/
ssh node1 "dnf install -y /tmp/slurm-*.rpm"
```

---

## 🧠 SLURM Configuration

### Create `slurm` User and Directories

Run on both Master and Node:

```bash
groupadd -g 1002 slurm
useradd -m -u 1002 -g slurm -c "SLURM workload manager" -s /bin/bash slurm
mkdir -p /var/spool/slurmctld /var/spool/slurmd /var/log/slurm /var/log/slurmd
chown -R slurm:slurm /var/spool /var/log/slurm /var/log/slurmd
```

### Generate `slurm.conf`

copy the slurm.conf.example file in /etc/slurm/ with slurm.conf

cp /etc/slurm/slurm.conf.example /etc/slurm/slurm.conf

Then make the some changes at the botom of the file add this lines

```conf
ControlMachine=master
NodeName=node1 CPUs=2 State=UNKNOWN
PartitionName=debug Nodes=node1 Default=YES MaxTime=INFINITE State=UP
```

Save to `/etc/slurm/slurm.conf` on Master and copy to Node:

```bash
scp /etc/slurm/slurm.conf root@node1:/etc/slurm/
```

---

## 🚀 Start SLURM Services

### On Master:

```bash
systemctl enable --now slurmctld
```

### On Node:

```bash
systemctl enable --now slurmd
```

---

## 🔎 Verify SLURM

On the **master** node:

```bash
scontrol show nodes
sinfo
srun -N1 hostname
```

---

## 📊 SLURM Accounting Setup

### Setup MariaDB:

```bash
systemctl enable --now mariadb
mysql_secure_installation
```

### Create Database and User:

```sql
CREATE DATABASE IF NOT EXISTS slurm_acct_db;
CREATE USER 'slurm'@'localhost' IDENTIFIED BY 'admin';
GRANT ALL PRIVILEGES ON slurm_acct_db.* TO 'slurm'@'localhost';
FLUSH PRIVILEGES;
```

### Configure `slurmdbd`:

```bash
cp /etc/slurm/slurmdbd.conf.example /etc/slurm/slurmdbd.conf
chown slurm: /etc/slurm/slurmdbd.conf
chmod 600 /etc/slurm/slurmdbd.conf
```

Edit `/etc/slurm/slurmdbd.conf` with correct DB credentials:

```conf
DbdHost=localhost
StorageHost=localhost
StorageUser=slurm
StoragePass=admin
StorageLoc=slurm_acct_db
```

Enable and start:

```bash
mkdir -p /var/run/slurm
chown slurm:slurm /var/run/slurm
systemctl enable --now slurmdbd
```

### Update `slurm.conf` for Accounting:

```conf
AccountingStorageType=accounting_storage/slurmdbd
AccountingStorageHost=localhost
```

Restart:

```bash
systemctl restart slurmctld
```

---

## 👤 User and QoS Management

### Add User and SLURM Account:

```bash
useradd -m alice
sacctmgr add cluster mycluster
sacctmgr add account general Description="General account" Organization="myorg"
sacctmgr add user alice Account=general Cluster=mycluster
```

### Add QoS (e.g., limit to 2 jobs):

```bash
sacctmgr add qos limit2 MaxSubmitJobs=2
sacctmgr modify user alice set qos=limit2
```

---

## 🧯 Common Issues & Fixes

### ❗ Cgroup V2 Error:

```bash
mkdir -p /sys/fs/cgroup/freezer
mount -t cgroup -o freezer cgroup /sys/fs/cgroup/freezer
```

---

## 🔧 Useful Commands

| Command                      | Description                  |
| ---------------------------- | ---------------------------- |
| `scontrol show nodes`        | Show node info               |
| `sinfo`                      | Show partition/node status   |
| `squeue`                     | Show job queue               |
| `sbatch job.sh`              | Submit batch job             |
| `srun hostname`              | Run interactive job          |
| `sacctmgr list user`         | Show SLURM users             |
| `systemctl status slurmctld` | Check controller status      |
| `sacctmgr show qos`          | View configured QoS policies |

---




