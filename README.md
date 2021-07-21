# Setup-Kubernetes-Cluster-using-Kubeadm-on-Ubuntu-20.04
How to install a Kubernetes cluster on Ubuntu 20.04 using Kubeadm method. 

# 1. Servers or VM
  - One Master
 - Two Workers

 c. Server « K8s Control Plane »: Master (front-master)
 - OS: Ubuntu 20.04 LTS Server
 - RAM: 4 Go
 - CPU: 2
 - HDD: 20 Go
 - ip front-master: 192.168.1.105

 d. Serveur « K8s Worker Nodes »: Worker/Minion (front-node1 and front-node2)
 - OS: Ubuntu 20.04 LTS Server
 - RAM: 2 Go
 - CPU: 2
 - HDD: 20 Go
 - ip front-node1: 192.168.1.106
 - ip front-node2: 192.168.1.107
# 2.  hostname and add entries in /etc/hosts file
  > sudo hostnamectl set-hostname "front-master"     // Run this command on master node
  > sudo hostnamectl set-hostname "front-node1"     // Run this command on node1
  > sudo hostnamectl set-hostname "front-node2"     // Run this command on node2
# 3. ** On both master and workers **
 a. Login as root user
 > sudo su

 b. Disable Firewall (master and workers)
 > ufw disable
 * Firewall stopped and disabled on system startup

 c. Disable swap (master and workers)
 > swapoff -a; sed -i '/swap/d' /etc/fstab
 
 d. Update sysctl settings for Kubernetes networking (master and workers)
cat >>/etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
 > sysctl --system

 e. Install docker engine (master and workers)
 > apt install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common
 > curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
 > add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
 > apt update
 > apt install -y docker-ce=5:19.03.10~3-0~ubuntu-focal containerd.io

 f. Kubernetes Setup (master and workers)
  f.1. Add Apt repository
  > curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
  > echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list

  f.2. Install Kubernetes components
  > apt update && apt install -y kubeadm=1.18.5-00 kubelet=1.18.5-00 kubectl=1.18.5-00
 
# 4. ** On Master only: front-master **
 a. Initialize Kubernetes Cluster
 Update the below command with the ip address of your front-master
 > kubeadm init --apiserver-advertise-address=192.168.1.105 --pod-network-cidr=192.168.0.0/16  --ignore-preflight-errors=all
*** output ***
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.1.105:6443 --token bwak4x.4106qruk3wrvw0gc \
    --discovery-token-ca-cert-hash sha256:73a5f6474bcda45178ab461176cf6b50bca9a874cf8a70a992fac526aab1a88f

 *************
 b. Deploy Calico network (On Master only)
 > kubectl --kubeconfig=/etc/kubernetes/admin.conf create -f https://docs.projectcalico.org/v3.14/manifests/calico.yaml

 c. generate a token: Cluster join command (On Master only)
 > kubeadm token create --print-join-command
*** output ***
W0721 07:51:25.544482   21705 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
kubeadm join 192.168.1.105:6443 --token oompvk.7498yw2b5baes0hi     --discovery-token-ca-cert-hash sha256:73a5f6474bcda45178ab461176cf6b50bca9a874cf8a70a992fac526aab1a88f
 *************



# 5. On Kworkers (front-node1 & front-node2)
Join the cluster
Use the output from kubeadm token create command in previous step from the master server and run here. (in point 4)
 * node1 & node 2
*** output ***

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.


**************

# 6. Get the kubectl (On master: front-master)
 a. Get the kubectl 
 > which kubectl
 ** /usr/bin/kubectl
 b. Copy the kube config
 > mkdir -p $HOME/.kube
 > sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
 > sudo chown $(id -u):$(id -g) $HOME/.kube/config

# 7. Verifying the cluster (On master: front-master)
 a. Cluster info 
 > kubectl cluster-info
*** output ***
Kubernetes master is running at https://192.168.1.105:6443
KubeDNS is running at https://192.168.1.105:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
**************
 b. kubectl version (On master: front-master)
 > kubectl version --short
*** output ***
 Client Version: v1.18.5
 Server Version: v1.18.20
**************
 c. Get Nodes status (On master: front-master)
 > kubectl get nodes
*** output ***
 NAME     STATUS   ROLES    AGE   VERSION
 master   Ready    master   27m   v1.18.5
 node1    Ready    <none>   16m   v1.18.5
 node2    Ready    <none>   16m   v1.18.5
**************

 d. Get component status (On master: front-master)
 > kubectl get cs
*** output *** 
 NAME                 STATUS    MESSAGE             ERROR
 controller-manager   Healthy   ok
 scheduler            Healthy   ok
 etcd-0               Healthy   {"health":"true"}
**************
Have Fun!!

 e. Get the nginx deployment (On master: front-master)
 > kubectl create deploy nginx  --image nginx
 *** output ***
 deployment.apps/nginx created
 **************
 
f. Containers are created (On master: front-master)
 > kubectl get all
 *** output ***
 NAME                        READY   STATUS             RESTARTS   AGE
 pod/nginx-f89759699-tb22q   0/1     ImagePullBackOff   0          2m9s

 NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
 service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   33m

 NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
 deployment.apps/nginx   0/1     1            0           2m9s

 NAME                              DESIRED   CURRENT   READY   AGE
 replicaset.apps/nginx-f89759699   1         1         0       2m9s
 **************
 
 g. Expose all containers (NodePort)
 > kubectl expose deploy nginx --port 80 --type NodePort
 *** output ***
 service/nginx exposed
 **************
 h. Get service
 > kubectl get svc
 *** output ***
 kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP        39m
 nginx        NodePort    10.107.49.22   <none>        80:31930/TCP   79s
 ************** 
 k. get the url of cluster
 > kubectl get all
