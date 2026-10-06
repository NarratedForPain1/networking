# Networking

## How to install Packet Tracer on Arch Linux

### 1. Install distrobox and docker

Both are available in the official Arch repos:

```bash
sudo pacman -S distrobox docker
sudo systemctl enable --now docker
```

### 2. Create the container environment

Spins up an Ubuntu container named `cisco-env`:

```bash
distrobox create --image ubuntu:latest --name cisco-env
```

As originally written in my notes: `distrobox create --image ubuntu: latest --name cisco-env`

