# Troubleshooting Guide

## Common Issues and Solutions

---

### Issue 1: NetBox Not Accessible

**Symptom:** Cannot access `http://localhost:8000` or getting connection refused

**Diagnosis:**
```bash
docker ps | grep netbox
docker compose logs netbox
curl -v http://localhost:8000
```

**Solutions:**

1. Check if NetBox container is running:
   ```bash
   docker compose ps netbox
   ```

2. If not running, check logs:
   ```bash
   docker compose logs netbox
   ```

3. Restart NetBox:
   ```bash
   docker compose restart netbox
   ```

4. Wait 2–3 minutes (NetBox initialization takes time):
   ```bash
   sleep 120
   curl http://localhost:8000
   ```

5. If still not working, rebuild the container:
   ```bash
   docker compose down
   docker compose up -d netbox
   ```

---

### Issue 2: Oxidized Web Interface Not Loading

**Symptom:** Cannot access `http://localhost:8888` or page shows error

**Diagnosis:**
```bash
docker ps | grep oxidized
docker compose logs oxidized
curl http://localhost:8888
```

**Solutions:**

1. Check if Oxidized container is running:
   ```bash
   docker compose ps oxidized
   ```

2. View Oxidized logs:
   ```bash
   docker compose logs oxidized
   ```

3. Remove stale PID file:
   ```bash
   docker exec -it oxidized rm /home/oxidized/.config/oxidized/oxidized.pid
   ```

4. Restart Oxidized:
   ```bash
   docker compose restart oxidized
   ```

5. Wait 30 seconds and verify:
   ```bash
   sleep 30
   curl http://localhost:8888
   ```

---

### Issue 3: Oxidized Cannot Connect to Devices

**Symptom:** Red squares in "Last Status" section of Oxidized web interface

**Diagnosis:**
```bash
docker compose logs oxidized
docker exec oxidized curl -s http://localhost:8888/api/nodes | jq .
```

**Solutions:**

1. Verify device is reachable:
   ```bash
   ping <device-ip>
   ssh <username>@<device-ip>
   ```

2. Check Oxidized configuration and check if the model's operative system is the correct:
   ```bash
   docker exec oxidized cat /home/oxidized/.config/oxidized/config
   ```

3. Verify credentials in NetBox — log in at `http://localhost:8000`, go to **Devices → Edit device** and check the username and password fields.

4. Test SSH connection from the Oxidized container:
   ```bash
   docker exec oxidized ssh -v <username>@<device-ip>
   ```

5. Check if SSH port is open:
   ```bash
   telnet <device-ip> 22
   ```

6. Verify NetBox API token:
   ```bash
   curl -H "Authorization: Bearer YOUR_TOKEN" http://localhost:8080/api/dcim/devices/
   ```

---

### Issue 4: NetBox API Token Not Working

**Symptom:** Oxidized logs show `401 Unauthorized` or `Invalid token`

**Diagnosis:**
```bash
docker compose logs oxidized | grep -i auth
curl -H "Authorization: Bearer YOUR_TOKEN" http://localhost:8000/api/
```

**Solutions:**

1. Verify token in `.env`:
   ```bash
   cat .env | grep NETBOX_TOKEN
   ```

2. Check token exists in NetBox — log in at `http://localhost:8000`, go to **Admin → Tokens** and verify the token is active.

3. Generate a new token if needed — in NetBox go to **Admin → Tokens → Add Token** and copy the new token.

4. Update `.env`:
   ```bash
   nano .env
   # Update: NETBOX_TOKEN=nbt_new_token_here
   ```

5. Restart Oxidized:
   ```bash
   docker compose restart oxidized
   ```

---

### Issue 5: Backup Files Not Being Created

**Symptom:** No files in `/backups/oxidized/` directory

**Diagnosis:**
```bash
ls -la ~/.config/oxidized/configs/
docker exec oxidized ls -la /home/oxidized/.config/oxidized/configs/
```

**Solutions:**

1. Check if devices exist in NetBox — log in at `http://localhost:8000` and verify devices are listed.

2. Verify Oxidized is reading from NetBox:
   ```bash
   docker compose logs -f oxidized | grep -i netbox
   ```

3. Check backup directory permissions:
   ```bash
   ls -la ~/.config/oxidized/
   chmod 755 ~/.config/oxidized/configs
   ```

4. Manually trigger a backup:
   ```bash
   docker exec -it oxidized oxidized -c /home/oxidized/.config/oxidized/config
   ```

