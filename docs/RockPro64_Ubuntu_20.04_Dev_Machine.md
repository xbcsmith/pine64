# Ubuntu 20.04 Dev Machine

- [Editor](#Editor)
- [Sudo](#sudo)
- [Update](#update)
- [Update /etc/issue](#update-etcissue)
- [Gnome](#gnome)
- [GO](#go)
- [Rust](#rust)
- [Docker](#docker)
- [Devel Pkgs](#devel-pkgs)
- [Python](#python)
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
sudo apt install nautilus gnome-tweaks
```

## GO

```bash
export GOVERSION=1.14.3
curl -kLO https://dl.google.com/go/go${GOVERSION}.linux-arm64.tar.gz
sudo rm -rfv /usr/local/go
sudo tar -C /usr/local/ -xvzf go${GOVERSION}.linux-arm64.tar.gz
export PATH=/usr/local/go/bin:$PATH
go version
rm -v go${GOVERSION}.linux-arm64.tar.gz

mkdir -p ~/go/{src,bin}

cat >> ~/.bashrc << EOF
# GO VARIABLES
export GOPATH=\$HOME/go
export PATH=\$PATH:/usr/local/go/bin:\$GOPATH/bin

EOF

go get github.com/oklog/ulid/cmd/ulid
go get -u golang.org/x/tools/...
go get -u golang.org/x/lint/golint
curl -sSfL https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh | sh -s -- -b $(go env GOPATH)/bin v1.27.0

## Most people don't need go-critic as it comes with golangci-lint
## I probably don't really need it locally either...
go get -v github.com/go-lintpack/lintpack/...
go get -v github.com/go-critic/go-critic/...
## For some reason with 1.14.3 install instructions didn't work
## So lets do this the hard way
mkdir -p ~/go/src/github.com/go-critic
cd ~/go/src/github.com/go-critic
git clone git@github.com:go-critic/go-critic.git
cd go-critic/
make gocritic
./gocritic check --help
mv gocritic ~/go/bin/
cd
```

## Rust

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
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

I use the RockPro64 as a remote docker host for arm64 containers
so it is convienent to be able to access it from my x86_64 box
without a direct ssh connection.

The following is not recomended.

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
sudo apt -y update && sudo apt -y install \
    build-essential \
    devscripts \
    fakeroot \
    debhelper \
    dpkg-dev \
    automake \
    autotools-dev \
    autoconf \
    libtool \
    systemtap-sdt-dev \
    libssl-dev \
    m4 \
    bison \
    flex \
    opensp \
    xsltproc \
    gettext \
    libyaml-dev \
    unzip \
    wget \
    pkg-config \
    libguestfs-tools \
    zip

```

## Python

```bash
sudo apt -y update && sudo apt -y install \
    libpython3-dev \
    python3-dev \
    virtualenvwrapper \
    virtualenv \
    python3-mock \
    python3-venv \
    flake8 \
    isort \
    black \
    tox


```

### Virtual Env

```bash
mkdir -p ~/.virtualenvs
PYTHON3=$(which python3.8)
$PYTHON3 -m venv ~/.virtualenvs/foo
source ~/.virtualenvs/foo/bin/activate

pip install --upgrade pip setuptools pbr wheel pip pkg_resources docker
pip install --upgrade rfc3987 enum34 PyYAML stevedore jsonschema Jinja2
pip install --upgrade autopep8 flake8 tox black isort pdbpp
```

## NPM

```bash
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
curl -s https://deb.nodesource.com/gpgkey/nodesource.gpg.key | sudo apt-key add -
curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt install nodejs yarn
```
