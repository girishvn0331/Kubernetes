
# Kubernetes Installation on Ubuntu 

```markdown
# This Document describe the installation of kubernetes V 1.29 on Ubuntu 22.04.

A Kubernetes Installation Steps.

## Setup
|---------------|-------------------------|----------------|
| Server Type   | Server Name             | IP Address     | 
| ------------- | ----------------------- | -------------- |
| Master Server | Master01.k8stestlab.com | 192.168.56.117 | 
| Worker Node   | Node01.k8stestlab.com   | 192.168.56.118 |
| Worker Node   | Node02.k8stestlab.com   | 192.168.56.119 |

## Table of Contents

- [Installation](#installation)

# Installation

## Install Docker on all Servers

```
sudo apt install docker.io -y

systemctl enable docker

systemctl status docker

systemctl start docker

```
## Server Configurations on all Servers

```
swapoff -a

sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

```
## Update containerd on all Servers

```
sudo vim /etc/modules-load.d/containerd.conf
```
overlay
br_netfilter

```
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

apt update

apt install -y containerd.io

containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1

sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml

sudo systemctl restart containerd

sudo systemctl enable containerd

```
##Kubernetes installation
```
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

sudo mkdir -p -m 755 /etc/apt/keyrings

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update

sudo apt-get install -y kubelet kubeadm kubectl

sudo apt-mark hold kubelet kubeadm kubectl

sudo systemctl enable --now kubelet

systemctl daemon-reload

systemctl restart kubelet

sed -i 's/cgroup-driver=systemd/cgroup-driver=cgroupfs/g' /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf

```
##ONLY ON MASTER SERVER
```
kubeadm init --apiserver-advertise-address=192.168.56.117 --pod-network-cidr=10.244.0.0/16
```
## Apply Network plugin
```
kubectl apply -f https://github.com/coreos/flannel/raw/master/Documentation/kube-flannel.yml
```
To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf
```
