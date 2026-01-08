# WinVM-Universal

```
  ⠀⠀⠀⠀⠀⠀⣀⠤⣒⣊⣭⠭⠉⠉⠑⠒⠢⠤⣀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⢠⢊⡵⣪⡿⠋⠀⠀⠀⠀⠀⠀⠀⠁⠪⣝⢦⠀⠀⠀⠀⠀
⠀⠀⠀⢠⠃⣾⣿⠏⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢹⡄⢣⠀⠀⠀⠀
⠀⠀⠀⣾⠰⣿⣿⢀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸⡇⢸⡄⠀⠀⠀
⠀⠀⢀⣿⢠⣿⣿⣸⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢿⡇⣼⣇⠀⠀⠀
⠀⠀⢿⣿⡿⣿⣿⣿⣷⣶⣶⣶⣶⣶⣆⣀⣀⣀⣹⣿⡿⣿⢿⠀⠀⠀
⠀⢀⣯⣿⣧⣿⣿⣿⣿⠿⣋⣥⠖⠢⢌⠙⢿⣶⣼⣿⢁⢸⣾⡧⠀⠀
⠀⠈⢿⢹⣶⣉⡉⠩⠰⣾⠏⠀⠀⠀⠀⠈⠂⠌⢉⣡⣾⢠⢽⣧⣄⠀
⠀⢀⠞⠹⣿⠃⠀⠀⣸⠇⠀⢀⣤⣤⣀⠀⠀⠀⠀⠘⣿⠀⠢⡙⢿⣧
⣰⣃⠤⣿⣌⢀⠀⠀⢸⣤⣾⠿⠿⠿⠿⣿⣦⡀⠀⢀⠏⢁⡴⠊⡂⣿
⣿⡇⠐⢝⠿⣿⡯⠲⢿⣿⣿⡏⠀⠀⠀⠀⠉⠛⠂⠌⠰⡿⠁⣠⣿⠇
⠹⣿⣤⣶⡏⠀⠈⠐⡀⢻⣿⠤⠤⠀⡀⠀⠀⡠⠀⠀⠀⢐⣴⡿⠋⠀
⠀⠹⣿⠻⣇⠠⢂⣐⡳⠤⢣⢰⣾⣦⢸⣖⠄⣐⣀⡒⢄⢠⣸⠁⠀⠀
⠀⠀⠈⢶⣄⠄⢯⣈⣻⡇⠸⣿⣿⣿⡷⠀⣻⣉⠿⠿⢘⣨⠇⠀⠀⠀
⠀⠀⠀⠀⠑⠠⢀⣀⣀⠤⠞⠉⠉⠙⠓⠤⣦⣀⣀⠤⠞⠁⠀⠀⠀⠀
```

