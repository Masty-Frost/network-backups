# Installation Guide

## Prerequisites

Before starting, make sure you have installed:

- **Docker** (version 20.10 or higher)
  ```bash
  docker --version
  ```

- **Docker Compose** (version 1.29 or higher)
  ```bash
  docker compose --version
  ```

- **Git** (for cloning the repository)
  ```bash
  git --version
  ```

---

## Step 1: Clone the Repository

```bash
git clone https://github.com/Masty-Frost/network-backups.git
cd network-backups
```

---

## Step 2: Configure Environment Variables

Copy the example environment file:

```bash
cp .env.example .env
```

Edit `.env` with your values:

```bash
nano .env
```

Or use your preferred editor. Configure:

```env
OXIDIZED_USERNAME=oxidized
OXIDIZED_PASSWORD=your_secure_password
NETBOX_TOKEN=your_netbox_token_here
NETBOX_URL=http://netbox:8080
```

> ⚠️ **Important:** Keep the `.env` file secure and never commit it to Git.

---

## Step 3: Start the Containers

```bash
docker compose up -d
```

Verify all containers are running:

```bash
docker compose ps
```

Expected output:

```
NAME                COMMAND             STATUS
netbox              python manage.py    Up 2 minutes
netbox-postgres     postgres            Up 2 minutes
netbox-redis        redis-server        Up 2 minutes
oxidized            oxidized            Up 2 minutes
```

---

## Step 4: Verify Services are Running

**Check NetBox:**

```bash
curl -s http://localhost:8000/api/ | head -20
```

Should return JSON data from the NetBox API.

**Check Oxidized:**

```bash
curl -s http://localhost:8888/api/nodes | head -20
```

Should return JSON with device status.

---

## Step 5: Initial Setup in NetBox

1. Open `http://localhost:8000` in your browser.
2. Log in with:
   - **Username:** `admin`
   - **Password:** `admin` (default)

> ⚠️ **Important:** Change the default admin password immediately!

3. Navigate to: **Admin → Users → admin**
4. Click **"Change Password"**
5. Enter a new secure password.

---

## Step 6: Generate NetBox API Token

1. In NetBox, go to **Admin → Tokens**
2. Click **"Add Token"**
3. Select user: `admin`
4. Click **"Generate Token"**
5. Copy the token
6. Update your `.env` file:

```env
NETBOX_TOKEN=nbt_xxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

---

## Step 7: Restart Oxidized

Apply the new token:

```bash
docker compose restart oxidized
```

Wait 30 seconds for Oxidized to initialize.

---

## Step 8: Verify Installation

Check if everything is working:

```bash
# View Oxidized logs
docker compose logs -f oxidized

# You should see messages like:
# [INFO] Reloading config
# [INFO] Loading devices from NetBox
```

Open the Oxidized web interface: `http://localhost:8888`

You should see an empty device list (no devices added yet).

---

## Troubleshooting Installation

### Containers Not Starting

Check logs:

```bash
docker compose logs
```

Common issues:

- Port `8000` or `8888` already in use
- Docker daemon not running
- Insufficient disk space

### NetBox Not Accessible

```bash
# Check if container is running
docker ps | grep netbox

# View logs
docker compose logs netbox

# Restart
docker compose restart netbox
```

Wait 1–2 minutes after restart (NetBox initialization takes time).

### Permission Denied Error

If you see `permission denied`:

```bash
# Add your user to the docker group
sudo usermod -aG docker $USER

# Apply new group
newgrp docker

# Try again
docker compose ps
```

---

## Next Steps

After successful installation:

1. Read the [How to Use](USE.md) guide
2. Add devices to the NetBox inventory
3. Configure Oxidized for your devices
4. Verify backups are running

---

## Getting Help

If you encounter issues:

1. View container logs:
   ```bash
   docker compose logs -f
   ```
2. Verify connectivity to devices:
   ```bash
   ssh user@device_ip
   ```

---

## System Requirements

|          |       Minimum       |     Recommended     |
|----------|---------------------|---------------------|
| **CPU**  |       2 cores       |       4 cores       |
| **RAM**  |        2 GB         |        4 GB         |
| **Disk** | 10 GB (for backups) | 50 GB (for backups) |
