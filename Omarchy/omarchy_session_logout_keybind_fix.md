# Incident & Fix Report: Omarchy User Logout Keybinding

## Overview
This report documents the diagnostic process and technical resolution for fixing the session logout shortcut (`SUPER + SHIFT + E`) on user profiles utilizing the Omarchy Lua configuration framework (specifically the `pumpkin` account).

---

## Issue Description

### Symptom
Pressing **`SUPER + SHIFT + E`** in the `pumpkin` Hyprland session either triggered an arbitrary placeholder binding (`"Hey"`) or failed to gracefully terminate the compositor session back to the display manager (SDDM).

### Context
While standard Hyprland dotfiles (e.g., end-4 configs) handle keybindings using `bind = $mainMod SHIFT, E, exec, <command>` in standard `.conf` files, Omarchy routes user input through a Lua-based abstraction layer (`hl.bind` and `hl.unbind`) defined in `~/.config/hypr/bindings.lua`.

---

## Root Cause Analysis

1. **Default Override Inheritance**: Omarchy defines default keybindings system-wide. User-level keybinding files inherit these defaults unless explicitly unbound using `hl.unbind()`.
2. **Lack of Process Interruption**: Soft exit commands or unmapped script triggers did not force a clean compositor termination required by Hyprland when running under non-UWSM setups.

---

## Resolution Procedure

### Step 1: Locate Active User Bindings File
Identify the Lua keybindings configuration file for the user profile:

```bash
ls -la ~/.config/hypr/bindings.lua
```

### Step 2: Apply Unbind and Rebind Rules
Edit `~/.config/hypr/bindings.lua`:

```bash
nvim ~/.config/hypr/bindings.lua
```

Add the following unbind and rebind directive at the bottom of the file:

```lua
-- Unbind default SUPER + SHIFT + E action ("Hey")
hl.unbind("SUPER + SHIFT + E")

-- Bind SUPER + SHIFT + E to execute a clean Hyprland process kill
hl.bind("SUPER + SHIFT + E", hl.dsp.exec_cmd("pkill -i Hyprland"), { description = "Session: Clean log out to SDDM" })
```

---

### Step 3: Apply Live Configuration
Reload the Hyprland configuration live without restarting the session:

```bash
hyprctl reload
```

---

## Verification

1. Press **`SUPER + SHIFT + E`** within the active user session.
2. Confirm that `pkill -i Hyprland` terminates the Wayland compositor instance cleanly.
3. Verify that control returns directly to the configured SDDM theme (`pixel-hollowknight`).