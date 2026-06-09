# HOW TO USE THIS SYSTEM

## Step 1: Start the Containers
docker compose up -d

### Verify all containers are running:
docker compose ps

## Step 2: Create a NetBox Superuser Account
1. Open http://localhost:8000 in your browser
2. Go to the NetBox container and create a superuser with the following command:
    python netbox/manage.py createsuperuser
3. Enter username, email and password
4. Save your credentials
5. Now, you should be able to create devices, ip addresses, new users, etc

## Step 3: Create Additional Users (Optional)

To add more users:

1. Click Admin (top right) → Users
2. Click "Add User"
3. Fill in username, email, password
4. Save

## Step 4: Add Devices in NetBox
1. Go to NetBox → Devices → Add Device
2. Fill in the following fields:
    - Device Name (e.g., "switch-01")
    - Device Type (e.g., "Cisco IOS")
    - Platform/OS slug (must match Oxidized model configuration)
3. Click Save

Example device configuration:
- Code
Name: switch-01
Type: Cisco IOS
Slug: cisco-ios
IP Address: 192.168.1.100

## Step 5: Oxidized is Already Configured

The Oxidized configuration is already set up and working correctly.

It will automatically:
- Read devices from NetBox API
- Connect via SSH to each device
- Save backups every 1 hour
- Display status in the web interface

**No configuration needed** unless you want to customize timeouts or add new device types.

To view the current configuration:
- bash
docker exec oxidized cat /home/oxidized/.config/oxidized/config

## Step 6: Verify Backups Are Running
1. Open http://localhost:8888 (Oxidized web interface)
2. Check the "Last Status" section

Expected results:
- ✅ Green square = Backup completed successfully
- ❌ Red square = Connection failed (check credentials/connectivity)

## Troubleshooting
### Issue: Oxidized Web Interface Not Loading
If the Oxidized web interface is not accessible:

- bash
docker exec -it oxidized rm /home/oxidized/.config/oxidized/oxidized.pid
docker compose restart oxidized

Wait 30 seconds and refresh your browser.

### Issue: Device Shows Red Status
1. Verify the device is reachable:
- bash
ping <device-ip>
2. Verify SSH credentials are correct in NetBox
3. Check Oxidized logs:
- bash
docker compose logs -f oxidized

### Check Backup Files
View all saved backups:

- bash
ls -la /path/to/backups/

Each device should have a configuration file. Example:

- Code
switch-01.conf
router-01.conf
firewall-01.conf

## Expected Results
- ✅ NetBox displays your inventory of devices
- ✅ Oxidized shows green squares for each device in "Last Status"
- ✅ Backup files exist in your backup folder
- ✅ Configurations update every 1 hour (default interval)
- ✅ You can view device configurations in Oxidized web interface

## Summary
1. Containers running → ✅
2. NetBox account created → ✅
3. Devices registered in NetBox → ✅
4. Oxidized models configured → ✅
5. Backups showing green status → ✅

System is working correctly!