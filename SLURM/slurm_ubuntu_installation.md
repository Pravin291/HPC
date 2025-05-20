
---

````markdown
# SLURM 23.11.11 Cluster Setup (Manager + Compute Nodes) on Ubuntu

This guide describes how to build and configure a SLURM 23.11.11-based HPC cluster using one Manager node and multiple Compute nodes on Ubuntu. It includes fixes for `cgroup/freezer` errors and basic accounting via MariaDB.

---

## 📦 Prerequisites

### On Manager Node

```bash
sudo apt update
sudo apt install -y build-essential munge libmunge-dev libmunge2 \
libmysqlclient-dev libssl-dev libpam0g-dev libnuma-dev perl \
mailutils mariadb-server
````

### On Compute Nodes

```bash
sudo apt update
sudo apt install -y build-essential munge libmunge-dev libmunge2
```

---

## 📁 Install SLURM 23.11.11 on All Nodes

```bash
tar -xvjf slurm-23.11.11.tar.bz2
cd slurm-23.11.11
./configure --prefix=/opt/slurm-23.11.11
make -j$(nproc)
sudo make install
```

---

## 🔐 Configure Munge (On All Nodes)

### On Manager:

```bash
chown munge: /etc/munge/munge.key
chmod 400 /etc/munge/munge.key
scp /etc/munge/munge.key slurm@<compute-node-ip>:/tmp
```

### On Compute Nodes:

```bash
sudo cp /tmp/munge.key /etc/munge/
```

### On All Nodes:

```bash
sudo chown -R munge: /etc/munge/ /var/log/munge/
sudo chmod 0700 /etc/munge/ /var/log/munge/
sudo systemctl enable munge
sudo systemctl start munge
```

---

## ⚙️ Configure SLURM

### On Manager:

```bash
cd /opt/slurm-23.11.11/etc
cp slurm.conf.example slurm.conf
vi slurm.conf
```

**Sample `slurm.conf`:**

```ini
ClusterName=cluster
SlurmctldHost=slurm-manager
MailProg=/bin/mail
MpiDefault=none
ProctrackType=proctrack/cgroup
ReturnToService=1
SlurmctldPidFile=/var/run/slurmctld.pid
SlurmctldPort=6817
SlurmdPidFile=/var/run/slurmd.pid
SlurmdPort=6818
SlurmdSpoolDir=/var/spool/slurmd
SlurmUser=root
StateSaveLocation=/var/spool/slurmctld
SwitchType=switch/none
TaskPlugin=task/affinity
SchedulerType=sched/backfill
SelectType=select/cons_tres
SelectTypeParameters=CR_Core
AccountingStorageType=accounting_storage/slurmdbd
JobAcctGatherFrequency=30
JobAcctGatherType=jobacct_gather/none
SlurmctldLogFile=/var/log/slurmctld.log
SlurmdLogFile=/var/log/slurmd.log
NodeName=slurm-manager,slurm-compute1 CPUs=1 State=UNKNOWN
PartitionName=debug Nodes=ALL Default=YES MaxTime=INFINITE State=UP
```

Copy config to compute nodes:

```bash
scp /opt/slurm-23.11.11/etc/slurm.conf slurm@<compute-node-ip>:/tmp
```

### On Compute Nodes:

```bash
sudo cp /tmp/slurm.conf /opt/slurm-23.11.11/etc/
```

---

## 🔧 Systemd Service Links

### On All Nodes:

```bash
sudo ln -s /opt/slurm-23.11.11/etc/slurmd.service /usr/lib/systemd/system/slurmd.service
```

### On Manager:

```bash
sudo ln -s /opt/slurm-23.11.11/etc/slurmctld.service /usr/lib/systemd/system/slurmctld.service
```

---

## 🧊 Fix Freezer CGroup Error

### Edit `/opt/slurm-23.11.11/etc/cgroup.conf`

```ini
CgroupPlugin=cgroup/v1
CgroupAutomount=yes
ConstrainCores=no
ConstrainDevices=yes
ConstrainRAMSpace=no
ConstrainSwapSpace=no
CgroupMountpoint=/sys/fs/cgroup
```

### Manually Mount Freezer Subsystem

```bash
sudo mkdir -p /sys/fs/cgroup/freezer
sudo mount -t cgroup -o freezer cgroup /sys/fs/cgroup/freezer
```

To make it permanent, add to `/etc/fstab`:

```fstab
cgroup  /sys/fs/cgroup/freezer  cgroup  defaults,freezer  0  0
```

---

## 🧮 Enable SLURM Accounting

### Configure MariaDB on Manager

```bash
sudo mysql
```

```sql
CREATE DATABASE slurm_acct_db;
CREATE USER 'slurm'@'localhost' IDENTIFIED BY 'slurm123';
GRANT ALL PRIVILEGES ON slurm_acct_db.* TO 'slurm'@'localhost';
FLUSH PRIVILEGES;
```

### Configure `slurmdbd.conf`

```bash
cd /opt/slurm-23.11.11/etc
cp slurmdbd.conf.example slurmdbd.conf
vi slurmdbd.conf
```

**Sample `slurmdbd.conf`:**

```ini
AuthType=auth/munge
DbdHost=localhost
SlurmUser=root
DebugLevel=verbose
LogFile=/var/log/slurm/slurmdbd.log
PidFile=/var/run/slurmdbd.pid
StorageType=accounting_storage/mysql
StorageUser=slurm
StoragePass=slurm123
StorageLoc=slurm_acct_db
```

### Enable slurmdbd Service

```bash
sudo ln -s /opt/slurm-23.11.11/etc/slurmdbd.service /usr/lib/systemd/system/slurmdbd.service
sudo systemctl daemon-reexec
sudo systemctl enable slurmdbd
sudo systemctl start slurmdbd
```

---

## 🟢 Start SLURM Services

### On Manager:

```bash
sudo systemctl start slurmctld
sudo systemctl start slurmd
```

### On Compute Nodes:

```bash
sudo systemctl start slurmd
```

---

## 🔁 Resume Nodes from DOWN State

```bash
scontrol update NodeName=<node-name> State=RESUME
```

---

## 🧪 Debugging Tips

To run `slurmctld` in foreground (debug):

```bash
/opt/slurm-23.11.11/sbin/slurmctld -D
```

---

## 📤 Environment Variables

Add these to `.bashrc` or run manually:

```bash
export PATH="/opt/slurm-23.11.11/bin:/opt/slurm-23.11.11/sbin:$PATH"
export LD_LIBRARY_PATH="/opt/slurm-23.11.11/lib:$LD_LIBRARY_PATH"
```

---

## ✅ Verification

Test with:

```bash
sinfo
scontrol show nodes
```

---

