# WinVM-Universal


A shell script for managing Windows VMs using Docker/Podman and the [dockurr/windows](https://github.com/dockurr/windows) image, built on top of [Omarchy's Windows VM script](https://github.com/basecamp/omarchy/blob/master/bin/omarchy-windows-vm). 

## Features

- **Cross-Distribution Support** - Works on Arch and probably Debian/Ubuntu, Fedora, openSUSE, Void, Alpine, and more
- **Docker & Podman** - Automatically detects available container engine
- **Docker Compose v1 & v2** - Supports both standalone and plugin versions
- **Multiple Instances** - Run multiple independent Windows VMs with separate configs
- **Multiple Desktop Environments** - Auto-detects display scaling for Niri, Hyprland, Sway, GNOME, KDE
- **RDP Client Choice** - Choose between FreeRDP or Remmina (or both)
- **Backup & Restore** - Compressed backups using zstd
- **Shared Folder** - Easy file sharing between host and Windows VM
- **No External TUI Dependencies** - Pure bash UI, no gum/dialog required

## Requirements

These should all be caught and installed automatically, but if not:
- **Container Engine**: Docker or Podman
- **Virtualization**: KVM support (`/dev/kvm`)
- **RDP Client**: FreeRDP (`xfreerdp3`, `xfreerdp`, or `freerdp`) and/or Remmina
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
- Prompt for container name and storage locations
- Prompt for VM configuration (RAM, CPU, disk size)
- Let you choose your preferred RDP client (FreeRDP, Remmina, or both)
- Set up Windows credentials
- Download and start the Windows image

## Usage

```
winvm [--instance NAME] [command] [options]

Global Options:
  --instance, -i NAME  Use a specific VM instance (default: "default")

Commands:
  install              Install and configure Windows VM
  remove               Remove Windows VM and optionally its data
  config               Configure directories and RDP client settings
  start                Start Windows VM without connecting via RDP
  launch [options]     Start Windows VM (if needed) and connect via RDP
                       Options:
                         --kill, -k         Stop VM when RDP closes
                         --remmina          Use Remmina for this session
                         --freerdp          Use FreeRDP for this session
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

# Connect to VM (keeps running after RDP closes)
winvm launch

# Connect and stop VM when RDP closes
winvm launch -k

# Force use of Remmina for this session
winvm launch --remmina

# Force use of FreeRDP and stop on disconnect
winvm launch --freerdp -k

# Check VM status
winvm status

# Change RDP client or directory settings
winvm config

# Stop the VM
winvm stop

# Create a backup
winvm backup

# Restore from a backup
winvm restore

# Remove VM completely
winvm remove
```

### Multiple Instances

Run multiple independent Windows VMs, granted, not simultaneously because of port things, by using the `--instance` (or `-i`) flag:

```bash
# Create a "work" instance
winvm -i work install

# Create a "gaming" instance
winvm -i gaming install

# Launch specific instances
winvm -i work launch
winvm -i gaming launch --remmina

# Configure a specific instance
winvm -i work config

# Each instance has completely separate:
# - Container name
# - Storage directories
# - Configuration
# - RDP settings
```

## Configuration

### Config Command

Use `winvm config` to view and modify settings without reinstalling:

```bash
winvm config
```

This allows you to change:
- **Container name** - Docker/Podman container identifier
- **Compose directory** - Location of docker-compose.yml
- **Storage directory** - Windows virtual disk location
- **Shared directory** - Host-guest shared folder
- **RDP client** - FreeRDP, Remmina, or both (with default selection)

### Config File

Configuration is stored in `~/.config/winvm-uni/<instance>/paths.conf`:

| Setting            | Default                                        | Description                        |
| ------------------ | ---------------------------------------------- | ---------------------------------- |
| `CONTAINER_NAME`   | `<instance>`                                   | Docker container name              |
| `COMPOSE_DIR`      | `~/.local/share/docker/winvm-uni/<instance>`   | Docker compose file location       |
| `STORAGE_DIR`      | `~/.winvm-uni/<instance>`                      | Windows virtual disk storage       |
| `SHARED_DIR`       | `~/windows-uni/<instance>`                     | Shared folder between host/guest   |
| `RDP_CLIENT_TYPE`  | `freerdp`                                      | `freerdp`, `remmina`, or `both`    |
| `RDP_DEFAULT`      | `freerdp`                                      | Default client when both installed |

### Pointing to Existing Setup

If you have an existing Windows VM setup, use `winvm config` to point to your existing directories without reinstalling:

```bash
winvm config
# Select "Directories"
# Enter your existing paths
```

## RDP Clients

### FreeRDP
- Traditional command-line RDP client
- Supports automatic display scaling detection
- Uses flags: `/dynamic-resolution`, `/gfx:AVC444`, etc.

### Remmina
- GTK-based graphical RDP client
- Better compatibility with some Wayland compositors (e.g., Niri). At least, I've had more success with it than  `xfreerdp3`
- Generates a `.remmina` connection file automatically

### Choosing at Install
During installation, you'll be prompted:
```
Select RDP client to install:
  1) FreeRDP
  2) Remmina
  3) Both
```

If you select "Both", you'll also choose a default. You can override the default per-session using `--remmina` or `--freerdp` flags.

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

## Supported Desktop Environments

Display scaling auto-detection might work with:
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

### Remmina not connecting
- Try using the `--freerdp` flag to test with FreeRDP
- Check if the `.remmina` file was generated: `ls ~/.local/share/docker/winvm-uni/<instance>/`
- Ensure Remmina has the RDP plugin installed

## Credits

- Built on top of [Omarchy's Windows VM script](https://github.com/basecamp/omarchy/blob/master/bin/omarchy-windows-vm)
- Uses [dockurr/windows](https://github.com/dockurr/windows) Docker image
- Stormtrooper ASCII art from [emojicombos.com](https://emojicombos.com/storm-trooper-ascii-art)

## License

MIT
