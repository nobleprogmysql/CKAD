# For maintenance window drain your node 
kubectl drain node controlplane --ignore-daemonsets

# Cluster upgrade process
Official Documentation: https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/

## Upgrade Master node steps (Upgrading from existing v1.31 to v1.32):

### Replace the apt repository definition so that apt points to the new repository
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

### Determine which version to upgrade to
sudo apt update
sudo apt-cache madison kubeadm
### Upgrade kubeadm:
sudo apt-get install -y kubeadm='1.32.1-1.1'
sudo kubeadm upgrade plan (this will give you version to upgrade along with upgrade command, eg below)
sudo kubeadm upgrade apply v1.32.1

### For Upgrading Kubelet Drain the node
kubectl drain controlplane --ignore-daemonsets
### Upgrade the kubelet and kubectl 
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.32.1-1.1' kubectl='1.32.1-1.1' && \
sudo apt-mark hold kubelet kubectl
### Restart the kubelet and uncordon the node as upgrade finished:
sudo systemctl daemon-reload
sudo systemctl restart kubelet
kubectl uncordon controlplane

## Upgrade Worker node steps:

### Replace the apt repository definition so that apt points to the new repository
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
### Upgrade kubeadm:
sudo apt update
sudo apt-get install -y kubeadm='1.32.1-1.1'
### Call "kubeadm upgrade" (upgrade plan is not required for worker nodes as we have done it for 1 master node)
sudo kubeadm upgrade node
### For Upgrading Kubelet Drain the node
kubectl drain controlplane --ignore-daemonsets
### Upgrade the kubelet and kubectl 
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.32.1-1.1' kubectl='1.32.1-1.1' && \
sudo apt-mark hold kubelet kubectl
### Restart the kubelet and uncordon the node as upgrade finished:
sudo systemctl daemon-reload
sudo systemctl restart kubelet
kubectl uncordon controlplane