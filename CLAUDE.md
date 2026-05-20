# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This is an infrastructure runbook repository for installing and operating Oracle APEX on Linux servers. It contains installation guides, shell scripts, configuration files, and documentation organized by server environment and component. There is no application code, build system, or test suite.

## Repository Architecture

The repo is split across four server environments, each with numbered component folders:

| Folder | Role |
|--------|------|
| `dev-apex-server/` | Development environment for Oracle APEX |
| `prod-apex-server/` | Production environment for Oracle APEX |
| `dev-pdf-server/` | Development environment for PDF processing |
| `prod-pdf-server/` | Production environment for PDF processing |

**APEX server stack** (both dev and prod):
1. `01_portainer/` — Docker container management UI
2. `02_nginx/` — Reverse proxy (terminates HTTPS, routes to ORDS)
3. `03_oracle_database/` — Oracle Database 19c/21c/23c
4. `04_ords/` — Oracle REST Data Services (serves APEX over HTTP)

**PDF server stack** (both dev and prod):
1. `01_portainer/` — Docker container management UI
2. `02_nginx/` — Reverse proxy
3. `03_apache/` — Apache HTTP Server
4. `04_python/` — Python PDF processing scripts
5. `05_n8n/` — n8n workflow automation

Each component folder has four standard sub-folders:

```
<component>/
  configs/      # Config files deployed to the server
  docs/         # Documentation specific to this component
  scripts/      # Shell scripts for install/configure/maintain
  screenshots/  # Reference screenshots
```

## Key Installation Flow

The master installation guide is `README.md`. The high-level APEX installation order is:

1. Install Oracle Database and set `ORACLE_SID`, `ORACLE_HOME`, `ORACLE_BASE` env vars
2. Install APEX into the PDB via sqlplus: `@apexins.sql SYSAUX SYSAUX TEMP /i/`
3. Copy APEX static files to `/var/www/html/i/`
4. Install and configure ORDS, pointing it at the PDB on port 1521
5. Run ORDS as a systemd service on port 8080
6. (Optional) Place Nginx in front of ORDS to terminate TLS

APEX is accessed at `http://SERVER-IP:8080/ords/apex` (workspace: `internal`, user: `ADMIN`).

## Conventions

- Scripts go in the `scripts/` sub-folder of the relevant component; configs go in `configs/`.
- Follow the numbered prefix order (01, 02, …) — it reflects the deployment dependency order.
- The dev and prod environments mirror each other in structure; document differences in component-level `docs/` folders.
- Shell scripts target Oracle Linux 8/9 / RHEL 8/9 (uses `dnf`, `systemctl`, `firewall-cmd`).
- Oracle Database is installed as the `oracle` OS user; ORDS runs under the same user account.
