# SDDM qylock Theme Setup Guide

## Overview
This document outlines the installation, directory structure resolution, and configuration process for deploying the `qylock` SDDM theme suite—specifically the `pixel-hollowknight` variant—on an Arch/Hyprland system running SDDM.

---

## Technical Background & Incident Summary

### Context
Previous display manager configurations relied on `sddm-astronaut-theme`. Upon removing `sddm-astronaut-theme` and attempting to transition to `qylock`, SDDM failed to render the graphic interface, dropping to fallback or displaying layout errors.

### Root Cause
1. **Dynamic Installer Behavior**: The `qylock` setup script (`sddm.sh`) does not create a monolithic theme folder titled `qylock`. Instead, it prompts the user to select a specific theme (e.g., `pixel-hollowknight`, `nier_automata`, `pixel_coffee`) and generates a corresponding named subfolder inside `/usr/share/sddm/themes/`.
2. **Configuration Mismatch**: SDDM configuration files (`/etc/sddm.conf` and `/etc/sddm.conf.d/theme.conf`) remained pointed to `Current=qylock` or `sddm-astronaut-theme`, causing SDDM to fail to locate a valid `Main.qml` file.

---

## Step-by-Step Implementation

### Step 1: Execute Theme Installer
Run the `qylock` Qt6 installation script:

```bash
./sddm.sh
```

During installation, select the `pixel-hollowknight` theme variant from the menu prompt.

### Step 2: Verify Installed Theme Directory
Verify that the installer created the target directory under `/usr/share/sddm/themes/`:

```bash
ls -la /usr/share/sddm/themes/
```

*Expected output line:*
```text
drwxr-xr-x - root 28 Oct pixel-hollowknight
```

---

### Step 3: Update SDDM System Configurations

#### 1. Main SDDM Configuration (`/etc/sddm.conf`)
Edit `/etc/sddm.conf` to update the active theme pointer:

```ini
[Theme]
Current=pixel-hollowknight
```

#### 2. Drop-in Theme Override (`/etc/sddm.conf.d/theme.conf`)
Update the drop-in theme override file to ensure global consistency across service restarts:

```ini
[Theme]
Current=pixel-hollowknight

[Users]
RememberLastUser=false
MinimumUid=1000
MaximumUid=60513
```

---

## Verification & Testing

Restart the SDDM service to apply changes live:

```bash
sudo systemctl restart sddm
```

### Results
- The login screen successfully renders the **Pixel Hollow Knight** theme.
- Obsolete preview references (e.g., `Screenshot=Previews/astronaut.png`) from previous setups were fully cleared with the uninstallation of `sddm-astronaut-theme`.

---

## Architectural Note: Theme Switching Procedure
To switch to a different `qylock` variant in the future:
1. Re-run `./sddm.sh` and choose a new variant (e.g., `nier_automata`).
2. Identify the new directory created in `/usr/share/sddm/themes/`.
3. Update `Current=<new-folder-name>` in both `/etc/sddm.conf` and `/etc/sddm.conf.d/theme.conf`.
4. Restart SDDM via `systemctl restart sddm`.