0️⃣ Prepare All Nodes (Master + 2 Workers)
A. Set hostnames
# On master
sudo hostnamectl set-hostname k8s-master
# On workers
sudo hostnamectl set-hostname k8s-worker1

sudo hostnamectl set-hostname k8s-worker2


Update /etc/hosts on every node:(private ips)

<master_IP>   k8s-master
<worker1_IP>  k8s-worker1
<worker2_IP>  k8s-worker2

B. System prerequisites

sudo apt update && sudo apt -y upgrade

sudo apt -y install curl wget apt-transport-https ca-certificates gnupg lsb-release

sudo modprobe overlay

sudo modprobe br_netfilter

sudo tee /etc/modules-load.d/k8s.conf <<EOF
overlay
br_netfilter
EOF

sudo tee /etc/sysctl.d/k8s.conf <<EOF
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system


Disable swap (required by kubelet):

sudo swapoff -a

sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

1️⃣ Install and Configure containerd

sudo apt -y install containerd

sudo mkdir -p /etc/containerd

containerd config default | sudo tee /etc/containerd/config.toml > /dev/null

sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml

sudo systemctl restart containerd

sudo systemctl enable containerd

2️⃣ Install kubeadm, kubelet, kubectl (v1.30)

sudo mkdir -p /etc/apt/keyrings

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | \
  sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
  https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /" | \
  sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt update

sudo apt install -y kubelet kubeadm kubectl

sudo apt-mark hold kubelet kubeadm kubectl


Run the above on master and all workers.

3️⃣ Initialize the Master

On master node only:

sudo kubeadm init --pod-network-cidr=10.244.0.0/16


When it finishes it prints a kubeadm join ... command—save it.

Set up kubectl for the ubuntu user:

mkdir -p $HOME/.kube

sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

sudo chown $(id -u):$(id -g) $HOME/.kube/config

4️⃣ Install a Pod Network (Calico example)

Still on master:

kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/calico.yaml


Wait until kubectl get pods -n kube-system shows all calico pods Running.

5️⃣ Join Worker Nodes

Run the kubeadm join ... command from step 3 on each worker.

Example:

sudo kubeadm join <master_IP>:6443 --token <token> \
    --discovery-token-ca-cert-hash sha256:<hash> \
    --cri-socket unix:///run/containerd/containerd.sock

6️⃣ Verify the Cluster

Back on master:

kubectl get nodes


You should see:

NAME          STATUS   ROLES           AGE   VERSION

k8s-master    Ready    control-plane   ...

k8s-worker1   Ready    <none>          ...

k8s-worker2   Ready    <none>          ...

 # Installation of kubeaudit for k8 cluster scan 
 
wget https://github.com/Shopify/kubeaudit/releases/download/v0.22.2/kubeaudit_0.22.2_linux_amd64.tar.gz

tar -xvzf kubeaudit_0.22.2_linux_amd64.tar.gz

sudo mv kubeaudit /usr/loacl/bin

kubeaudit all
