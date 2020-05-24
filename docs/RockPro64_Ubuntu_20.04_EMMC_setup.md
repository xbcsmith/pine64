# RockPro64 arm64 EMMC Setup

- [Start](#start)
- [Editor](#editor)
- [Sudo](#sudo)
- [Update](#update)
- [Update /etc/issue](#update-etcissue)
- [Gnome](#gnome)
- [Dev Tools](#dev-tools)
- [Build Tools](#build-tools)
- [GO](#go)
- [Rust](#rust)
- [Docker](#docker)
- [Python](#python)
- [NPM](#npm)
- [ATS](#ats)

## Start

grab the focal image for the rockpro64

```bash
curl -kLO https://github.com/ayufan-rock64/linux-build/releases/download/0.10.12/focal-gnome-rockpro64-0.10.12-1184-arm64.img.xz
```

Install to EMMC

```bash
xzcat focal-gnome-rockpro64-0.10.12-1184-arm64.img.xz | \
    sudo dd bs=4M of=/dev/mmcblk1 iflag=fullblock oflag=direct status=progress
```

## ATS

```bash
zegrep "PWM" /proc/config.gz
```

```bash
sudo apt remove lua5.1 liblua5.1-dev
sudo apt install lua5.3 liblua5.3-dev luarocks lua-sec
sudo update-alternatives --config lua-interpreter
sudo update-alternatives --config lua-compiler
sudo luarocks build https://raw.githubusercontent.com/tuxd3v/ats/master/ats-master-0.rockspec
```

Edit `/etc/ats.conf`

```bash
MAX_CONTINOUS_TERMAL_TEMP = 60
MIN_CONTINOUS_TERMAL_TEMP = 35
MAX_PWM = 255
MIN_PWM = 30
```

```bash
sudo systemctl restart ats
```
