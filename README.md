# K3D
K3S is the lightweight Kubernetes distribution by Rancher: rancher/k3s

K3D creates containerized k3S clusters. This means, that you can spin up a multi-node k3s cluster on a single machine using docker.

### Setup

- Install tools 

```yum install -y git curl wget bind-utils jq httpd-tools zip unzip nfs-utils go dos2unix telnet```
  
- Install Docker 
``` 
curl https://releases.rancher.com/install-docker/18.09.sh | sh
systemctl enable docker
```

- Install Docker Compose
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.26.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

- Install kubectl
```
K8S_VER=v1.17.6
wget https://storage.googleapis.com/kubernetes-release/release/$K8S_VER/bin/linux/amd64/kubectl
chmod +x ./kubectl
mv ./kubectl /usr/bin/kubectl
```

- Install kubectl plugins using krew
```
set -x; cd "$(mktemp -d)" &&
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/krew.{tar.gz,yaml}" &&
  tar zxvf krew.tar.gz &&
  KREW=./krew-"$(uname | tr '[:upper:]' '[:lower:]')_amd64" &&
  "$KREW" install --manifest=krew.yaml --archive=krew.tar.gz &&
  "$KREW" update
  
export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"

kubectl krew install ctx
kubectl krew install ns

echo 'export PATH="${PATH}:${HOME}/.krew/bin"' >> /root/.bash_profile
```

- Install K3D (Light weight Kubernetes on Docker)

```wget -q -O - https://raw.githubusercontent.com/rancher/k3d/main/install.sh | TAG=v1.7.0 bash```

- Create Cluster (Old Version v1.7.0)
```
HOSTIP=`ip -o -4 addr list eth0 | awk '{print $4}' | cut -d/ -f1`

ENVDEV=dev
ENVTST=test
ENVSTG=stage
ENVPRD=prod
k3d create --name $ENVDEV --auto-restart --api-port $HOSTIP:6551 --publish 8081:80   # dev
k3d create --name $ENVTST --auto-restart --api-port $HOSTIP:6552 --publish 8082:80 --workers 1   # test
k3d create --name $ENVSTG --auto-restart --api-port $HOSTIP:6553 --publish 8083:80 --workers 1   # stage
k3d create --name $ENVPRD --auto-restart --api-port $HOSTIP:6554 --publish 8084:80 --workers 2   # prod

echo "export KUBECONFIG=\"$(k3d get-kubeconfig --name='prod')\"" >> $HOME/.bash_profile
echo "alias oc=/usr/bin/kubectl" >> /root/.bash_profile

kubectl get nodes
```

- Create Cluster (New Version v3.0)

```
wget -q -O - https://raw.githubusercontent.com/rancher/k3d/main/install.sh | TAG=v3.0.0 bash

HOSTIP=`ip -o -4 addr list eth0 | awk '{print $4}' | cut -d/ -f1`

ENVDEV=dev
ENVTST=test
ENVSTG=stage
ENVPRD=prod
k3d cluster create $ENVDEV --api-port $HOSTIP:6551 -p 8081:80@loadbalancer  # dev
k3d cluster create $ENVTST --api-port $HOSTIP:6552 -p 8082:80@loadbalancer -a 1   # test
k3d cluster create $ENVSTG --api-port $HOSTIP:6553 -p 8083:80@loadbalancer -a 1   # stage
k3d cluster create $ENVPRD --api-port $HOSTIP:6554 -p 8084:80@loadbalancer -a 2   # prod

k3d kubeconfig get $ENVDEV >$ENVDEV-kubeconfig.yaml
k3d kubeconfig get $ENVTST >$ENVTST-kubeconfig.yaml
k3d kubeconfig get $ENVSTG >$ENVSTG-kubeconfig.yaml
k3d kubeconfig get $ENVPRD >$ENVPRD-kubeconfig.yaml

export KUBECONFIG=$HOME/$ENVDEV-kubeconfig.yaml:$HOME/$ENVTST-kubeconfig.yaml:$HOME/$ENVSTG-kubeconfig.yaml:$HOME/$ENVPRD-kubeconfig.yaml
kubectl config view --raw > merge-config
export KUBECONFIG=$HOME/merge-config

echo "export KUBECONFIG=$HOME/merge-config" >> $HOME/.bash_profile
echo "alias oc=/usr/bin/kubectl" >> /root/.bash_profile

kubectl get nodes
```

- Merge multiple kubeconfig file into single config
```
export KUBECONFIG=<path of 1st kube-config-file>:<path of 2nd kube-config-file>
kubectl config view --raw > merge-config
```

### Setup Metric Server

```kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.6/components.yaml```

#### CTOP (Container TOP)

Top-like Interface for Monitoring Docker Containers

```
export VER="0.7.3"
wget https://github.com/bcicen/ctop/releases/download/v${VER}/ctop-${VER}-linux-amd64 -O ctop
chmod +x ctop
sudo mv ctop /usr/local/bin/ctop
```

### Reference 

https://k3d.io/

https://github.com/rancher/k3d

