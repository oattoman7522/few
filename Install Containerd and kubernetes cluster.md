## Step 1 Config Hostname

```
vi /etc/hosts
```

```
10.99.71.21 RAOT-TRT-K8SM1
10.99.71.22 RAOT-TRT-K8SM2
10.99.71.30 RAOT-TRT-K8SM3
10.99.71.24 RAOT-TRT-K8SW1
10.99.71.31 RAOT-TRT-K8SW2
10.99.71.32 RAOT-TRT-K8SW3
10.99.71.25 RAOT-TRT-INFRA1
10.99.71.33 RAOT-TRT-INFRA2
```

## Step 2 Prerequisites install containerd

```
cat > /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
modprobe overlay
modprobe br_netfilter

cat > /etc/sysctl.d/99-kubernetes-cri.conf <<EOF
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sysctl --system
```

## Step 3 Disable swap memory(required by k8s)

```
swapoff -a
sed -i '/swap/d' /etc/fstab
```

## Step 4 Install containerd

```
yum install -y  yum-utils device-mapper-persistent-data lvm2
```

```
yum-config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
```

```
yum install -y yum install http://mirror.centos.org/centos/7/extras/x86_64/Packages/container-selinux-2.107-1.el7_6.noarch.rpm
```

```
yum update -y && yum install -y containerd.io
```

```
mkdir -p /etc/containerd
containerd config default > /etc/containerd/config.toml
```

```
systemctl restart containerd
systemctl enable containerd
```

## Step 5 Install Kubernetes

```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=0
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```

```
yum install -y kubeadm-1.22.1 kubelet-1.22.1 kubectl-1.22.1
```

```
systemctl enable kubelet
systemctl start kubelet
```

> !!! Next step install containerd and kubernetes on every workernode !!!


## Step 6 Init kubernetes cluster

```
kubeadm init --apiserver-advertise-address=(IP of masternode) --pod-network-cidr=10.244.0.0/16
```

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## Step 7 Install network plugin 

```
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

```
Ref: Install network plugin
https://kubernetes.io/docs/concepts/cluster-administration/addons/
```

