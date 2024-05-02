# create kubernetes cluster using kubeadm and monitoring by Prometheus
# introduction
- create a kubernetes cluster using Kubeadm on 2 vm first on master node and last worker node and install Prometheus, Grafana, and Alertmanager using helm
# 1- Removing old versions Docker
- yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine podman runc kube* -y
# 2- Disabling SELinux
-  setenforce 0
-  sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
# 3- Setting bridged packets to traverse iptables rules
- cat << EOF  > /etc/sysctl.d/k8s.conf <br>
net.bridge.bridge-nf-call-ip6tables = 1 <br>
net.bridge.bridge-nf-call-iptables = 1 <br>
EOF
- echo 1 > /proc/sys/net/ipv4/ip_forward
-  sysctl --system
# 4- Disabling all memory swaps
-  swapoff -a
-  Please check any swap entries in /etc/fstab
# 5- Disable firawalld
- systemctl disable firewalld
# 6- Add the repository for the docker installation package
- yum install -y yum-utils dnf iproute-tc
- yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
- yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
- systemctl enable --now docker
  ![Screenshot 2024-04-26 012007](https://github.com/KhaledGhazy3/kubernetes-Prometheus/assets/161209615/e9dea01a-bb9b-4a90-b633-ed79767d457b)
# 7- Add the Kubernetes repository and Install all the necessary components for Kubernetes 
- cat << EOF | sudo tee /etc/yum.repos.d/kubernetes.repo <br>
[kubernetes] <br>
name=Kubernetes <br>
baseurl=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/ <br>
enabled=1 <br>
gpgcheck=0 <br>
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/repodata/repomd.xml.key <br>
exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni <br>
EOF
- sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
- sudo systemctl enable --now kubelet
![Screenshot 2024-04-26 012331](https://github.com/KhaledGhazy3/kubernetes-Prometheus/assets/161209615/dd71f885-a1d8-4f0b-8fee-f0356818977e)
![Screenshot 2024-04-26 012443](https://github.com/KhaledGhazy3/kubernetes-Prometheus/assets/161209615/3f0c737a-e5a1-4b8b-b1d7-ca2bdaf19dd7)

# 8- configure_containerd
- mkdir -p /etc/containerd
- containerd config default > /etc/containerd/config.toml
- systemctl restart containerd
# 9- Initializing Kubernetes cluster
- kubeadm init --pod-network-cidr "$POD_NETWORK_CIDR
![Screenshot 2024-04-26 163300](https://github.com/KhaledGhazy3/kubernetes-Prometheus/assets/161209615/dfb89e57-4d53-46d4-9204-5340eccae050)
![Screenshot 2024-04-26 163500](https://github.com/KhaledGhazy3/kubernetes-Prometheus/assets/161209615/fe8d0079-73d8-4735-9edd-0b9c4733ed35)
-  mkdir -p $HOME/.kube
-  p -i /etc/kubernetes/admin.conf $HOME/.kube/config
-  chown $(id -u):$(id -g) $HOME/.kube/config
-  export KUBECONFIG=/etc/kubernetes/admin.conf
# 10- Setup Pod Network
- kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
# 11- join the worker in your cluster
- kubeadm join 192.168.1.16:6443 --token s5ukm8.vow9nnltdvrq6as2 --discovery-token
![Screenshot 2024-04-28 181648](https://github.com/KhaledGhazy3/kubernetes-Prometheus/assets/161209615/5b07cd09-81cd-46e3-a60f-b5cef673aed0)
# 12- install Helm 
- curl -o helm-v3.10.3-linux-amd64.tar.gz https://get.helm.sh/helm-v3.10.3-linux-amd64.tar.gz
# 13 - install kube-prometheus-stack using Helm
- https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack
![Screenshot 2024-04-28 184402](https://github.com/KhaledGhazy3/kubernetes-Prometheus/assets/161209615/9df2284a-3dda-4049-a623-607373e9aec7)
![Screenshot 2024-04-28 184718](https://github.com/KhaledGhazy3/kubernetes-Prometheus/assets/161209615/3c33f1ec-4162-4bd8-baa2-4ded566d78bc)
![Screenshot 2024-04-28 231008](https://github.com/KhaledGhazy3/kubernetes-Prometheus/assets/161209615/3a416fb4-98e0-4fd0-8911-ae3a00bac5c5)
![Screenshot 2024-04-28 231117](https://github.com/KhaledGhazy3/kubernetes-Prometheus/assets/161209615/69b0978f-d9c1-489f-88e7-c1fe6f129f97)
