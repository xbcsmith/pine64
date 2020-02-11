# RockPro64 arm64 EMMC Setup

## Start


grab the minimal bionic image for the rockpro64

```bash
curl -klO https://github.com/ayufan-rock64/linux-build/releases/download/0.9.16/bionic-minimal-rockpro64-0.9.16-1163-arm64.img.xz
```

Install to EMMC

```bash
xzcat bionic-minimal-rockpro64-0.9.16-1163-arm64.img.xz | sudo dd bs=4M of=/dev/mmcblk1 iflag=fullblock oflag=direct status=progress
```



## EDITOR

### Always vim

```
sudo update-alternatives --config editor
```

## Sudo

```bash
sudo visudo
```

```bash
# Allow member of wheel to do stupid things
%wheel  ALL=(ALL)       NOPASSWD: ALL
```

```bash
sudo groupadd wheel
```

```bash
sudo usermod -aG sudo $USER
sudo usermod -aG wheel $USER
```

```bash
sudo su -
export NEWUSER=<foo>
groupadd $NEWUSER
useradd -m -g $NEWUSER -G wheel -s /bin/bash -d /home/$NEWUSER $NEWUSER
passwd $NEWUSER
```

## Update

```bash
sudo mv /etc/apt/trusted.gpg.d/ayufan.key.chroot.gpg .
sudo apt update
sudo apt upgrade
```
 
## Install Vanilla Gnome 3

```bash
sudo apt update
sudo apt install vanilla-gnome-desktop
sudo apt install gnome-shell-extension-* nautilus
sudo update-alternatives --config gdm3.css
```

## GO

```
sudo add-apt-repository ppa:longsleep/golang-backports
sudo apt-get update
sudo apt-get install golang-go
```

## Rust

```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
```


## Dev tools

```bash
sudo apt install firefox terminator git
sudo apt install unzip wget libguestfs-tools virtualenvwrapper pkg-config zip
sudo apt install retext gedit
```

## Build Tools

```bash 
sudo add-apt-repository ppa:ubuntu-toolchain-r/test
sudo apt update
sudo apt install gcc-5 g++-5 build-essential git libsecret-1-dev fakeroot rpm libx11-dev libxkbfile-dev
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-5 80 --slave /usr/bin/g++ g++ /usr/bin/g++-5
sudo update-alternatives --config gcc # choose gcc-5 from the list
```

## Docker

```bash
export ARM="arm64"
sudo apt remove docker docker-engine docker.io
sudo apt install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository "deb [arch=${ARM}] https://download.docker.com/linux/ubuntu $(lsb_release -cs) edge"
sudo apt update
sudo apt install docker-ce
```

### Expose port to world... seems safe

Don't do this unless you understand the security risk

```bash
sudo su -
mkdir -p /etc/docker
cat > /etc/docker/daemon.json << EOF
{
"debug": true,
"hosts": ["tcp://0.0.0.0:2375", "unix:///var/run/docker.sock"]
}

EOF

sed -i 's~dockerd -H fd://~dockerd~g' /lib/systemd/system/docker.service
sed -i 's~StartLimitInterval=60s~StartLimitInterval=60s\nIPForward=yes\n~g' /lib/systemd/system/docker.service
```


```bash
sudo groupadd docker
sudo usermod -aG docker $USER
sudo systemctl daemon-reload
sudo systemctl enable docker.service
sudo systemctl start docker.service
```

```bash
docker run --rm -it hello-world
```



## Python

```bash
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt install python3.7 python3.8
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.6 1
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.7 2
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.8 3
sudo update-alternatives --set python3 /usr/bin/python3.6
```

### Fix apt-get update

```bash
sudo apt-get install --reinstall python3-apt
```

## NPM and Yarn

### NPM

```bash
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
curl -s https://deb.nodesource.com/gpgkey/nodesource.gpg.key | sudo apt-key add -
curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt install nodejs yarn
```


## Atom

### Not Working

https://flight-manual.atom.io/hacking-atom/sections/hacking-on-atom-core/#platform-linux

```bash
mkdir -p git/github.com/atom
git clone https://github.com/atom/atom.git
```


## Install ATS

```bash
zegrep "PWM" /proc/config.gz
```

```bash
sudo apt remove lua5.1 liblua5.1-dev
sudo apt install lua5.3-dev
sudo update-alternatives  --install /usr/bin/lua lua-interpreter /usr/bin/lua5.3 130
sudo update-alternatives --install /usr/bin/luac lua-compiler /usr/bin/luac5.3 130
sudo update-alternatives --config lua-interpreter
sudo update-alternatives --config lua-compiler
sudo luarocks build https://raw.githubusercontent.com/tuxd3v/ats/master/ats-master-0.rockspec
```

Edit `/etc/ats.conf`

MAX_CONTINOUS_TERMAL_TEMP = 60

MIN_CONTINOUS_TERMAL_TEMP = 35

MAX_PWM = 255

MIN_PWM = 30

ALWAYS_ON = true


```bash
sudo systemctl restart ats
```







