
---

```markdown
# Warewulf latest 4.6.1 Installation on AlmaLinux 9.5

## ✅ Prerequisites

1. **Install AlmaLinux 9.5** (Minimal or Server environment)
2. **Update the system packages**

```bash
dnf update -y
```

3. **Configure basic network**

Ensure your system is configured with a static IP:

```
IP Address: 10.0.0.1  
Netmask: 255.255.255.0  
Network: 10.0.0.0  
```

---

## 📦 Install Warewulf (RPM method - Recommended)

```bash
dnf install -y https://github.com/warewulf/warewulf/releases/download/v4.6.1/warewulf-4.6.1-1.el9.x86_64.rpm
```

---

## 🛠️ Install Warewulf from Source (Optional)

```bash
dnf install -y git epel-release
dnf install -y golang {libassuan,gpgme}-devel unzip tftp-server dhcp-server nfs-utils ipxe-bootimgs-{x86,aarch64}
dnf config-manager --set-enabled crb
```

```bash
git clone https://github.com/warewulf/warewulf.git
cd warewulf
PREFIX=/usr/local make defaults
make install
```

---

## 🔥 Configure firewalld

```bash
systemctl restart firewalld
firewall-cmd --permanent --add-service=warewulf
firewall-cmd --permanent --add-service=dhcp
firewall-cmd --permanent --add-service=nfs
firewall-cmd --permanent --add-service=tftp
firewall-cmd --reload
```

> Alternatively, you can disable firewalld (not recommended for production):
> ```bash
> systemctl disable --now firewalld
> ```

---

## ⚙️ Configure Warewulf

Edit the configuration file (`/etc/warewulf/warewulf.conf`) to match the following:

```yaml
ipaddr: 10.0.0.1
netmask: 255.255.252.0
network: 10.0.0.0

warewulf:
  port: 9873
  secure: false
  update interval: 60
  autobuild overlays: true
  host overlay: true
  datastore: /usr/share
  grubboot: false

dhcp:
  enabled: true
  template: default
  range start: 10.0.1.1
  range end: 10.0.1.255
  systemd name: dhcpd

tftp:
  enabled: true
  tftproot: /var/lib/tftpboot
  systemd name: tftp
  ipxe:
    "00:00": undionly.kpxe
    "00:07": ipxe-snponly-x86_64.efi
    "00:09": ipxe-snponly-x86_64.efi
    00:0B: arm64-efi/snponly.efi

nfs:
  enabled: true
  export paths:
    - path: /home
      export options: rw,sync
    - path: /opt
      export options: ro,sync,no_root_squash
  systemd name: nfs-server

image mounts:
  - source: /etc/resolv.conf
    dest: /etc/resolv.conf
    readonly: true

paths:
  bindir: /usr/bin
  sysconfdir: /etc
  localstatedir: /var/lib
  ipxesource: /usr/share/ipxe
  srvdir: /var/lib
  firewallddir: /usr/lib/firewalld/services
  systemddir: /usr/lib/systemd/system
  wwoverlaydir: /var/lib/warewulf/overlays
  wwchrootdir: /var/lib/warewulf/chroots
  wwprovisiondir: /var/lib/warewulf/provision
  wwclientdir: /warewulf
```

---

## 🚀 Start Warewulf Service

```bash
systemctl enable --now warewulfd
```

---

## 🔧 Configure Warewulf System Services

This command will automatically configure services like DHCP, TFTP, NFS, etc.

```bash
wwctl configure --all
```

---

## 📥 Import a Base Node Image

Import a prebuilt container image (Rocky Linux 9 in this example):

```bash
wwctl image import docker://ghcr.io/warewulf/warewulf-rockylinux:9 rockylinux-9 --build
```

Set it as the default profile image:

```bash
wwctl profile set default --image rockylinux-9
```

---

## 🧰 Configure Default Node Profile

```bash
wwctl profile set -y default --netmask=255.255.252.0 --gateway=10.0.0.1
```

Check the profile settings:

```bash
wwctl profile list
```

---

## ➕ Add a Node

```bash
wwctl node add n1 --ipaddr=10.0.2.1 --discoverable=true
```

View node details:

```bash
wwctl node list -a n1
```

---

## 🧱 Build Node Overlays

```bash
wwctl overlay build
```

> You can also build for a specific node:
> ```bash
> wwctl overlay build n1
> ```

---

## 🔌 Boot the Compute Node

Turn on your PXE-bootable compute node and ensure it boots using Warewulf. Monitor the output for successful provisioning.

---

## 📎 References

- [Warewulf GitHub](https://github.com/warewulf/warewulf)
- [Warewulf Documentation](https://warewulf.org/docs/)
- [Rocky Linux Container Image](https://ghcr.io/warewulf/warewulf-rockylinux)

---





