# Ubuntu Server Hardware & Storage Setup

## Status: ⚠️ Not Yet Implemented (Planning / In Progress)

---

## 📋 Planned Architecture & Goals

### 1. Host Infrastructure
* **Operating System**: Ubuntu Server LTS
* **Access**: Passwordless SSH via key pair authentication

### 2. Storage & Volume Management
* **File System**: `ext4`
* **Volume Manager**: LVM (Logical Volume Manager) setup for dynamic partition scaling.
* **Hardware Interconnects**: Drive expansion utilizing motherboard SATA connections and power supply splitters.

---

## 📝 Roadmap & Next Steps
- [ ] Complete host OS installation.
- [ ] Physical installation of storage drives and cabling.
- [ ] Initialize Physical Volumes (`pvcreate`), Volume Groups (`vgcreate`), and Logical Volumes (`lvcreate`).
- [ ] Mount points configuration in `/etc/fstab`.