A cross-platform (hopefully) shell script for managing Windows VMs using Docker/Podman and the [dockurr/windows](https://github.com/dockurr/windows) image and built on top of [Omarchy's Windows VM script](https://github.com/basecamp/omarchy/blob/master/bin/omarchy-windows-vm).

## Features

- **Cross-Distribution Support** - Works on Arch, might work on Debian/Ubuntu, Fedora, openSUSE, Void, Alpine, and maybe more
- **Docker & Podman** - Automatically detects available container engine
- **Docker Compose v1 & v2** - Could support both standalone and plugin versions
- **Multiple Desktop Environments** - Auto-detects display scaling for Niri, Hyprland, Sway, GNOME, KDE
- **RDP Connection** - Connects via FreeRDP with automatic client detection
- **Backup & Restore** - Compressed backups using zstd
- **Shared Folder** - Easy file sharing between host and Windows VM
- **No External TUI Dependencies** - Pure bash UI, no gum/dialog required

## Requirements
- These should all be caught and provided for in the install script, but if not:
- **Container Engine**: Docker or Podman
- **Virtualization**: KVM support (`/dev/kvm`)
- **RDP Client**: FreeRDP (`xfreerdp3`, `xfreerdp`, or `freerdp`)
- **Utilities**: `jq`, `zstd`, `bc`, `netcat`
## Installation

1. Download the script:
```bash
curl -o ~/.local/bin/winvm https://raw.githubusercontent.com/f4des/winvm/master/winvm-uni
chmod +x ~/.local/bin/winvm
```

2. Run the installer:
```bash
winvm install
```

The installer will:
- Detect your package manager and install dependencies
- Prompt for VM configuration (RAM, CPU, disk size)
- Set up Windows credentials
- Download and start the Windows image

## Usage

```
winvm [command] [options]

Commands:
  install              Install and configure Windows VM
  remove               Remove Windows VM and optionally its data
  launch [options]     Start Windows VM and connect via RDP
                       Options:
                         --keep-alive, -k   Keep VM running after RDP closes
  stop                 Stop the running Windows VM
  status               Show current VM status
  backup               Backup Windows VM to compressed archive
  restore              Restore Windows VM from backup
  help                 Show help message
```

### Examples

```bash
# First-time setup
winvm install

# Connect to VM (auto-stops when you close RDP)
winvm launch

# Connect to VM (keeps running after RDP closes)
winvm launch -k

# Check VM status
winvm status

# Stop the VM
winvm stop

# Create a backup
winvm backup

# Restore from a backup
winvm backup

# Remove VM completely
winvm remove
```

## Configuration

Configuration is stored in `~/.config/winvm-uni/paths.conf`:

| Setting       | Default                           | Description                      |
| ------------- | --------------------------------- | -------------------------------- |
| `COMPOSE_DIR` | `~/.local/share/docker/winvm-uni` | Docker compose file location     |
| `STORAGE_DIR` | `~/.winvm-uni`                    | Windows virtual disk storage     |
| `SHARED_DIR`  | `~/Windows-uni`                   | Shared folder between host/guest |

## Supported Distributions

| Distribution         | Package Manager | Tested |
| -------------------- | --------------- | ------ |
| CachyOS / Arch Linux | pacman/yay/paru | ✅      |
| Debian/Ubuntu        | apt             | ❌      |
| Fedora/RHEL          | dnf             | ❌      |
| openSUSE             | zypper          | ❌      |
| Void Linux           | xbps            | ❌      |
| Alpine               | apk             | ❌      |
| Gentoo               | emerge          | ❌      |
| NixOS                | nix-env         | ❌      |

## Likely Supported Desktop Environments

Display scaling auto-detection works with:
- Niri
- Hyprland
- Sway
- GNOME
- KDE Plasma
- X11 (via `GDK_SCALE` / `QT_SCALE_FACTOR`)

## Ports

| Port | Service               |
| ---- | --------------------- |
| 8006 | Web-based VNC console |
| 3389 | RDP (TCP/UDP)         |

## Troubleshooting

### KVM not available
```bash
# Check if KVM is loaded
lsmod | grep kvm

# Load KVM module (Intel)
sudo modprobe kvm-intel

# Load KVM module (AMD)
sudo modprobe kvm-amd

# Add yourself to kvm group
sudo usermod -aG kvm $USER
```

### Docker permission denied
```bash
# Add yourself to docker group
sudo usermod -aG docker $USER
# Log out and back in
```

### RDP connection fails
- Wait for Windows installation to complete (monitor at http://127.0.0.1:8006)
- Check if VM is running: `winvm status`
- Ensure port 3389 is not in use: `ss -tlnp | grep 3389`

## Credits

- Built on top of [Omarchy's Windows VM script](https://github.com/basecamp/omarchy/blob/master/bin/omarchy-windows-vm)
- Uses [dockurr/windows](https://github.com/dockurr/windows) Docker image
- Stormtrooper ASCII art from [emojicombos.com](https://emojicombos.com/storm-trooper-ascii-art)

## License

MIT
