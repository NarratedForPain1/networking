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

### 3. Enter the container

```bash
distrobox enter --root cisco-env
```

Drops you into a shell inside the `cisco-env` container. Everything from here on runs in Ubuntu, not Arch:

- `distrobox enter` — opens a shell inside an existing distrobox container
- `--root` — enters as root (same as how the container was created)
- `cisco-env` — the container to enter

Your home directory is shared with Arch, so files stay visible from both sides.

### 4. Update Ubuntu

```bash
sudo apt update
```

Refreshes Ubuntu's package lists so `apt` knows about the latest available versions before anything gets installed.

### 5. Download Packet Tracer

Downloaded the Linux installer (`.deb`) from the Cisco NetAcad site ([netacad.com](https://www.netacad.com)) — you need a (free) NetAcad account to grab it.

### 6. Install Packet Tracer inside the container

Inside `cisco-env`, navigated to wherever the installer was saved (mine was in `/mnt/app`):

```bash
cd /mnt/app
sudo apt install ./CiscoPacketTracer.deb
```

- `./` tells apt the file is right here in the current folder, not something to search for online
- `apt install` (rather than `dpkg`) automatically pulls in any dependencies Packet Tracer needs

### 7. Export the app to the host

```bash
distrobox-export --app packetracer
```

Registers Packet Tracer with your Arch application launcher so you can open it like any native app:

- `distrobox-export` — copies an app out of the container for the host to use
- `--app packetracer` — the app/command to export

This drops a `.desktop` launcher on the host that silently starts the app inside `cisco-env` for you.

