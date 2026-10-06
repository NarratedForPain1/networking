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

### 8. Troubleshooting

#### Clicking Packet Tracer crashes immediately (SIGSEGV in `ld-linux`)

The app dies within a second of launching. `coredumpctl` shows the process faulting in the dynamic loader (`ld-linux-x86-64.so.2`) while the AppImage runtime starts up, and the core is tiny (~30 kB) because nothing of the app has loaded yet.

**Root cause:** the Cisco `.AppImage` needs FUSE to mount itself, but the base Ubuntu container ships no `fusermount`. Trying to run it reports:

```
fuse: failed to exec fusermount: No such file or directory
open dir error: No such file or directory
```

Instead of exiting cleanly, the runtime's failure path segfaults inside the loader.

**Fix:** don't use FUSE at all. Run the AppImage in extract-and-run mode by editing the exported launcher (e.g. `~/.local/share/applications/cisco-env-CiscoPacketTracer-9.0.1.desktop`) and setting `APPIMAGE_EXTRACT_AND_RUN=1` on its `Exec` line:

```ini
Exec=/usr/bin/distrobox-enter -n cisco-env -- env APPIMAGE_EXTRACT_AND_RUN=1 /opt/pt/packettracer.AppImage %f
```

`APPIMAGE_EXTRACT_AND_RUN=1` unpacks the AppImage into a temp directory and runs it from there, bypassing FUSE completely. (Alternative: `sudo apt install fuse3` inside the container.)

#### Missing shared libraries once the mount is fixed

A minimal Ubuntu base also lacks the GUI libraries Packet Tracer expects from the system, producing errors like:

```
./PacketTracer: error while loading shared libraries: libOpenGL.so.0: cannot open shared object file
./PacketTracer: error while loading shared libraries: libnss3.so: cannot open shared object file
```

plus `gsettings: command not found`. Install them as root inside the container:

```bash
docker exec -u root cisco-env apt install -y \
  libopengl0 libglib2.0-bin libnss3 libnspr4 \
  libgl1 libgl1-mesa-dri libasound2t64 libpulse0 \
  libxcomposite1 libxdamage1 libxrandr2 libxss1 libxtst6 \
  libxi6 libxrender1 libx11-xcb1 libxkbcommon0 libxkbcommon-x11-0 \
  libgtk-3-0 libgbm1 libcurl4 fontconfig fonts-liberation
```

Use `docker exec -u root` for this rather than `distrobox enter --root` — the latter prompts for a password over sudo and hangs in a non-interactive session. Note `libasound2t64` (the `t64` suffix is current Ubuntu ABI naming; on older releases it's `libasound2`).

#### Harmless remaining messages

Log lines like `Failed to connect to /run/dbus/system_bus_socket: No such file or directory` are expected in a container without a system D-Bus. In-container notifications won't work, but Packet Tracer itself runs fine.

