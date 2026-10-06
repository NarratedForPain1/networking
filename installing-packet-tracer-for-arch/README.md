# Networking

## How to install Packet Tracer on Arch Linux

### 1. Install distrobox and docker

Both are available in the official Arch repos:

```bash
sudo pacman -S distrobox docker
sudo systemctl enable --now docker
```

### 2. Create the container environment

```bash
distrobox-create --root --image docker.io/library/ubuntu:latest --name cisco-env -Y
```

Essentially this builds a disposable Ubuntu container called `cisco-env` that Packet Tracer will be installed into:

- `distrobox-create` — creates the container and wires it to your existing Arch system
- `--root` — runs the container as root (rootful) instead of rootless
- `--image docker.io/library/ubuntu:latest` — pulls the latest official Ubuntu image from Docker Hub
- `--name cisco-env` — gives the container the name `cisco-env`
- `-Y` — answers "yes" to every prompt so it runs without waiting on you

