# 🗄️ Install Oracle APEX on Linux

A complete, step-by-step installation runbook for deploying **Oracle APEX 26.1** on Linux servers — fully documented with screenshots, Docker Compose configurations, and shell commands.

> **Stack:** Oracle DB 23ai · APEX 26.1 · ORDS 26.1 · Docker · Nginx · Tailscale

---

## 🏗️ Infrastructure Overview

| Server | Role | Status |
|--------|------|--------|
| Dev — APEX Server | Development — APEX + Oracle DB | ✅ Complete |
| Dev — PDF Server | Development — PDF processing stack | ✅ Complete |
| Prod — APEX Server | Production — APEX + Oracle DB | ✅ Complete |
| Prod — PDF Server | Production — PDF processing stack | ⏳ Pending |
| Backup Server | Backup | ⏳ Pending |

---

## 📚 Documentation

### 🔧 00 — Server Preparation

Applies to all servers. Base configuration, user setup, Docker installation.

| Chapter | Description |
|---------|-------------|
| [Server Preparation](docs/00_server-preparation/server-preparation.md) | Ubuntu 22.04 setup, SSH hardening, Docker & Docker Compose install |

---

### 🗄️ 01 — Database Server (APEX)

Production Oracle APEX server

| # | Chapter | Description |
|---|---------|-------------|
| 01 | [Portainer](docs/01_database-server/01_portainer/portainer.md) | Docker management UI — container overview and stack deployment |
| 02 | [Nginx Proxy Manager](docs/01_database-server/02_nginx/nginx.md) | Reverse proxy, HTTPS termination, Let's Encrypt SSL |
| 03 | [Oracle Database 23ai](docs/01_database-server/03_database/database.md) | Oracle DB 23.26.1 EE in Docker — PDB setup, health checks |
| 04 | [Oracle APEX 26.1](docs/01_database-server/04_apex_install/apex.md) | APEX installation into ORCLPDB, REST config, admin account |
| 05 | [ORDS 26.1](docs/01_database-server/05_ords/ords.md) | Oracle REST Data Services — Docker deploy, SSL proxy fix |
| 06 | [Tailscale, Domain & Firewall](docs/01_database-server/06_tailscale/tailscale.md) | VPN setup, Strato DNS, Tailscale Services, iptables hardening |

**Live services after setup:**

| Service | URL | Access |
|---------|-----|--------|
| Oracle APEX | `https://prod-app.app.de/ords/apex` | 🌐 Public |
| Oracle DB (SQL Developer) | `prod-apex-db.app.de:1521` | 🔒 VPN only |
| Nginx Admin | `https://prod-apex-nginx.tailscale.ts.net` | 🔒 VPN only |
| Portainer | `https://prod-apex-portainer.tailscale.ts.net` | 🔒 VPN only |

---

### 📄 02 — PDF Server

Production PDF processing stack

| # | Chapter | Description |
|---|---------|-------------|
| 01 | [Portainer](docs/02_pdf-server/01_portainer/portainer.md) | Docker management UI |
| 02 | [Nginx Proxy Manager](docs/02_pdf-server/02_nginx/nginx.md) | Reverse proxy & SSL |
| 03 | [Apache + PHP](docs/02_pdf-server/03_apache/apache.md) | Apache HTTP Server with PHP 8.3, OCI8, TCPDF |
| 04 | [Python API](docs/02_pdf-server/04_python/python.md) | Python 3.13 + Flask — PDF processing REST API |
| 05 | [n8n](docs/02_pdf-server/05_n8n/n8n.md) | Workflow automation |
| 06 | [FTP Server](docs/02_pdf-server/06_ftp/ftp.md) | File transfer — delfer/alpine-ftp-server |
| 07 | [Tailscale, Domain & Firewall](docs/02_pdf-server/07_tailscale/tailscale.md) | VPN, DNS, iptables — same pattern as APEX server |

> ⏳ Documentation in progress — screenshots pending.

---

## 🧰 Tech Stack

| Component | Version / Image |
|-----------|----------------|
| OS | Ubuntu 22.04 LTS |
| Oracle Database | 23.26.1 EE (`oracle/database:23.26.1-ee`) |
| Oracle APEX | 26.1.0 |
| Oracle ORDS | 26.1.1 |
| Docker | Latest stable |
| Portainer | `portainer/portainer-ce` |
| Nginx Proxy Manager | `jc21/nginx-proxy-manager` |
| Apache | `php:8.3-apache` |
| Python | 3.13 + Flask |
| n8n | 1.88.0 |
| FTP | `delfer/alpine-ftp-server` |
| VPN | Tailscale |

---

## 🔒 Security Model

All admin interfaces are **never exposed on the public internet**. Access is controlled at two levels:

1. **Tailscale VPN** — admin tools get stable `.ts.net` domains, only reachable inside the VPN tunnel
2. **iptables `DOCKER-USER` chain** — all admin ports blocked at kernel level; only Tailscale IP range (`100.64.0.0/10`) is allowed

| Port | Service | Access |
|------|---------|--------|
| `80`, `443` | Nginx (public HTTPS) | 🌐 Open |
| `81` | Nginx admin UI | 🔒 Tailscale only |
| `1521` | Oracle DB listener | 🔒 Tailscale only |
| `8181` | ORDS direct | 🔒 Tailscale only |
| `9000` | Portainer | 🔒 Tailscale only |

---

## 📁 Repository Structure

```
docs/
├── 00_server-preparation/
│   └── server-preparation.md
├── 01_database-server/
│   ├── 01_portainer/
│   ├── 02_nginx/
│   ├── 03_database/
│   ├── 04_apex_install/
│   ├── 05_ords/
│   └── 06_tailscale/          ← VPN + DNS + iptables
└── 02_pdf-server/
    ├── 01_portainer/
    ├── 02_nginx/
    ├── 03_apache/
    ├── 04_python/
    ├── 05_n8n/
    ├── 06_ftp/
    └── 07_tailscale/
```

Each chapter folder contains:
- `<chapter>.md` — step-by-step guide with commands and explanations
- `screenshots/` — numbered screenshots matching each step

---

## 🚀 Quick Start

To deploy the APEX server from scratch, follow the chapters in order:

```
00 Server Preparation → 01 Portainer → 02 Nginx → 03 Database → 04 APEX → 05 ORDS → 06 Tailscale
```

Each guide is self-contained with all required commands, Docker Compose configs, and screenshots.

---

*Sajjad IT Admin · 2026*
