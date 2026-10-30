# SDDM qylock Theme Setup & Management Guide

## Overview
This document outlines the installation, configuration hierarchy, theme selection picker, and troubleshooting workflow for deploying and dynamically switching themes using the `qylock` SDDM theme suite on Arch/Hyprland.

---

## Technical Background & Incident Summary

### Context
Previous display manager configurations relied on static themes like `sddm-astronaut-theme`. Transitioning to `qylock` caused SDDM rendering failures or fallback behavior due to configuration precedence conflicts and installer mechanics.

### Root Cause Analysis
1. **Dynamic Installer & Subfolder Generation**: The `qylock` setup scripts (`sddm.sh` / `quickshell.sh`) create specific variant directories inside `/usr/share/sddm/themes/` (e.g., `pixel-hollowknight`, `nier_automata`) rather than a single monolithic folder named `qylock`.
2. **Hardcoded Configuration Precedence**: System configuration files (`/etc/sddm.conf` and `/etc/sddm.conf.d/theme.conf`) explicitly set `Current=pixel-hollowknight` or legacy paths. Because drop-in configurations take precedence over script-based updates, active calls to `quickshell.sh` were ignored by SDDM.

---

## Step-by-Step Implementation

### Step 1: Clear System Overrides for Dynamic Switching
To allow the interactive launcher (`quickshell.sh`) to control and swap active themes without manual config editing, ensure hardcoded theme keys are commented out or removed.

#### 1. Edit Drop-in Override (`/etc/sddm.conf.d/theme.conf`)
```ini
[Theme]
# Current=pixel-hollowknight ; Commented out to enable quickshell.sh dynamic picker

[Users]
RememberLastUser=false
MinimumUid=1000
MaximumUid=60513
```

#### 2. Edit Main Configuration (`/etc/sddm.conf`)
Ensure `[Theme]` does not force a hardcoded `Current=` directive:
```ini
[Theme]
# Current=pixel-hollowknight
```

---

### Step 2: Execute Theme Picker Script
Make `quickshell.sh` executable and run the interactive selection interface:

```bash
chmod +x quickshell.sh && ./quickshell.sh
```

Choose your desired theme variant (e.g., `pixel-hollowknight`, `nier_automata`, or `pixel_coffee`) from the interactive prompt.

---

### Step 3: Verify Installed Theme Directory
Verify that the installer generated or updated the target directory under `/usr/share/sddm/themes/`:

```bash
ls -la /usr/share/sddm/themes/
```

*Expected output structure:*
```text
drwxr-xr-x - root  pixel-hollowknight
drwxr-xr-x - root  nier_automata
```

---

## Non-Destructive Testing & Verification

Instead of restarting the entire SDDM service (which terminates active Wayland/Hyprland sessions), test theme rendering live inside a window:

### 1. Test Windowed Rendering
Run the SDDM greeter in test mode for the chosen theme:

```bash
sddm-greeter-qt6 --test-mode --theme /usr/share/sddm/themes/pixel-hollowknight
```

### 2. Live Service Restart (Optional)
Once confirmed, restart the system service to apply changes across full reboots:

```bash
sudo systemctl restart sddm
```

---

## Workflow: Dynamic Theme Switching

To switch variants seamlessly in the future:

1. **Run Picker**: Execute `./quickshell.sh` to choose a new variant.
2. **Test Theme**: Run `sddm-greeter-qt6 --test-mode --theme /usr/share/sddm/themes/<selected-theme>` to preview graphics and layout.
3. **Reboot or Lock**: SDDM will automatically pull the selected theme without needing manual config edits as long as `/etc/sddm.conf.d/theme.conf` remains un-hardcoded.