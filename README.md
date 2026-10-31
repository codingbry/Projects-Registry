# Projects Registry

Welcome to my central technical repository. This space serves as a master registry and log for all my ongoing software engineering projects, system administration tasks, homelab infrastructure fixes, and academic developments. 

Each report inside this repository documents technical configurations, incident root-cause analyses, deployment steps, or architectural plans. Browse the registry below to access individual project documentations.

---

## 📂 Project Index & Technical Reports

| Category | Project / System | Status | Direct Report Link |
| :--- | :--- | :--- | :--- |
| **Omarchy** | ZTopInc USB Wi-Fi Adapter Fix | **Resolved & Stable** | [View Fix Report](./Omarchy/zt9101_wi_fi_driver_setup_guide.md) |
| **Omarchy** | ThinkPad T440s Intel Wi-Fi Watchdog | **Installed; Recovery Verified** | [View Watchdog Report](./Omarchy/thinkpad_t440s_iwlwifi_watchdog_report.md) |
| **Omarchy** | Multi-User End-4 Hyprland Setup | **Resolved & Stable** | [View Setup Guide](./Omarchy/omarchy_multi_user_end_4_hyprland_setup.md) |
| **Omarchy** | SDDM `qylock` Theme Integration | **Resolved & Stable** | [View Setup Guide](./Omarchy/sddm_qylock_theme_setup_guide.md) |
| **Omarchy** | Omarchy Session Logout Keybind Fix | **Resolved & Stable** | [View Fix Report](./Omarchy/omarchy_session_logout_keybind_fix.md) |
| **Omarchy** | Omarchy System Menu Logout Override | **Resolved & Stable** | [View Fix Report](./Omarchy/omarchy_system_menu_logout_override.md) |
| **Omarchy** | Hyprland Global Opacity Fix | **Resolved & Stable** | [View Fix Report](./Omarchy/hyprland_global_opacity_fix_technical_report.md) |
| **Omarchy** | Hyprland Compositor Blur Tuning | **Resolved & Stable** | [View Setup Guide](./Omarchy/hyprland_compositor_blur_tuning_technical_report.md) |
| **Homelab** | Ubuntu Server Hardware Lab | **Not Yet Implemented** | [View Storage Notes](./homelab/ubuntu-server-lab.md) |
| **Homelab** | Pi-hole Gravity Update & Docker DNS Fix | **Resolved & Stable** | [View Fix Report](./homelab/pi_hole_gravity_resolution_fix_incident_report.md) |
| **DevOps** | Breon SMS (Laravel / Docker Stack) | **Deployed & Running** | [View Deployment Log](./devops/breon-sms/breon-sms-deployment.md) |
| **App Scripts** | Office Mini Honesty Store (Google Apps Script) | **Setup documented** | [View Project Guide](./appscripts/honesty-store.md) |
| **Academic** | ADDAI Capstone Project | **Active Development** | [View Overview](./academic/addai-capstone/addai-capstone-overview.md) |
| **Academic** | TimetableMaster | **Not Yet Implemented** | [View Planning Notes](./academic/timetablemaster/timetablemaster-plan.md) |

---

## 🛠️ Repository Layout

To maintain clarity and support multiple reports under the same category, documentation files use unique, descriptive names instead of generic `README.md` titles:

```text
.
├── README.md                          <-- Main index & overview
├── Omarchy/
│   ├── zt9101_wi_fi_driver_setup_guide.md
│   ├── thinkpad_t440s_iwlwifi_watchdog_report.md
│   ├── omarchy_multi_user_end_4_hyprland_setup.md
│   ├── sddm_qylock_theme_setup_guide.md
│   ├── omarchy_session_logout_keybind_fix.md
│   ├── omarchy_system_menu_logout_override.md
│   ├── hyprland_global_opacity_fix_technical_report.md
│   └── hyprland_compositor_blur_tuning_technical_report.md
├── appscripts/
│   └── honesty-store.md
├── homelab/
│   ├── ubuntu-server-lab.md
│   └── pi_hole_gravity_resolution_fix_incident_report.md
├── devops/
│   └── breon-sms/
│       └── breon-sms-deployment.md
└── academic/
    ├── addai-capstone/
    │   └── addai-capstone-overview.md
    └── timetablemaster/
        └── timetablemaster-plan.md
