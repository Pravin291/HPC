
---

````markdown
# Intel MPI Installation Guide

This guide provides a step-by-step process to install Intel MPI using the Intel® oneAPI HPC Toolkit on RHEL-based systems (e.g., CentOS, AlmaLinux, RHEL).

## Prerequisites

- RHEL/CentOS/AlmaLinux system with `dnf` or `yum` installed.
- Sudo/root access for package installation and configuration.

---

## Step-by-Step Installation

### 1. Create the Intel oneAPI Repository File

As a **regular user**, create the YUM/DNF repo file in the `/tmp` directory:

```bash
tee > /tmp/oneAPI.repo << EOF
[oneAPI]
name=Intel® oneAPI repository
baseurl=https://yum.repos.intel.com/oneapi
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://yum.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB
EOF
````

### 2. Move the Repository File to the YUM/DNF Directory

Move the `oneAPI.repo` file to the official repository directory:

```bash
sudo mv /tmp/oneAPI.repo /etc/yum.repos.d/
```

---

### 3. Install Intel oneAPI HPC Toolkit

Install the Intel MPI (as part of the HPC Toolkit):

```bash
sudo dnf install intel-oneapi-hpc-toolkit
```

---

### 4. Set Up the Environment

Activate the Intel environment variables (Intel compilers, MPI, etc.):

```bash
source /opt/intel/oneapi/setvars.sh
```

To make this setting permanent for your user:

```bash
echo 'source /opt/intel/oneapi/setvars.sh' >> ~/.bashrc
```

Apply the change immediately:

```bash
source ~/.bashrc
```

---

### 5. Verify the MPI Installation

Check the location and version of the Intel MPI compiler wrapper:

```bash
which mpicc
mpicc --version
```

---

## Notes

* For multi-node MPI programs, ensure passwordless SSH is configured between nodes.
* Use `mpirun` or `mpiexec` to run MPI applications.

---





