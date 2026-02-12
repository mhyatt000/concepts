# macOS Colima Setup (Intel Mac)

Colima is a free alternative to Docker Desktop that runs Docker containers via QEMU on macOS.

## Prerequisites

[Homebrew](https://brew.sh) must be installed.

## Install

```sh
brew install colima docker
```

## Start and enable autostart on login

```sh
brew services start colima
```

This registers a launchd plist at `~/Library/LaunchAgents/homebrew.mxcl.colima.plist` so Colima starts automatically every login. It also starts Colima immediately.

## Verify

```sh
colima status
docker ps
```

## Useful commands

```sh
brew services stop colima    # stop and disable autostart
brew services restart colima # restart
colima status                # show VM info and socket paths
docker context show          # should print "colima"
tail -f /usr/local/var/log/colima.log  # view service logs
```

## Notes

- Runtime: Docker (default)
- VM: QEMU x86_64 on Intel Macs
- Docker socket: `~/.colima/default/docker.sock`
- Docker context is automatically set to `colima` on first start
- No Docker Desktop required
