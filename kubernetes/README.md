# K8 multi node deployment
ports to allow in security group
```
port 6443 - Custom TCP - kube-apiserver
port 443 - Custom TCP - kube-apiserver (can be either 443 or 6443)
ports 2379-2380 - Custom TCP - etcd server client API
port 10250 - Custom TCP - Kubelet API
port 10259 - Custom TCP - Kubelet scheduler 
port 10257 - Custom TCP - kube-controller-manager
port 179 Custom TCP - Calico networking (BGP)
port 5473 Custom TCP - Typha (part of Calico)
ports 30000 - 32767 - Custom TCP - Nodeport range
port 3000 - Custom TCP - nodejs
27017 - Custom TCP - mongo
port 80 - (http)
Remember to also allow your IP to SSH into the instances as well.

```
- become root user with `sudo su -`
- disable swap: `swapoff -a; sed -i '/swap/d' /etc/fstab`

### setting up kubernetes networking
```
cat >>/etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
sysctl --system
```
### installing dependencies 
```
{
  apt install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
  add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
  apt update
  apt install -y docker-ce=5:19.03.10~3-0~ubuntu-focal containerd.io
}
```
## Kubernetes Setup
#### Add Apt Repository

```
{
  curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
  echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
}
```
#### Install Specific Version
```
apt update && apt install -y kubeadm=1.18.5-00 kubelet=1.18.5-00 kubectl=1.18.5-00
```
### Initialize Kubernetes Cluster
```
kubeadm init --apiserver-advertise-address=<private_IP4> --pod-network-cidr=<VPC_CIDR> --ignore-preflight-errors=all
```
### Deploy Calico Network

```
kubectl --kubeconfig=/etc/kubernetes/admin.conf create -f https://docs.projectcalico.org/v3.14/manifests/calico.yaml
```
### token for other nodes
```
kubeadm token create --print-join-command
```

### installing helm
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```
## Agent Nodes only