---

### Issue 6: Permission Denied Errors

**Symptom:** `permission denied` when running docker commands

**Diagnosis:**
```bash
docker ps
# Error: permission denied while trying to connect to the Docker daemon
```

**Solutions:**

1. Add user to the docker group:
   ```bash
   sudo usermod -aG docker $USER
   ```

2. Apply group changes:
   ```bash
   newgrp docker
   ```

3. Verify:
   ```bash
   docker ps
   ```

4. If still not working, restart Docker:
   ```bash
   sudo systemctl restart docker
   ```

---

### Issue 7: High CPU or Memory Usage

**Symptom:** System is slow, containers consuming too many resources

**Diagnosis:**
```bash
docker stats
docker compose logs
```

**Solutions:**

1. Check for memory leaks:
   ```bash
   docker stats --no-stream
   ```

2. Reduce number of threads in Oxidized config:
   ```yaml
   # Edit oxidized/config
   # Change: threads: 30 → threads: 5
   ```
   ```bash
   docker compose restart oxidized
   ```

3. Reduce polling interval:
   ```yaml
   # Edit oxidized/config
   # Change: interval: 3600 → interval: 7200 (2 hours)
   ```
   ```bash
   docker compose restart oxidized
   ```

4. Clean up unused images and volumes:
   ```bash
   docker system prune -a
   docker volume prune
   ```

---

### Issue 8: Cannot Access from Other Devices on Network

**Symptom:** Can access `localhost:8000` but not from other computers on LAN

**Diagnosis:**
```bash
# From another device on the network:
curl http://192.168.1.100:8000
ping 192.168.1.100
```

**Solutions:**

1. Find your machine's IP:
   ```bash
   hostname -I
   # or
   ip addr show
   ```

2. Configure firewall to allow ports:
   ```bash
   sudo ufw allow 8000
   sudo ufw allow 8888
   sudo ufw allow 5432
   sudo ufw status
   ```

3. Update `docker-compose.yml` to bind to all interfaces:
   ```yaml
   ports:
     - "0.0.0.0:8000:8000"
     - "0.0.0.0:8888:8888"
   ```

4. Restart containers:
   ```bash
   docker compose down
   docker compose up -d
   ```

---

### Issue 9: Device's IP address is not correct in NetBox

**Symptom:** Can't recollect correctly the information of the device

**Solutions:**

1. Find your device's IP:
   ```bash
   ip addr
   ```

2. Change device's IP address in NetBox (IPAM section)

3. Restart Oxidized container:
   ```bash
   docker compose restart oxidized
   ```

---

## General Troubleshooting Commands

```bash
# View all logs
docker compose logs

# Follow logs in real-time
docker compose logs -f

# View specific container logs
docker compose logs netbox
docker compose logs oxidized
docker compose logs postgres

# Check container status
docker compose ps

# Restart all containers
docker compose restart

# Stop all containers
docker compose stop

# Remove all containers (keeps volumes)
docker compose down

# Remove everything (WARNING: deletes all data)
docker compose down -v

# Access container shell
docker exec -it oxidized bash
docker exec -it netbox bash

# View container resource usage
docker stats

# Check network connectivity between containers
docker exec oxidized ping netbox
docker exec oxidized nslookup netbox
```

---

## When Nothing Works

If you've tried everything above:

1. Check system logs:
   ```bash
   journalctl -xe
   dmesg | tail -20
   ```

2. Nuclear option — rebuild everything (**⚠️ WARNING: deletes all data**):
   ```bash
   docker-compose down -v
   rm -rf volumes/
   docker system prune -a
   docker compose up -d
   ```

3. Verify Docker installation:
   ```bash
   docker --version
   docker compose --version
   docker run hello-world
   ```

4. Check GitHub issues:
   - NetBox: https://github.com/netbox-community/netbox/issues
   - Oxidized: https://github.com/ytti/oxidized/issues

5. Ask for help — include the output of `docker-compose ps` and `docker-compose logs`, describe what you've tried, and mention your OS and Docker version.

---

## Useful Debug Commands

```bash
# Export all device data
docker exec netbox python manage.py dumpdata dcim.Device > devices.json

# Test API connectivity
curl -v http://localhost:8000/api/dcim/devices/
curl -v http://localhost:8888/api/nodes

# Check file permissions
ls -la ~/.config/oxidized/

# Measure network latency
docker exec oxidized ping -c 4 8.8.8.8
```
