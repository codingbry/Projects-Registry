# Hyprland Compositor Blur Unblocking & Aesthetic Tuning Guide

## Overview

In the `end-4/dots-hyprland` (`illogical-impulse`) environment on Omarchy Linux, application windows rendered flat transparency without background blur even when global blur was enabled. This report documents unblocking window-level blur overrides in `rules.lua` and fine-tuning the compositor's frosted-glass parameters in `general.lua`.

---

## Problem & Analysis

1. **Default Blanket Override:** The stock `illogical-impulse` configuration enforces a global rule inside `rules.lua` that explicitly disables blur for all regular application surfaces:
   ```lua
   hl.window_rule({match = {class = ".*" }, no_blur = true })
   ```
2. **Terminal Surface Incompatibility:** Wayland compositors require transparent terminal surfaces (such as Kitty) to explicitly permit compositor blur passes and have an internal background opacity defined (`background_opacity`).

---

## Implementation Steps

### Step 1: Unblock Global Window Blur in `rules.lua`
Open `~/.config/hypr/hyprland/rules.lua` and comment out the blanket `no_blur` rule:

```lua
-- Disable blur for xwayland context menus
hl.window_rule({match = {class = "^()$", title = "^()$" },                   no_blur = true })

-- Disable blur for every window (DISABLED TO ALLOW COMPOSITOR BLUR)
-- hl.window_rule({match = {class = ".*" }, no_blur = true })
```

### Step 2: Tune Frosted Glass Aesthetics in `general.lua`
Open `~/.config/hypr/hyprland/general.lua` and adjust the `decoration.blur` table inside `hl.config({})`:

```lua
decoration = {
    rounding_power = 2.5,
    rounding = 18,

    active_opacity = 0.90,
    inactive_opacity = 0.75,

    blur = {
        enabled = true,
        size = 5,                   -- Lower radius for sharp, readable backdrop detail
        passes = 2,                 -- Optimized pass count for performance and subtle blur
        new_optimizations = true,
        xray = true,                -- Keeps backdrops light and renders efficiently
        noise = 0.0,                -- Clean glass texture without visual noise
        contrast = 1.0,
        brightness = 1.0,
        vibrancy = 0.2,
    },
}
```

### Step 3: Enable Terminal Surface Transparency & Rule Opt-In
To allow terminal surfaces (Kitty) to receive the compositor blur pass:

1. Update `~/.config/kitty/kitty.conf`:
   ```ini
   background_opacity 0.82
   ```
2. Add an explicit opt-in rule at the bottom of the window rules section in `~/.config/hypr/hyprland/rules.lua`:
   ```lua
   -- Force compositor blur pass for Kitty terminal
   hl.window_rule({match = {class = "^(kitty)$" }, no_blur = false })
   ```

---

## Verification & Status

1. Reloaded Hyprland configuration via `Super + Shift + R`.
2. Restarted Kitty terminal instances to refresh the alpha channel.
3. Verified smooth, subtle acrylic blur behind active windows and terminal surfaces without performance degradation.

* **Status:** **Resolved & Stable**