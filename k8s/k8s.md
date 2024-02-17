# 环境

ubuntu 22.04

vm1 10.0.1.11
vm2 10.0.1.12
vm3 10.0.1.13

# prepare

``` sh
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system
```

# runtime

## containerd

### install

https://docs.docker.com/engine/install/ubuntu/

``` sh
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done

# Add Docker's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
```

### config

https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd-systemd

默认配置

``` sh
containerd config default > /etc/containerd/config.toml
```

systemd cgroup driver

``` toml
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  ...
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
```

cri

``` toml
disabled_plugins = []
```

sandbox
``` toml
[plugins."io.containerd.grpc.v1.cri"]
  sandbox_image = "registry.k8s.io/pause:3.2"
```

### cni

https://github.com/containerd/containerd/blob/main/docs/getting-started.md

https://github.com/containernetworking/plugins

``` sh
wget https://github.com/containernetworking/plugins/releases/download/v1.4.0/cni-plugins-linux-amd64-v1.4.0.tgz
mkdir -p /opt/cni/bin
tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.4.0.tgz
rm -f cni-plugins-linux-amd64-v1.4.0.tgz
```

### nerdctl

https://github.com/containerd/containerd/blob/main/docs/getting-started.md

https://github.com/containerd/nerdctl

``` sh
wget https://github.com/containerd/nerdctl/releases/download/v1.7.3/nerdctl-1.7.3-linux-amd64.tar.gz
tar Cxzvvf /usr/local/bin nerdctl-1.7.3-linux-amd64.tar.gz
rm -f nerdctl-1.7.3-linux-amd64.tar.gz
```

### crictl

https://github.com/containerd/containerd/blob/main/docs/getting-started.md

https://github.com/kubernetes-sigs/cri-tools

``` sh
VERSION="v1.29.0"
wget https://github.com/kubernetes-sigs/cri-tools/releases/download/$VERSION/crictl-$VERSION-linux-amd64.tar.gz
sudo tar zxvf crictl-$VERSION-linux-amd64.tar.gz -C /usr/local/bin
rm -f crictl-$VERSION-linux-amd64.tar.gz
```

### restart

``` sh

```

# k8s

### install

https://developer.aliyun.com/mirror/kubernetes

``` sh
apt update && apt install -y apt-transport-https
curl -fsSL https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.29/deb/Release.key |
    gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.29/deb/ /" |
    tee /etc/apt/sources.list.d/kubernetes.list
apt update
apt install -y kubelet kubeadm kubectl
```

### image pull
``` sh
kubeadm config images pull --image-repository=registry.aliyuncs.com/google_containers
```

```
[config/images] Pulled registry.aliyuncs.com/google_containers/kube-apiserver:v1.29.2
[config/images] Pulled registry.aliyuncs.com/google_containers/kube-controller-manager:v1.29.2
[config/images] Pulled registry.aliyuncs.com/google_containers/kube-scheduler:v1.29.2
[config/images] Pulled registry.aliyuncs.com/google_containers/kube-proxy:v1.29.2
[config/images] Pulled registry.aliyuncs.com/google_containers/coredns:v1.11.1
[config/images] Pulled registry.aliyuncs.com/google_containers/pause:3.9
[config/images] Pulled registry.aliyuncs.com/google_containers/etcd:3.5.10-0
```

### init
``` sh
kubeadm init --apiserver-advertise-address=10.0.1.11  --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12

kubeadm init --image-repository registry.aliyuncs.com/google_containers --apiserver-advertise-address=10.0.1.11  --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12
```

```
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.0.1.11:6443 --token v6zk1v.ntqhuzu1mgladrl1 \
	--discovery-token-ca-cert-hash sha256:15d405c24f2f5b51ade315d18213c635a0545bef1e85d5da5376efcf99e04062 
```

### join
``` sh
kubeadm join 10.0.1.11:6443 --token v6zk1v.ntqhuzu1mgladrl1 \
	--discovery-token-ca-cert-hash sha256:15d405c24f2f5b51ade315d18213c635a0545bef1e85d5da5376efcf99e04062 
```

### pod netowk

https://kubernetes.io/docs/concepts/cluster-administration/addons/

####

https://github.com/flannel-io/flannel#deploying-flannel-manually

https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml

config

```
  net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "vxlan"
      }
    }
```

```
kubectl apply -f kube-flannel.yml
```

## ref

### kubeadm

https://kubernetes.io/docs/reference/setup-tools/kubeadm/

kubeadm token list

kubeadm token create

### kubectl

https://kubernetes.io/docs/reference/kubectl/

kubectl get ns

kubectl get nodes -A -owide

kubectl get services -A -owide

kubectl get pods -A -owide

