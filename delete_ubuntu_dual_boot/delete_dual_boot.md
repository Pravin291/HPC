

````markdown
# Remove Ubuntu from Dual Boot (Windows + Ubuntu)

This guide explains how to safely remove Ubuntu from a dual boot system with Windows. It walks through deleting Ubuntu partitions, cleaning the bootloader, and extending your Windows partition to reclaim unallocated space.

> ⚠️ **Warning:** These steps will permanently delete Ubuntu and its data. Proceed only if you no longer need the Linux OS.

---

## 🛠️ Prerequisites

- Dual-boot setup (Ubuntu + Windows)
- Administrator access on Windows
- Basic familiarity with Command Prompt and Disk Management

---

## ✅ Steps to Remove Ubuntu Dual Boot

### 1. **Delete Ubuntu Partitions via Disk Management**

1. Press `Win + X` → Select **Disk Management**
2. Look for partitions **without drive letters** or labeled as **“Healthy (Primary Partition)”** used by Ubuntu
3. **Right-click** on those Ubuntu partitions and select **Delete Volume**
4. You will now see **Unallocated Space**

> 💡 Be careful not to delete Windows or recovery partitions. Double-check sizes and labels.

---

### 2. **Extend Windows Partition**

1. Right-click your main Windows partition (usually `C:`)
2. Select **Extend Volume**
3. Follow the wizard to merge the unallocated space into the Windows partition

---

### 3. **Remove Ubuntu Entry from UEFI Boot Menu**

1. Open **Command Prompt as Administrator**
2. Type the following commands:

```cmd
diskpart
list disk
select disk 0        # Replace with system disk if different
list partition
select partition 1    # Usually EFI System Partition (~100 MB)
assign letter=P       # Temporarily assign a drive letter
exit
````

3. Now navigate and delete Ubuntu boot entry:

```cmd
P:
cd EFI
dir
rd ubuntu /s
```

* You will be prompted: `Are you sure (Y/N)?` → Press **Y**
* Then:

```cmd
dir   # Confirm ubuntu folder is gone
exit
```

---

### 4. **Restart the System**

* Reboot your PC to confirm Ubuntu no longer appears in the boot menu
* Windows should boot directly

---

## 📌 Optional: Clean UEFI Boot Entries (Advanced)

To remove leftover boot entries from UEFI (optional):

```cmd
bcdedit /enum firmware
```

Look for entries related to Ubuntu, then:

```cmd
bcdedit /delete {bootmgr ID}
```

---

## ✅ Completion Checklist

* [x] Ubuntu partitions removed
* [x] Windows partition extended
* [x] Ubuntu UEFI boot entry deleted
* [x] System boots directly into Windows

---


```
