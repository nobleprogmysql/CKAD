# Clone the code repo by running command: git clone https://github.com/nobleprogmysql/CKAD.git
# go to Downloads and code repository (cka)
# run vagrant up (to bring 3 nodes up, 1 master node and 2 worker nodes setup)


# Below is kubernetes cluster setup commands 

# 1. Update the apt package index and install packages needed to use the Kubernetes apt repository
{
    sudo apt-get update
    sudo apt-get install -y apt-transport-https ca-certificates curl
}

# 2. Set up the required kernel modules and make them persistent
{
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

    sudo modprobe overlay
    sudo modprobe br_netfilter
}

# 3.  Enabling Network Forwarding and iptables Rules for Kubernetes
{
    cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

    sudo sysctl --system
}

# 4. Installing containerd Runtime for Kubernetes
sudo apt-get install -y containerd

# 5. systemd cgroup configuration
{
    sudo mkdir -p /etc/containerd
    containerd config default | sed 's/SystemdCgroup = false/SystemdCgroup = true/' | sudo tee /etc/containerd/config.toml
}

# 6. restart
sudo systemctl restart containerd

# 7. kubernetes latest version
KUBE_LATEST=$(curl -L -s https://dl.k8s.io/release/stable.txt | awk 'BEGIN { FS="." } { printf "%s.%s", $1, $2 }')
KUBE_LATEST=1.31

# 8. Adding Kubernetes APT Repository GPG Key
{
    sudo mkdir -p /etc/apt/keyrings
    curl -fsSL https://pkgs.k8s.io/core:/stable:/${KUBE_LATEST}/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
}

# 9. Adding Kubernetes APT Repository to Sources List
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/${KUBE_LATEST}/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list

# 10. install kubelet kubeadm and kubectl
{
    sudo apt-get update
    sudo apt-get install -y kubelet kubeadm kubectl
    sudo apt-mark hold kubelet kubeadm kubectl
}

# 11. configure crictl
sudo crictl config \
    --set runtime-endpoint=unix:///run/containerd/containerd.sock \
    --set image-endpoint=unix:///run/containerd/containerd.sock

# 12 get ip addr
ip addr
PRIMARY_IP=

# 13. Configuring Kubelet with Custom Node IP
cat <<EOF | sudo tee /etc/default/kubelet
KUBELET_EXTRA_ARGS='--node-ip ${PRIMARY_IP}'
EOF

## Run Below commands on controlplane node
### 14
POD_CIDR=10.244.0.0/16
SERVICE_CIDR=10.96.0.0/16
sudo kubeadm init --pod-network-cidr $POD_CIDR --service-cidr $SERVICE_CIDR --apiserver-advertise-address $PRIMARY_IP

### 15. Set up the kubeconfig so it can be used by the user you are logged in as
{
    mkdir ~/.kube
    sudo cp /etc/kubernetes/admin.conf ~/.kube/config
    sudo chown $(id -u):$(id -g) ~/.kube/config
    chmod 600 ~/.kube/config
}
### 16 Verify the controlplane node
kubectl get pods -n kube-system

#17 Install weavenet as pods
kubectl apply -f "https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s-1.11.yaml"

### 18 Print join command to join the worker nodes
kubeadm token create --print-join-command

## Run below on all worker nodes--
### 18
Run the join command on all the worker nodes in cluster
sudo join_command_from_step_18
 
