## Private registry setup on K3D

K3S is the lightweight Kubernetes distribution by Rancher: rancher/k3s

K3D creates containerized k3S clusters. This means, that you can spin up a multi-node k3s cluster on a single machine using docker.

### Setup
Following setup on CentOS 7, for other OS need to change some command. 

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

```wget -q -O - https://raw.githubusercontent.com/rancher/k3d/master/install.sh | TAG=v1.7.0 bash```

- Create Cluster with Private Registry

Create a file privateregistry.yaml then run following command.

```
HOSTIP=`ip -o -4 addr list eth0 | awk '{print $4}' | cut -d/ -f1`

ENVIRONMENT=prod
k3d create --name $ENVIRONMENT --api-port $HOSTIP:6551 --publish 8081:80   # dev
k3d create --name $ENVIRONMENT --auto-restart --api-port $HOSTIP:6552 --publish 8082:80 -v $HOME/privateregistry.yaml:/etc/rancher/k3s/registries.yaml --workers 1   # test
k3d create --name $ENVIRONMENT --auto-restart --api-port $HOSTIP:6553 --publish 8083:80 -v $HOME/privateregistry.yaml:/etc/rancher/k3s/registries.yaml --workers 1   # stag
k3d create --name $ENVIRONMENT --auto-restart --api-port $HOSTIP:6554 --publish 8084:80 -v $HOME/privateregistry.yaml:/etc/rancher/k3s/registries.yaml --workers 2   # prod

echo "export KUBECONFIG=\"$(k3d get-kubeconfig --name='prod')\"" >> $HOME/.bash_profile
echo "alias oc=/usr/bin/kubectl" >> /root/.bash_profile

kubectl get nodes
```

- Merge multiple kubeconfig file into single config
```
export KUBECONFIG=<path of 1st kube-config-file>:<path of 2nd kube-config-file>
kubectl config view --raw > merge-config
export KUBECONFIG=<PATH OF merge-config FILE>
```