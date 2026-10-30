# Technical Report: Omarchy System Menu Logout Function Override

## Overview
This report details the modification of Omarchy's internal system menu structure to ensure the GUI **System > Logout** option performs a reliable process cleanup and returns the user to the display manager (SDDM).

---

## Problem Statement

Omarchy's system launcher/bar provides a graphical menu containing **System > Logout**. By default, Omarchy routes session logouts through system binaries (such as `/usr/bin/omarchy-system-logout`) designed around standard `uwsm` session management.

On environments running direct Hyprland sessions, invoking this default menu item resulted in hung processes or failure to drop back to SDDM.

---

## Root Cause Analysis

- Omarchy's GUI shell (`~/.config/omarchy/shell.json`) defines UI component layout (`omarchy.power`, `omarchy.menu`), but external shell actions (like logout/reboot) are delegated to system-level binaries located in `/usr/bin/` or path-resolved script hooks.
- Because `omarchy-system-logout` relied on a non-active session manager service, triggering logout via the GUI menu failed to issue a direct `SIGTERM`/`SIGKILL` to the `Hyprland` process.

---

## Solution & Implementation

To resolve this across both user-specific environments and system-wide calls, a two-tiered binary override was established.

### Method 1: User-Level Executable Override (Recommended)

Create a custom wrapper in the user's local path (`~/.local/bin`) to take priority in `$PATH`:

1. Ensure `~/.local/bin` exists:
   ```bash
   mkdir -p ~/.local/bin
   ```

2. Create the executable script:
   ```bash
   nvim ~/.local/bin/omarchy-system-logout
   ```

3. Insert the following execution block:
   ```bash
   #!/usr/bin/env bash
   pkill -i Hyprland
   ```

4. Grant execution permissions:
   ```bash
   chmod +x ~/.local/bin/omarchy-system-logout
   ```

---

### Method 2: System-Wide Script Override

For multi-user profiles where path overrides may be bypassed by absolute binary calls inside desktop menus, update the main system executable directly:

1. Edit `/usr/bin/omarchy-system-logout` with elevated privileges:
   ```bash
   sudo nvim /usr/bin/omarchy-system-logout
   ```

2. Replace the file content with:
   ```bash
   #!/usr/bin/env bash
   pkill -i Hyprland
   ```

3. Re-apply executable permissions:
   ```bash
   sudo chmod +x /usr/bin/omarchy-system-logout
   ```

---

## Verification & Results

1. Open the Omarchy desktop menu using **`SUPER`** / **`SUPER + ESC`**.
2. Navigate to **System** -> **Logout**.
3. Confirm that the GUI menu invocation immediately executes `pkill -i Hyprland` and successfully lands on the SDDM login screen.