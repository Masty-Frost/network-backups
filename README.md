# 🌐 Network Backups
 
Automated network device configuration backup system using **NetBox** as inventory source and **Oxidized** as the backup engine. Deployable in minutes with Docker.
 
---
 
## What it does
 
- Reads the device inventory from NetBox (routers, switches, firewalls...)
- Connects to each device via SSH and pulls its current configuration
- Stores versioned backups automatically
- Exposes a web interface to browse backup history per device
---
 
## Stack
 
| Component | Role |
|---|---|
| [NetBox](https://github.com/netbox-community/netbox) | Network inventory & source of truth |
| [Oxidized](https://github.com/ytti/oxidized) | Configuration backup engine |
| [PostgreSQL](https://www.postgresql.org/) | NetBox database |
| [Redis](https://redis.io/) | NetBox cache & task queue |
| [Docker](https://www.docker.com/) | Container orchestration |
 
---
 
## Requirements
 
- Docker 20.10+
- Docker Compose 1.28+
---
 
## Quick Start
 
```bash
git clone https://github.com/Masty-Frost/network-backups.git
cd network-backups
cp .env.example .env
# Edit .env with your values
docker compose up -d
```
 
NetBox will be available at `http://localhost:8000`
Oxidized will be available at `http://localhost:8888`
 
---
 
## Documentation
 
- [Installation Guide](INSTALLATION.md) — full setup instructions
- [Troubleshooting](TROUBLESHOOTING.md) — common issues and fixes
---
 
## Based on
 
Built on top of [netbox-community/netbox-docker](https://github.com/netbox-community/netbox-docker).
