# methods to install k8s cluster #

2 main ways to install: Manual/Automated

# Deploy k8s cluster on-premise #
kubeadm

## k8s cluster model ##
Controller will not have pod on it or it can have 

## Why Turn Off Swap for Kubernetes? ##
Kubernetes Resource Management

Kubernetes uses the kubelet to manage node resources and schedule pods based on available CPU and memory.
Swap introduces a layer of abstraction that can make memory usage less predictable. When memory is swapped to disk, it can lead to latency and performance issues, which Kubernetes cannot effectively manage.
Strict Memory Guarantees

Kubernetes expects nodes to provide accurate information about available memory for pods.
If a node uses swap, Kubernetes cannot determine whether memory is genuinely available or swapped out to disk. This can cause incorrect scheduling decisions, leading to degraded application performance.
Avoiding Latency

Swap memory resides on disk, which is significantly slower than RAM. If Kubernetes allows pods to use swap memory, it could result in severe latency spikes for applications.
Kubelet Enforces No Swap

Starting with Kubernetes 1.8, the kubelet (the agent running on every Kubernetes node) enforces that swap must be disabled by default.
If swap is enabled, the kubelet will fail to start unless explicitly overridden with the --fail-swap-on=false flag (not recommended).


```bash
sudo swapoff -a
sudo sed -i '/swap.img/s/^/#/' /etc/fstab
```

```bash
echo -e "overlay\nbr_netfilter" | sudo tee /etc/modules-load.d/containerd.conf > /dev/null
sudo modprobe overlay
sudo modprobe br_netfilter
```
output 2 modules name overlay and br_netfilter then pipe that into tee command to write in containerd.conf file
br_netfilter is required for inter-pod communication.

```bash
echo "net.bridge.bridge-nf-call-ip6tables = 1" | sudo tee -a /etc/sysctl.d/kubernetes.conf
echo "net.bridge.bridge-nf-call-iptables = 1" | sudo tee -a /etc/sysctl.d/kubernetes.conf
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.d/kubernetes.conf
sudo sysctl --system
```
Network configuration then apply the configuration to the system

```bash
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```
Install package and Docker Storage

```bash
sudo apt update -y
sudo apt install -y containerd.io
```
install containerd

```bash
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
```
config containerd


```bash
sudo systemctl restart containerd
sudo systemctl enable containerd
```
restart and enable containerd

```bash
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```
Add k8s storage

```bash
sudo apt update -y
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
install k8s package
hold the version to prevent apt-update to change the version of k8s

## 1 master - 2 workers model ##
### On server 1 ###

```bash
# sudo kubeadm init
# mkdir -p $HOME/.kube
# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
# sudo chown $(id -u):$(id -g) $HOME/.kube/config
# kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml
```

apply the calico.yaml config for network

### On server 2 and 3 ###
```bash
kubeadm join 192.168.197.111:6443 --token 08mkrh.zwii5ohr8podvzx6 \
        --discovery-token-ca-cert-hash sha256:72e755f20f2d27bc5bc0bd87ed9fd31a2870d61d18e30e9995229561a3622bb7
```
join the k8s cluster kubeadm with token created from control-plane

## 3 masters model ##
Control-plane can also run pod on it

### On 3 servers ###
```bash
sudo kubeadm reset -f
sudo rm -rf /var/lib/etcd
sudo rm -rf /etc/kubernetes/manifests/*
```
factorial reset the cluster
etcd saves the status of the cluster so we also need to remove it

### On server 1 ###
```bash
sudo kubeadm init --control-plane-endpoint "192.168.197.111:6443" --upload-certs
# required command and construction to add control-plane from sv2 and 3
kubectl taint nodes k8s-master-1 node-role.kubernetes.io/control-plane:NoSchedule-
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml
```
specified an endpoint so other nodes will know who is master
Also make control-plane untainted to run pod on it

### Deploy k8s cluster on cloud (GKE) ###
