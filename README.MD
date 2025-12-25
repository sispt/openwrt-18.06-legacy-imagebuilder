# OpenWrt 18.06 Legacy Image Builder

This repository documents how to build custom OpenWrt firmware for obsolete routers
(4MB flash, 32MB RAM) that are no longer supported by modern OpenWrt releases.

Examples:
- TP-Link TL-WR841 v11
- ar71xx / tiny target

These devices still work electrically, but cannot run OpenWrt 19+.
This project shows how to build optimized firmware for them using Docker and ImageBuilder.

---

## What these instructions do

They create a custom firmware image that includes:

- WiFi
- Firewall and NAT
- iptables
- iptables-mod-ipopt 
[optional. I use it to alter mangle prerouting TTL]
- wget
- BusyBox

And removes:

- PPP
- PPPoE
[also optional. it depends on what you're gonna use this image for]
to fit inside 4MB flash.

---

## Why Docker is used

OpenWrt 18.06 uses very old build tools (Python 2, old GCC, old ncurses).
Modern Linux distributions no longer provide them.

Docker allows running an old Ubuntu (18.04) environment safely without polluting your host system.

Your host OS can be Fedora, Arch, Ubuntu, or anything.

---

# Docker install

Fedora:
`sudo dnf install docker`

`sudo systemctl enable --now docker`

Ubuntu:

`sudo apt install docker.io`

`sudo systemctl start docker`

Start a container with your home directory mounted:

`docker run -it --rm -v $HOME:/work ubuntu:18.04`

This makes /work inside the container point to your real $HOME folder.

Anything copied into /work is saved outside the container.

# Building an image inside the Docker container

Inside the Ubuntu 18.04 container:

`apt update`

`apt install -y build-essential gawk unzip wget rsync file`

find the correct link for the ImageBuilder (example: ar71xx tiny):

`cd /root`

`wget https://downloads.openwrt.org/releases/18.06.9/targets/ar71xx/tiny/openwrt-imagebuilder-18.06.9-ar71xx-tiny.Linux-x86_64.tar.xz`

[The link depends on your router's model. DON'T USE THIS LINK IF NOT USING TL-WR841N-V11]


`tar xf openwrt-imagebuilder-18.06.9-ar71xx-tiny.Linux-x86_64.tar.xz`

`cd openwrt-imagebuilder-18.06.9-ar71xx-tiny.Linux-x86_64`

## Build firmware:

`make image PACKAGES="-ppp -ppp-mod-pppoe iptables iptables-mod-ipopt wget"`

`` add - before a package's name to exclude it `` 

## Firmware output

After building, firmware files appear in:

bin/targets/<target>/<subtarget>/

Example:

bin/targets/ar71xx/tiny/


Files:

- `*-factory.bin` → flash from stock firmware
- `*-sysupgrade.bin` → upgrade from existing OpenWrt

Now you can flash one of these images to your router.
