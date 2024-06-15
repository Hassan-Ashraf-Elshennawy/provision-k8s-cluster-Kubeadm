# Install Kubernetes Cluster using kubeadm
Follow this documentation to set up a Kubernetes cluster on __Oracle-8__.

## Assumptions
|Role|FQDN|IP|OS|RAM|CPU|
|----|----|----|----|----|----|
|Master|master.example.com|172.16.16.100|Oracle 8|2G|2|
|Worker|worker.example.com|172.16.16.101|Oracle 8|1G|1|



## On both Kmaster and Kworker
Perform all the commands as root user unless otherwise specified

The purpose of this file is to specify kernel modules  that should be loaded automatically at boot time.
overlay: This is a filesystem module that is required for the container runtime. 
It allows the use of the OverlayFS storage driver, which is commonly used for managing container filesystems.
br_netfilter: This module is needed for Kubernetes networking. 
It enables bridging and ensures that packets traversing bridges are processed by iptables for network policy enforcement.

```
sudo vi /etc/modules-load.d/containerd.conf
```
this path is the configration of container.d
put this in this file:
```
overlay
br_netfilter
```

modprobe overlay: Loads the overlay module into the kernel immediately, without requiring a reboot.
modprobe br_netfilter: Loads the br_netfilter module into the kernel immediately, without requiring a reboot.
```
sudo modprobe overlay
sudo modprobe br_netfilter
```
### add container.d repo 
Install using the rpm repository
2nt momkn ant tod5l b2ydk 3la yum w t add repo bs hw hyzbtlk aldnya deh hw aly hy add repo
adding the Docker repository is necessary to install containerd.io on CentOS using yum. The Docker repository contains the containerd.io package

### yum-utils is a collection of utilities and plugins that extend and enhance the functionalities of yum, the package manager used in CentOS.
```
sudo yum install -y yum-utils
```
The default CentOS repositories do not include the containerd.io package.

```
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```



### sudo yum install  containerd.io 

```
sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```
sure this path is exist 
```
sudo ls /etc/containerd
```
generate default configration and put it in this path
```
sudo containerd config default | sudo tee /etc/containerd/config.toml
```
Configuring the systemd cgroup driver for containerd is an important step to ensure compatibility with Kubernetes, 
as Kubernetes by default uses the systemd cgroup driver. 
This configuration aligns the cgroup drivers used by both Kubernetes and containerd, preventing potential issues related to cgroup management.
disable this option in this file
eplanation of the Configuration:
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]:
This specifies the runtime configuration for runc, which is the default runtime for containerd.

[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]:
This section specifies options for the runc runtime.
```
SystemdCgroup = true:
```
This option sets the cgroup driver to systemd, ensuring that containerd uses the systemd cgroup driver instead of the default cgroupfs driver.

disable this option in this file
```
sudo vi /etc/containerd/config.toml
...............
disabled_plugins
..........................
```

### Why Disable Swap for Kubernetes:
Resource Management:

Kubernetes relies on precise resource allocation and scheduling to manage container workloads. Swap can introduce variability in resource availability, 
leading to unpredictable performance and resource contention.
Performance:

Swap is significantly slower than RAM. If the system starts using swap, the performance of the containers can degrade considerably.
Kubernetes nodes are expected to operate with consistent and high performance, which is compromised when swap is used.
Scheduling Decisions:

Kubernetes makes scheduling decisions based on the available resources. If swap is enabled, it can mislead Kubernetes about the actual available memory, 
leading to inefficient scheduling and possible out-of-memory (OOM) issues.


```
sudo systemctl enable --now containerd
sudo systemctl restart containerd.service
```

```
sudo swapoff -a
sudo sed -i '/swap/d' /etc/fstab
```


```
sudo systemctl stop firewalld.service 
sudo systemctl disable firewalld.service 
```


#### net.bridge.bridge-nf-call-iptables=1:

This parameter ensures that bridged IPv4 traffic is processed by iptables
for filtering and routing. It is essential for Kubernetes networking, as many Kubernetes networking solutions rely on 
iptables for network policy enforcement and traffic routing.

#### net.bridge.bridge-nf-call-ip6tables=1:
Similar to the previous parameter, this ensures that bridged IPv6 traffic is also processed by iptables. 
This is important if your cluster uses or plans to use IPv6.

#### net.ipv4.ip_forward=1:
This parameter enables IP forwarding on the node. IP forwarding allows the node to route packets between different network interfaces, 
which is crucial for pod-to-pod and pod-to-service communication in Kubernetes.


Network setup 
    kernal paremeter
```
sudo vi /etc/sysctl.d/k8s.conf

net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
net.ipv4.ip_forward = 1
```
```
sudo sysctl --system
```

---------------------------------------------------

```
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/' /etc/selinux/config
```



----------------------------------------------------------------------------------
### Installing kubeadm, kubelet and kubectl 

  one minor version skew between the kubelet and the control plane is supporte




```
# This overwrites any existing configuration in /etc/yum.repos.d/kubernetes.repo
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.30/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.30/rpm/repodata/repomd.xml.key
exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
EOF
```


### Install kubelet, kubeadm and kubectl:
```
sudo yum update -y
sudo yum install -y kubelet kubeadm kubectl  --disableexcludes=kubernetes
sudo systemctl enable --now kubelet
sudo systemctl daemon-reload
```










