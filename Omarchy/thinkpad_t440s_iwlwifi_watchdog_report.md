# ThinkPad T440s Intel Wi-Fi Watchdog Report

## Status

**Installed and recovering the recurring Wi-Fi failure.** The watchdog restores the built-in Wi-Fi connection after detected driver/device failures. The underlying cause of those failures remains unresolved.

## Purpose

The built-in Intel Dual Band Wireless-AC 7260 in a Lenovo ThinkPad T440s repeatedly stopped working shortly after connecting. The issue occurred across multiple Wi-Fi networks and had also been observed in Windows. Reloading the Linux Wi-Fi driver restored the connection temporarily, but the failure recurred.

The watchdog was created to automatically recover the built-in card when the kernel reports selected serious `iwlwifi` failures, while leaving the separate ZTop USB Wi-Fi adapter alone.

## System

- **Operating system:** Omarchy (Arch Linux)
- **Built-in Wi-Fi:** Intel Dual Band Wireless-AC 7260
- **PCI device:** `0000:03:00.0`
- **Interface:** `wlp3s0`
- **Driver modules:** `iwlwifi` and `iwlmvm`
- **Firmware:** `17.bfb58538.0` (`7260-17.ucode`)
- **USB fallback:** ZTop adapter using a separate driver
- **Kernel captured in verification logs:** `7.2.8-9-omarchy`

## Watchdog behavior

The systemd service `iwlwifi-watchdog.service` follows kernel messages and filters for the built-in Intel device at `0000:03:00.0`. It reacts to these message patterns:

- `transaction failed`
- `stuck for 10000 ms`
- `Failed to start INIT ucode`
- `Could not load the [0] uCode section`

On a match, the script attempts to unload `iwlmvm` and `iwlwifi`, waits two seconds, and reloads `iwlmvm` with `power_scheme=1` (active). It does not force module unload if removal fails, does not restart NetworkManager, does not toggle the global Wi-Fi radio, and does not monitor or control the ZTop adapter.

Recovery is limited to three attempts per hour, with a five-minute cooldown between attempts. The systemd unit restarts the watchdog process if it exits.

## Installation

The prepared watchdog files were installed to the system locations and the service was enabled and started with:

```bash
sudo install -Dm755 /home/pumpkin/Agents-init/WifiCardWatchdog/iwlwifi-watchdog /usr/local/sbin/iwlwifi-watchdog
sudo install -Dm644 /home/pumpkin/Agents-init/WifiCardWatchdog/iwlwifi-watchdog.service /etc/systemd/system/iwlwifi-watchdog.service
sudo systemctl daemon-reload
sudo systemctl enable --now iwlwifi-watchdog.service
```

## Verification

On October 31, 2026 (UTC+08:00), logs from kernel `7.2.8-9-omarchy` showed the watchdog detecting and recovering two Intel Wi-Fi transaction failures:

1. At `09:18:26`, the kernel reported `iwlwifi transaction failed`; mapped device registers read `0xffffffff`. The watchdog reloaded the modules, and `wlp3s0` associated again at `09:18:39`.
2. At `09:35:39`, after the user toggled Wi-Fi, another transaction failure occurred while NetworkManager requested station statistics. The watchdog reloaded the modules, logged success at `09:35:48`, and NetworkManager reconnected at `09:35:52` with a DHCP lease.

At `09:36:06`, after the second recovery and reconnection, the kernel also logged `Hardware error detected`, firmware error `NMI_INTERRUPT_HOST`, and `Device error - SW reset`. The driver logged `restart completed` at `09:36:07`. Thus the watchdog recovery was verified, but the device continued to report a hardware/firmware fault afterward. The available logs do not establish whether that later event was an independent failure or part of the same episode.

## Monitoring commands

Check service state:

```bash
systemctl status iwlwifi-watchdog.service
```

Follow watchdog recovery messages:

```bash
sudo journalctl -u iwlwifi-watchdog.service -f
```

View built-in Intel Wi-Fi kernel messages from the last ten minutes:

```bash
sudo journalctl -k --since "10 minutes ago" | grep -Ei 'iwlwifi|iwlmvm|wlp3s0|0000:03:00.0'
```

## Current conclusion and follow-up

The watchdog is a recovery workaround, not a root-cause fix. The logs show that the Intel device becomes inaccessible during transaction failures and later reports firmware/hardware errors. They do not isolate whether the fault is in the Wi-Fi card, its PCIe seating/connection, or a platform/kernel/firmware interaction.

The next planned checks are to boot the available `7.2.11-arch1-1` kernel and observe whether failures recur, then reseat the internal Wi-Fi card if they do. If reseating does not resolve the problem, test a known-good compatible card to help distinguish a faulty card from its connection or platform.

