# Siemer CNC Ansible Playbook

Automated setup for Raspberry Pi as a CNC controller kiosk using CNCjs.

## What This Does

This Ansible playbook transforms a fresh Raspberry Pi OS installation into a fully configured CNC controller with:

- **CNCjs**: Web-based CNC controller interface
- **TinyWeb**: Lightweight pendant interface
- **PM2**: Process manager to keep CNCjs running
- **Chromium Kiosk**: Auto-starts fullscreen browser on boot
- **Custom Screensaver**: Plays `screensaver.m4v` after 5 minutes of inactivity

## Roles

The playbook is organized into isolated roles:

| Role          | Description                                                     |
| ------------- | --------------------------------------------------------------- |
| `common`      | System updates, base packages, user permissions, display config |
| `nodejs`      | Node.js, Yarn, and PM2 installation                             |
| `cncjs`       | Clone repo, build CNCjs, configure PM2                          |
| `screensaver` | xscreensaver setup with custom video playback                   |
| `kiosk`       | Chromium kiosk mode and autostart configuration                 |
| `update`      | Pull latest changes, update submodules, rebuild and redeploy    |

## Prerequisites

### On Your Mac/Linux Machine

```bash
# Install Ansible
pip3 install ansible

# Or on macOS with Homebrew
brew install ansible
```

### On the Raspberry Pi

1. Flash **Raspberry Pi OS (with Desktop)** to your SD card using [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
2. During setup, configure:
   - Username: `pi`
   - Enable SSH
   - Configure WiFi (if not using ethernet)
3. Boot the Pi and ensure it's on your network

## Quick Start

### 1. Configure Inventory

Edit `inventory.ini` with your Pi's IP address or hostname:

```ini
[pi_cnc]
pi ansible_host=cnc.local ansible_user=pi

[pi_cnc:vars]
ansible_ssh_private_key_file=~/.ssh/rpi
```

### 2. Test Connection

```bash
cd ansible
ansible pi_cnc -m ping
```

### 3. Run the Playbook

```bash
ansible-playbook install.yml
```

Run individual roles:
```bash
# Only run nodejs setup
ansible-playbook install.yml --tags nodejs

# Skip package upgrades (faster)
ansible-playbook install.yml -e "upgrade_packages=false"
```

### 4. Update & Redeploy

To pull the latest changes and rebuild CNCjs without running the full setup:

```bash
ansible-playbook update.yml
```

This will:
- Pull latest changes from the git repository
- Update submodules with `--remote` to get latest upstream changes
- Rebuild CNCjs for production
- Restart CNCjs via PM2

### 5. Reboot the Pi

```bash
ansible pi_cnc -m reboot -b
```

## Configuration

Edit `group_vars/all.yml` to customize:

```yaml
# CNCjs port (default: 8000)
cncjs_port: 8000

# Screensaver timeout in minutes
screensaver_timeout_minutes: 5

# Display resolution (for video positioning)
display_width: 800
display_height: 480
```

## File Structure

```
ansible/
├── ansible.cfg                 # Ansible configuration
├── inventory.ini               # Pi host configuration
├── install.yml                 # Main playbook (full setup)
├── update.yml                  # Update & redeploy playbook
├── group_vars/
│   └── all.yml                 # Global variables
└── roles/
    ├── common/
    │   └── tasks/main.yml      # System setup tasks
    ├── nodejs/
    │   └── tasks/main.yml      # Node.js, Yarn, PM2
    ├── cncjs/
    │   ├── tasks/main.yml      # CNCjs build & PM2 config
    │   └── templates/
    │       └── ecosystem.config.js.j2
    ├── screensaver/
    │   ├── tasks/main.yml      # Screensaver setup
    │   └── templates/
    │       ├── xscreensaver.j2
    │       ├── screensaver-watcher.sh.j2
    │       └── screensaver.service.j2
    ├── kiosk/
    │   ├── tasks/main.yml      # Chromium kiosk setup
    │   └── templates/
    │       ├── kiosk.sh.j2
    │       ├── autostart.j2
    │       └── kiosk.service.j2
    └── update/
        └── tasks/main.yml      # Git pull, rebuild, redeploy
```

## Usage After Setup

### CNCjs Web Interface

- Main interface: `http://cnc.local:8000`
- TinyWeb pendant: `http://cnc.local:8000/tinyweb`

### PM2 Commands

SSH into the Pi and use:

```bash
# Check status
pm2 status

# View logs
pm2 logs cncjs

# Restart CNCjs
pm2 restart cncjs

# Stop CNCjs
pm2 stop cncjs
```

### Screensaver

- Activates after 5 minutes of inactivity
- Plays `screensaver/screensaver.m4v` on loop
- Move mouse or touch screen to exit

### Adding G-Code Files

Place files in `/home/pi/watch` - they'll appear in CNCjs automatically.

## Troubleshooting

### Can't Connect to Pi

```bash
# Check if Pi is reachable
ping cnc.local

# Try with IP address directly
ansible pi_cnc -m ping -e "ansible_host=192.168.1.XXX"
```

### CNCjs Not Starting

```bash
ssh pi@cnc.local
pm2 logs cncjs
pm2 restart cncjs
```

### Run a Single Role

```bash
# Re-run just the kiosk setup
ansible-playbook install.yml --tags kiosk

# Re-run CNCjs build
ansible-playbook install.yml --tags cncjs
```

### Screensaver Not Working

```bash
# Check xscreensaver is running
pgrep xscreensaver

# Check the watcher service
sudo systemctl status screensaver-watcher
```

## License

MIT
