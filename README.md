# Siemer CNC

Turn a Raspberry Pi into a dedicated CNC controller kiosk with [CNCjs](https://github.com/cncjs/cncjs).

![CNCjs](cncjs/media/banner.png)

## Features

- **CNCjs** — Full-featured web interface for Grbl, Marlin, Smoothieware, and TinyG
- **TinyWeb Pendant** — Lightweight touch-friendly control interface
- **Kiosk Mode** — Auto-starts Chromium in fullscreen on boot
- **Custom Screensaver** — Custom CNC sleeping screensaver after 5 minutes of inactivity
- **PM2 Process Manager** — Keeps CNCjs running reliably

## Quick Start

### Prerequisites

- Raspberry Pi (3B+ or newer recommended)
- Raspberry Pi OS with Desktop
- Ansible installed on your local machine

### Installation

```bash
# Clone this repo
git clone --recursive https://github.com/andrewsiemer/siemer-cnc.git
cd siemer-cnc

# Configure your Pi's hostname/IP in the inventory
nano ansible/inventory.ini

# Run the Ansible playbook
cd ansible
ansible-playbook install.yml

# Reboot the Pi
ansible pi_cnc -m reboot -b
```

See [ansible/README.md](ansible/README.md) for detailed setup instructions.

## Project Structure

```
siemer-cnc/
├── ansible/          # Automated Pi setup playbook
├── cncjs/            # CNCjs submodule
├── tinyweb/          # TinyWeb pendant interface
├── kiosk/            # Chromium kiosk scripts
└── screensaver/      # Custom screensaver video
```

## Usage

After setup, the Pi will:

1. **Auto-login** to the desktop
2. **Start CNCjs** via PM2 on port 8000
3. **Launch Chromium** in kiosk mode showing the TinyWeb interface
4. **Play screensaver** video after 5 minutes of inactivity

### Access Points

| Interface       | URL                                 |
| --------------- | ----------------------------------- |
| CNCjs Main      | `http://<pi-hostname>:8000`         |
| TinyWeb Pendant | `http://<pi-hostname>:8000/tinyweb` |

### G-Code Files

Drop G-code files into `/home/pi/watch` — they'll appear automatically in CNCjs.

## Manual Commands

SSH into the Pi to manage CNCjs:

```bash
# Check status
pm2 status

# View logs
pm2 logs cncjs

# Restart
pm2 restart cncjs
```

## License

MIT

