# Breon SMS - Multi-Tenant Docker Stack Deployment

## Status: ✅ Deployed & Running

---

## 🚀 Architecture Overview

Breon SMS is a multi-tenant Laravel application containerized using Docker Compose.

```
       [ Client Request ]
               │
               ▼
        ┌─────────────┐
        │  Nginx Web  │
        └──────┬──────┘
               │
     ┌─────────┴─────────┐
     ▼                   ▼
┌──────────┐      ┌─────────────┐
│  Laravel │      │  Node.js /  │
│ App (PHP)│      │ Build Tools │
└────┬─────┘      └─────────────┘
     │
     ▼
┌──────────┐
│  MySQL   │
│ Database │
└──────────┘
```

---

## ⚙️ Stack Components & Containers

| Service Name | Description | Base Image / Environment |
| :--- | :--- | :--- |
| **webserver** | Web server reverse proxy | `nginx:alpine` |
| **app** | Laravel PHP application backend | `php:8.x-fpm` |
| **db** | Primary relational database | `mysql:8.0` |
| **frontend** | Frontend asset building | `node:lts-alpine` |

---

## 🔒 Network & Remote Access
* Managed via **Tailscale** overlay network.
* Access restricted via internal Tailscale IP routing and domain mapping.