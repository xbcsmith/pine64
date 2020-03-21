# Ubuntu 18.04 Dev Machine

- [Editor](#Editor)
- [Sudo](#sudo)
- [Update](#update)
- [Update /etc/issue](#update-etcissue)
- [Gnome](#gnome)
- [GO](#go)
- [Rust](#rust)
- [Docker](#docker)
- [Devel Pkgs](#devel-pkgs)
- [Virtual Env](#virtual-env)

## Editor

_Always **vim**_

```bash
sudo update-alternatives --config editor
```

## Sudo

```bash
sudo visudo
```

```bash
%wheel  ALL=(ALL)       NOPASSWD: ALL # Allow member of wheel to do stupid things
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

## Update /etc/issue

Display the IP address

```bash
echo -en "IP: \4\n" >> /etc/issue
```

## Gnome

```bash
sudo apt install vanilla-gnome-desktop
sudo apt install gnome-shell-extension-* nautilus
sudo update-alternatives --config gdm3.css
```

## GO

```bash
sudo add-apt-repository ppa:longsleep/golang-backports
sudo apt update
sudo apt install golang-go
```

## Rust

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
```

## Docker

```bash
export ARM="armhf" ## If you are headless and using aarch64 set this to "arm64"
sudo apt remove docker docker-engine docker.io
sudo apt install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository "deb [arch=${ARM}] https://download.docker.com/linux/ubuntu $(lsb_release -cs) edge"
sudo apt update
sudo apt install docker-ce
```

### Expose port to world... seems safe

```bash
sudo su -
mkdir /etc/docker
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
sudo systemctl enable docker
sudo systemctl start docker
```

```bash
docker run --rm -it hello-world
```

## Devel Pkgs

```bash
sudo apt -y update && sudo apt -y install build-essential devscripts fakeroot \
  debhelper dpkg-dev automake autotools-dev autoconf libtool perl libperl-dev \
  systemtap-sdt-dev libssl-dev python-dev python3-dev m4 bison flex opensp \
  xsltproc gettext unzip wget libguestfs-tools virtualenvwrapper tox \
  python3-virtualenv openjdk-8-jre-headless openjdk-8-jdk-headless pkg-config \
  python-logilab-common python-unittest2 python-mock zip python-autopep8 \
  python3-flake8 flake8 python-flake8 isort python-isort python3-isort \
  vim-autopep8 python-wheel python3-wheel python-pip python3-pip tox
```

```bash
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update
sudo apt install python3.7 python3.7-venv
```

## Virtual Env

```bash
mkdir -p ~/.virtualenvs
PYTHON3=$(which python3.7)
$PYTHON3 -m venv ~/.virtualenvs/foo
source ~/.virtualenvs/foo/bin/activate

pip install --upgrade pip setuptools pbr wheel pip pkg_resources functools32 docker
pip install --upgrade rfc3987 enum34 PyYAML stevedore jsonschema Jinja2
pip install --upgrade autopep8 flake8 tox black isort pdbpp
```
