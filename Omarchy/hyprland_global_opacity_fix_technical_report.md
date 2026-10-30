# Hyprland Global Opacity & Rule Syntax Incident Report

## Executive Summary

During the custom desktop environment configuration for user `nininjoy` on Omarchy Linux (`end-4/dots-hyprland` / `illogical-impulse`), an attempt to declare global window opacity via `hl.window_rule` inside `rules.lua` resulted in an IPC compilation fault, causing a total compositor session freeze. This report details the root cause analysis, syntax incompatibility in end-4's Lua wrapper layer, and the verified solution for declaring global window opacity via `general.lua`.

---

## Incident Analysis & Root Cause

### Trigger Scenario
An unhandled key structure was passed to the `hl.window_rule` function inside `~/.config/hypr/hyprland/rules.lua`:

```lua
-- BROKEN: Caused compositor freeze during IPC reload
hl.window_rule({match = {class = ".*" }, opacity = "0.90 0.80" })
```

### Root Cause
1. **Wrapper Mapping Mismatch:** End-4's `illogical-impulse` uses custom Lua wrapper functions (`hl.window_rule`) that transpile tables into Hyprland IPC socket commands.
2. **Invalid Parameter Parsing:** The internal Lua table transpiler does not recognize `opacity` as a top-level table key inside `hl.window_rule`. 
3. **IPC Hang:** Passing an invalid table structure to the Hyprland IPC socket during live reloads (`Super + Shift + R`) crashed the Hyprland compositor daemon, locking user input and necessitating a hardware power-cycle.

---

## Remediation & Configuration Fix

To safely apply active and inactive window transparency across all application surfaces without crashing the compositor, opacity must be defined in the global `hl.config` table inside `general.lua`.

### Step 1: Clean `rules.lua`
Remove or comment out any attempted `opacity` keys inside `~/.config/hypr/hyprland/rules.lua`.

### Step 2: Configure Global Decoration in `general.lua`
Open `~/.config/hypr/hyprland/general.lua` and locate the `decoration` block within `hl.config({})`:

```lua
hl.config({
    -- ...
    general = {
        -- ...
    },
    decoration = {
        rounding_power = 2.5,
        rounding = 18,

        -- Global window opacity (Active / Inactive)
        active_opacity = 0.90,
        inactive_opacity = 0.75,

        blur = {
            enabled = true,
            -- Blur settings...
        },
        shadow = {
            enabled = true,
            -- Shadow settings...
        },
        dim_inactive = true,
        dim_strength = 0.05,
        dim_special = 0.2
    },
    -- ...
})
```

---

## Verification & Status

1. **Compositor Reload:** Re-executed Hyprland configuration reload (`Super + Shift + R`).
2. **IPC Stability:** Verified system stability with zero socket hangs or display server crashes.
3. **Visual Output:** Active windows render at **90% opacity**, while inactive windows drop to **75% opacity**.

* **Status:** **Resolved & Stable**