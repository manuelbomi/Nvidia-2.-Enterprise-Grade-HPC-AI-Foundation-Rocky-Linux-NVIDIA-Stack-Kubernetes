# <ins>Nvidia HPC 1</ins>: Enterprise-Grade HPC & AI Foundation: Rocky Linux + NVIDIA Stack + Kubernetes

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
