# Rock64 K3S Nodes

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

## Hack the MAC

```bash
sudo apt install net-tools
ifconfig | grep ether
sudo apt install macchanger
sudo ifdown eth0
sudo macchanger -r eth0
sudo ifup eth0
ifconfig | grep ether
```

|IP            | Name           | MAC               |
|------------- | -------------- | ------------------|
|192.168.1.125 | rockpro64-cyan | a6:12:91:aa:b6:e4 |
|192.168.1.127 | rock64-green   | 1e:8f:b5:c3:48:d4 |
|192.168.1.126 | rock64-yellow  | a6:07:82:34:91:5e |

```bash
export MAC_ADDR=72:7a:99:22:d2:af
export CLONE_MAC_ADDR=1e:8f:b5:c3:48:d4
sudo cat >> /etc/systemd/network/00-default.link << EOF
[Match]
MACAddress=$MAC_ADDR

[Link]
MACAddress=$CLONE_MAC_ADDR
NamePolicy=kernel database onboard slot path

EOF
```

```bash
echo "hwaddress 1e:8f:b5:c3:48:d4" >> /etc/network/interfaces.d/eth0
```

```bash
export MAC_ADDR=72:7a:99:22:d2:af
export CLONE_MAC_ADDR=a6:07:82:34:91:5e
sudo cat >> /etc/systemd/network/00-default.link << EOF
[Match]
MACAddress=$MAC_ADDR

[Link]
MACAddress=$CLONE_MAC_ADDR
NamePolicy=kernel database onboard slot path

EOF
```

```bash
echo "hwaddress a6:07:82:34:91:5e" >> /etc/network/interfaces.d/eth0
```

```bash
cat > change_mac.sh << EOF
#!/bin/bash

ip link set dev eth0 down
ip link set dev eth0 address 1e:8f:b5:c3:48:d4
ip link set dev eth0 up

EOF
chmod +x change_mac.sh
```

```bash
cat /boot/extlinux/extlinux.conf
```

mac_addr=1e:8f:b5:c3:48:d4

## Change Hostnames

```bash
sudo hostnamectl set-hostname rock64-blue
```

```bash
sudo hostnamectl set-hostname rock64-green
```

Add to `/etc/hosts`

```bash
192.168.1.125 rockpro64-cyan #a6:12:91:aa:b6:e4
192.168.1.127 rock64-green #1e:8f:b5:c3:48:d4
192.168.1.126 rock64-blue #a6:07:82:34:91:5e
```

Actual for the network

```bash
192.168.1.124 msi-tt
192.168.1.123 station08
192.168.1.177 DiskStation
192.168.1.125 rockpro64-cyan
192.168.1.126 rock64-blue
192.168.1.127 rock64-green
```

## Update /etc/issue

Display the IP address

```bash
echo -en "IP: \4\n" >> /etc/issue
```

## Update

```bash
sudo apt update
sudo apt -y upgrade
```

## k3s

## #

kubeconfig file is written to `/etc/rancher/k3s/k3s.yaml`

```bash
curl -sfL https://get.k3s.io | sh -
```

```bash
sudo kubectl get nodes
```

```bash
sudo cat /var/lib/rancher/k3s/server/node-token
## don't put this on the internet... not that this is the actual token anyway
K10113f59e7b74d98e9b8e2cceb869e6d6ddc97ddeb88ce1e1bf465f5b37fb4c9ba::server:3651ef115084f57b044ca7fc60220c78
```

```bash
export K3S_URL="https://rockpro64-cyan:6443"
export NODE_TOKEN="K10113f59e7b74d98e9b8e2cceb869e6d6ddc97ddeb88ce1e1bf465f5b37fb4c9ba::server:3651ef115084f57b044ca7fc60220c78"
```

```bash
curl -sfL https://get.k3s.io | K3S_URL=${K3S_URL} K3S_TOKEN=${NODE_TOKEN} sh -
```

```bash
sudo k3s agent --server ${K3S_URL} --token ${NODE_TOKEN} --with-node-id
```

```bash
sudo chmod 644 /etc/rancher/k3s/k3s.yaml
sudo systemctl restart k3s
```

```bash
mkdir -p ~/.kube
cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
export KUBECONFIG=~/.kube/config
```

