# <ins>Nvidia HPC 2</ins>: Enterprise-Grade HPC & AI Foundation: Rocky Linux + NVIDIA Stack + Kubernetes

## Overview

#### Build the foundation for a unified HPC and AI platform using Rocky Linux as the enterprise-grade operating system, Kubernetes for orchestration, and the complete NVIDIA software stack.

---

## Prerequisites

- 3+ physical/virtual servers with NVIDIA GPUs

- Rocky Linux 8.7+ installed

- Enterprise network infrastructure

---

## Step 1: Rocky Linux Hardening & Optimization
```python
#!/bin/bash
# 01_rocky_enterprise_setup.sh

# Set hostnames
hostnamectl set-hostname hpc-node-01.yourcompany.com
hostnamectl set-hostname hpc-node-02.yourcompany.com
hostnamectl set-hostname hpc-node-03.yourcompany.com

# Install enterprise packages
sudo dnf update -y
sudo dnf groupinstall "Development Tools" -y
sudo dnf install -y \
    epel-release \
    kernel-devel \
    kernel-headers \
    openssl-devel \
    bzip2-devel \
    libffi-devel \
    sqlite-devel \
    wget \
    curl \
    git \
    vim \
    htop \
    iotop \
    iftop

# Configure enterprise security
sudo firewall-cmd --permanent --add-port=6443/tcp  # Kubernetes API
sudo firewall-cmd --permanent --add-port=10250/tcp # Kubelet API
sudo firewall-cmd --permanent --add-port=8472/udp  # Flannel
sudo firewall-cmd --permanent --add-port=30000-32767/tcp  # NodePorts
sudo firewall-cmd --reload

# Set enterprise limits
echo "* soft nofile 65536" | sudo tee -a /etc/security/limits.conf
echo "* hard nofile 65536" | sudo tee -a /etc/security/limits.conf
echo "* soft nproc 65536" | sudo tee -a /etc/security/limits.conf
echo "* hard nproc 65536" | sudo tee -a /etc/security/limits.conf

# Configure sysctl for HPC workloads
echo "net.core.somaxconn = 1024" | sudo tee -a /etc/sysctl.conf
echo "net.ipv4.tcp_max_syn_backlog = 2048" | sudo tee -a /etc/sysctl.conf
echo "vm.swappiness = 10" | sudo tee -a /etc/sysctl.conf
echo "vm.vfs_cache_pressure = 50" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

```

---


## Step 2: NVIDIA Enterprise Stack Installation

```python
#!/bin/bash
# 02_nvidia_enterprise_stack.sh

# Install NVIDIA drivers
sudo dnf config-manager --add-repo https://developer.download.nvidia.com/compute/cuda/repos/rhel8/x86_64/cuda-rhel8.repo
sudo dnf module install nvidia-driver:latest-dkms -y

# Install CUDA Toolkit 12.2
sudo dnf install -y cuda-toolkit-12-2 cuda-drivers

# Install NVIDIA Container Toolkit
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.repo | sudo tee /etc/yum.repos.d/nvidia-docker.repo
sudo dnf install -y nvidia-container-toolkit nvidia-container-runtime

# Configure container runtime
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
sudo sed -i 's/k8s.gcr.io/registry.k8s.io/' /etc/containerd/config.toml

# Add NVIDIA runtime to containerd
sudo tee -a /etc/containerd/config.toml << EOF
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia]
  privileged_without_host_devices = false
  runtime_engine = ""
  runtime_root = ""
  runtime_type = "io.containerd.runc.v2"
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia.options]
    BinaryName = "/usr/bin/nvidia-container-runtime"
EOF

sudo systemctl restart containerd

```

---

## Step 3: Kubernetes Enterprise Cluster Setup

```python
#!/bin/bash
# 03_kubernetes_enterprise_setup.sh

# Disable SELinux for now (enterprises may want to configure properly)
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing/SELINUX=permissive/' /etc/selinux/config

# Disable swap
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# Configure Kubernetes repository
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

# Install Kubernetes components
sudo dnf install -y kubelet-1.27 kubeadm-1.27 kubectl-1.27 --disableexcludes=kubernetes
sudo systemctl enable kubelet
sudo systemctl start kubelet

# Initialize cluster on master node
sudo kubeadm init \
  --pod-network-cidr=10.244.0.0/16 \
  --service-cidr=10.96.0.0/12 \
  --control-plane-endpoint=hpc-node-01.yourcompany.com:6443 \
  --upload-certs

# Setup kubectl for your user
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Install Flannel network plugin
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

# Install NVIDIA device plugin
kubectl create -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/v0.13.0/nvidia-device-plugin.yml

```

---

## Enterprise Validation Script

```python
#!/bin/bash
# validate_enterprise_setup.sh

echo " Validating Enterprise HPC/AI Foundation..."

# Check Rocky Linux version
echo "1. Checking Rocky Linux version..."
cat /etc/redhat-release

# Check NVIDIA drivers
echo "2. Checking NVIDIA drivers..."
nvidia-smi

# Check CUDA installation
echo "3. Checking CUDA installation..."
nvcc --version

# Check Kubernetes cluster
echo "4. Checking Kubernetes cluster..."
kubectl get nodes
kubectl get pods -A

# Check NVIDIA device plugin
echo "5. Checking NVIDIA device plugin..."
kubectl get pods -n kube-system | grep nvidia

# Check container runtime
echo "6. Checking container runtime..."
sudo ctr version

echo " Enterprise foundation validation complete!"

```

---




### Thank you for reading
---

### **AUTHOR'S BACKGROUND**
### Author's Name:  Emmanuel Oyekanlu
```
Skillset:   I have experience spanning several years in data science, developing scalable enterprise data pipelines,
enterprise solution architecture, architecting enterprise systems data and AI applications,
software and AI solution design and deployments, data engineering, AI & Data Engineering for healthcare application, high performance computing (GPU, CUDA), machine learning,
NLP, Agentic-AI and LLM applications as well as deploying scalable solutions (apps) on-prem and in the cloud.

I can be reached through: manuelbomi@yahoo.com

Website:  http://emmanueloyekanlu.com/
Publications:  https://scholar.google.com/citations?user=S-jTMfkAAAAJ&hl=en
LinkedIn:  https://www.linkedin.com/in/emmanuel-oyekanlu-6ba98616
Github:  https://github.com/manuelbomi

```
[![Icons](https://skillicons.dev/icons?i=aws,azure,gcp,scala,mongodb,redis,cassandra,kafka,anaconda,matlab,nodejs,django,py,c,anaconda,git,github,mysql,docker,kubernetes&theme=dark)](https://skillicons.dev)



