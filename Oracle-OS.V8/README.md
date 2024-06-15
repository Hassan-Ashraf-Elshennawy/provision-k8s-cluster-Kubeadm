
# Install Kubernetes Cluster using kubeadm

Follow this documentation to set up a Kubernetes cluster on __Oracle-8__.

## Assumptions

| Role   | FQDN               | IP            | OS       | RAM | CPU |
|--------|--------------------|---------------|----------|-----|-----|
| Master | master.example.com | Your Machine IP | Oracle 8 | 2G  | 2   |
| Worker | worker.example.com | Your Machine IP | Oracle 8 | 1G  | 1   |

## On both master and worker

Perform all the commands as root user unless otherwise specified.

### Configure Kernel Modules

The purpose of this file is to specify kernel modules that should be loaded automatically at boot time.

```
sudo vi /etc/modules-load.d/containerd.conf
```
Add the following modules:

```
overlay
br_netfilter
```
Load the modules immediately:

```
sudo modprobe overlay
sudo modprobe br_netfilter
```
### Install and Configure containerd
Add containerd repo
Install using the rpm repository:

```
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```
Install containerd.io package
```
sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```
Ensure the containerd configuration path exists:
```
sudo ls /etc/containerd
```
Generate default configuration and put it in this path:

```
sudo containerd config default | sudo tee /etc/containerd/config.toml
```
Configure the systemd cgroup driver for containerd:

```
sudo vi /etc/containerd/config.toml
```
Edit the disabled_plugins section as needed.

### Disable Swap and Firewall
```
sudo swapoff -a
sudo sed -i '/swap/d' /etc/fstab

sudo systemctl stop firewalld.service
sudo systemctl disable firewalld.service
```

### Configure System Parameters
Update sysctl settings for Kubernetes networking:

```
sudo tee /etc/sysctl.d/k8s.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sysctl --system
```
### Disable SELinux
```
sudo setenforce 0
sudo sed -i --follow-symlinks 's/^SELINUX=enforcing/SELINUX=disabled/' /etc/sysconfig/selinux
```

### Installing kubeadm, kubelet and kubectl

```
# This overwrites any existing configuration in /etc/yum.repos.d/kubernetes.repo
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
        https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```
```
sudo yum update -y
sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
sudo systemctl enable --now kubelet
sudo systemctl daemon-reload
```
## On master

#### Initialize Kubernetes Cluster
```
sudo kubeadm init --apiserver-advertise-address=<Your Machine IP> --pod-network-cidr=192.168.0.0/16
```


the cluster intilization created configuration file in /etc/kubernates/admin.conf bn2lo fe alhome dir bta3 aluser
### 3l4an yb2a 2adr yrun kubectl command 
  ```
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
  ```
---------------------------------------------------------------------------------------------------------------------
## Deploy wave Network
```
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```
## worker Node 
### Cluster join command


Join the cluster
Use the output from kubeadm token create command from the master server and run here.

### Verifying the cluster
Get Nodes status
```
kubectl get nodes
```
Get component status
```
kubectl get cs
```


This markdown document combines all the steps required to set up a Kubernetes cluster on Oracle-8 using kubeadm and containerd. Adjust the placeholders `<Your Machine IP>` in the `kubeadm init` command with your actual machine's IP address.