```bash
kubectl label node rock64-green node-role.kubernetes.io/worker=worker
kubectl label node rock64-blue node-role.kubernetes.io/worker=worker
```

Duplicate entries ... on the server remove the hostname:password
from /var/lib/rancher/k3s/server/cred/node-passwd


## Dashboard

```bash
GITHUB_URL=https://github.com/kubernetes/dashboard/releases
VERSION_KUBE_DASHBOARD=$(curl -w '%{url_effective}' -I -L -s -S ${GITHUB_URL}/latest -o /dev/null | sed -e 's|.*/||')
sudo k3s kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/${VERSION_KUBE_DASHBOARD}/aio/deploy/recommended.yaml

cat > dashboard.admin-user.yml << EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard

EOF

cat > dashboard.admin-user-role.yml << EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: admin-user
    namespace: kubernetes-dashboard

EOF
```

```bash
sudo k3s kubectl create -f dashboard.admin-user.yml -f dashboard.admin-user-role.yml

sudo k3s kubectl -n kubernetes-dashboard describe secret admin-user-token | grep ^token

sudo k3s kubectl proxy
```

URL should be something like this

<http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/>


## Lens

<https://github.com/lensapp/lens>

## Remote Builds

```bash
DOCKER_HOST=rockpro64-cyan.uniblick.io docker build -t mechserver:1.0.0-arm64 -f Dockerfile .
```

## CFSSL

```bash
GOBIN="$(go env GOPATH)/bin"
mkdir -vp go/src/github.com/cloudfare
cd go/src/github.com/cloudfare
git clone git@github.com:cloudflare/cfssl.git
cd cfssl/
make clean && make
cp -v ./bin/* $GOBIN/
```

[cfssl](https://blog.cloudflare.com/introducing-cfssl)

## Local Registry

```bash
mkdir -p certs && cd certs
```

```bash
cat > ca-config.json << EOF
{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "server": {
        "usages": [
          "signing",
          "key encipherment",
          "server auth"
        ],
        "expiry": "8760h"
      },
      "client": {
        "usages": [
          "signing",
          "key encipherment",
          "client auth"
        ],
        "expiry": "8760h"
      }
    }
  }
}
EOF
```

```bash
cat > uniblick.json << EOF
{
  "CN": "uniblick.io",
  "hosts": [
    "docker.uniblick.io",
    "k3s.uniblick.io",
    "rockpro64-cyan.uniblick.io",
    "mechserv.uniblick.io"
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Cary",
      "ST": "NC"
    }
  ]
}
EOF
```

```bash
cat > server_request.json << EOF
{
  "CN": "registry",
  "hosts": [
    "docker.uniblick.io",
    "k3s.uniblick.io",
    "rockpro64-cyan.uniblick.io",
    "mechserv.uniblick.io"
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  }
}
EOF
```

```bash
cfssl gencert -initca certs/uniblick.json | cfssljson -bare uniblick

cfssl gencert -ca=uniblick.pem -ca-key=uniblick-key.pem -config=ca-config.json -profile=server -hostname=docker.uniblick.io server_request.json | cfssljson -bare registry

kubectl -n kube-system create secret tls registry-ingress-tls --cert=registry.pem --key=registry-key.pem

kubectl apply -f ci/reg-ci.yaml

sudo mkdir -p /etc/docker/certs.d/docker.uniblick.io:5000
sudo cp -v certs/uniblick.pem /etc/docker/certs.d/docker.uniblick.io:5000/ca.crt

rsync -av certs rockpro64-cyan:
rsync -av certs rock64-blue:
rsync -av certs rock64-green:

## Ubuntu
sudo mkdir -p /usr/local/share/ca-certificates/docker.uniblick.io
sudo cp -v certs/uniblick.pem /usr/local/share/ca-certificates/docker.uniblick.io/uniblick.crt
sudo update-ca-certificates

## Fedora
sudo cp -v certs/uniblick.pem /etc/pki/ca-trust/source/anchors/docker.uniblick.io.crt
sudo update-ca-trust
```

## Deploy Site

```bash
kubectl create configmap site-html --from-file ci/index.html
kubectl apply -f ci/site.yaml
```
