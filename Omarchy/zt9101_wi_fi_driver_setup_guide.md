# ZTopInc ZT9101 (350b:9101) Wi-Fi Adapter Setup & Hotplug Fix

This documentation details the process of installing, persistent-boot configuring, and stabilizing the **ZTopInc 802.11n NIC** (`zt9101_ztopmac_usb` vendor driver) on Linux.

---

## 1. Hardware & System Context

* **Device:** ZTopInc 802.11n NIC
* **USB ID:** `350b:9101`
* **Kernel Module:** `zt9101_ztopmac_usb`
* **Target Environment:** Designed for seamless boot initialization and child-safe USB hot-plugging without kernel crashes or hangs.

---

## 2. Key Challenges & Root Cause Analysis

### The Hardcoded Firmware Path Issue
The vendor out-of-tree driver contains a hardcoded firmware loader looking for `./fw/ZT9101_fw_c1ab8ba.bin`. 
* Standard Linux system paths (`/lib/firmware` or `/usr/lib/firmware`) fail because the execution path for system services and standard kernel events defaults to the root directory (`/`).
* **Symptom:** Kernel logs (`dmesg`) report `[E]firmware open failed` upon hotplug or boot:
  ```text
  [I]download firmware fullpath://fw/ZT9101_fw_c1ab8ba.bin
  [E]firmware open failed
  [E]Can't get firmware for downloading
  ```

### Hotplug / Reboot Inconsistency
* On cold boots or hot-unplug events, the interface fails to appear unless the module is cleanly loaded *after* firmware resolution succeeds.
* Heavy script commands inside `udev` `RUN+=` rules can be blocked by `systemd-udevd` sandbox restrictions.

---

## 3. Step-by-Step Resolution

### Step 1: Fix Firmware Path Resolution Permanently
To satisfy the driver's hardcoded relative lookup (`//fw/`), create a root-level `/fw` directory and copy the firmware binaries into it:

```bash
sudo mkdir -p /fw
sudo cp -r /home/pumpkin/ZT9101_vendor_linux_driver-main/ZT9101_android_V1.2.1800.20241115/fw/* /fw/
```

### Step 2: Ensure Automatic Module Loading at Boot
Add the kernel module to the system startup configuration:

```bash
echo "zt9101_ztopmac_usb" | sudo tee /etc/modules-load.d/zt9101.conf
```

### Step 3: Configure Udev Hotplug Matching Rule
Create a targeted `udev` rule at `/etc/udev/rules.d/99-zt9101-wifi.rules` to match the exact hardware IDs (`350b:9101`):

```text
ACTION=="add", SUBSYSTEM=="usb", ATTRS{idVendor}=="350b", ATTRS{idProduct}=="9101"
```

Reload `udev` rules:
```bash
sudo udevadm control --reload-rules
```

---

## 4. Testing & Verification

1. **Hot-Unplug Test:** Physical disconnect and reconnect loads the driver cleanly and detects `/fw/ZT9101_fw_c1ab8ba.bin`.
2. **Interface Check:**
   ```bash
   ip link
   ```
   *Expected Output:* `wlp0s20u1` (or equivalent wireless interface name) appears state `UP`/`DOWN` as ready for NetworkManager.
3. **Reboot Persistence:** System reboots maintain interface availability without requiring manual `modprobe` commands.

---

## 5. Maintenance & Safety Notes

* **Accidental Unplugs:** With `/fw/` configured, pulling out the USB adapter no longer causes driver initialization errors on re-insertion.
* **Firmware Directory:** Do not delete `/fw` on the root filesystem, as the vendor driver strictly relies on this directory location.