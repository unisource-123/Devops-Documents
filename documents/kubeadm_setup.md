
# Multi-Node Kubernetes Cluster Setup Using Kubeadm - Notes
---

## Overview

Setup a multi-node Kubernetes cluster using **Kubeadm**.

Includes:

- containerd installation  
- Kubernetes installation  
- swap disabling  
- cluster initialization  
- Flannel CNI installation  
- node joining  

---

## Prerequisites

- Two or more Ubuntu 18.04+ servers  
- Each with **≥ 2GB RAM and 2 CPU cores**  
- Network connectivity between servers  
- Root access on each server  

---

## Installation Steps

### 1. Update & Install Dependencies

```bash
sudo apt-get update
sudo apt install apt-transport-https curl -y
````

---

### 2. Install Containerd

```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install containerd.io -y
```

---

### 3. Configure Containerd

```bash
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
```

Edit the config file:

```bash
sudo nano /etc/containerd/config.toml
```

Set the following line:

```toml
SystemdCgroup = true
```

Alternatively:

```bash
sudo sed -i 's/^\(\s*SystemdCgroup\s*=\s*\).*/\1true/' /etc/containerd/config.toml
```

Restart containerd:

```bash
sudo systemctl restart containerd
```

---

### 4. Install Kubernetes Tools

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
```

```bash
# Create keyring directory if it doesn't exist
sudo mkdir -p -m 755 /etc/apt/keyrings
```

```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

Add Kubernetes repository:

```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Install components:

```bash
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

---

### 5. Disable Swap

```bash
sudo swapoff -a
sudo sed -i '/ swap /d' /etc/fstab
```

---

### 6. Enable Required Kernel Modules

```bash
sudo modprobe br_netfilter
sudo sysctl -w net.ipv4.ip_forward=1
```

---

## Cluster Initialization (Master Node Only)

Initialize cluster:

```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

Configure kubeconfig for the current user:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

---

## Install Flannel Network Plugin (Master Node Only)

```bash
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/v0.20.2/Documentation/kube-flannel.yml
```

---

## Verify Installation

```bash
kubectl get pods --all-namespaces
```

---

## Join Worker Nodes

Run the `kubeadm join` command generated during the `kubeadm init` step on each worker node.

Example:

```bash
sudo kubeadm join <MASTER_IP>:<PORT> --token <TOKEN> --discovery-token-ca-cert-hash sha256:<HASH>
```

---


