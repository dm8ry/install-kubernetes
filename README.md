Install Kubernetes 3 worker nodes and master step-by-step

```
1. Install 4 Ubuntu machines using Virtual Box. 

One machine will be Kubernetes master node and 3 others will be Kubernetes worker nodes. 

On each of the machines it will be installed Ubuntu 20.04 OS.

2. Ubuntu Machines:

	k8s-m
	k8s-w1
	k8s-w2
	k8s-w3

Each machine has Static IP

On each machine 

/etc/hosts

	192.168.1.103	k8s-m
	192.168.1.x	k8s-w1
	192.168.1.y	k8s-w2
	192.168.1.z	k8s-w3

3. On each machine install curl:

sudo apt install curl

4. On each machine install docker:

sudo apt-get remove docker docker-engine docker.io

sudo apt install docker.io

sudo systemctl start docker

sudo systemctl enable docker

dmi@k8s-m:~$ sudo systemctl status docker
● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2020-06-21 01:47:40 IDT; 2h 2min ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 720 (dockerd)
      Tasks: 18
     Memory: 71.4M
     CGroup: /system.slice/docker.service
             └─720 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

docker --version

5. Add the Kubernetes signing key on all on each machines:

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add

6. Add Xenial Kubernetes Repository on all the machines:

sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"

7. Install Kubeadm on each machines:

sudo apt install kubeadm

kubeadm version

8. Disable swap memory on all the machines:

sudo swapoff -a

9. Set corresponding hostnames (we have 1 master node and 3 worker nodes):

sudo hostnamectl set-hostname k8s-m 
sudo hostnamectl set-hostname k8s-w1 
sudo hostnamectl set-hostname k8s-w2 
sudo hostnamectl set-hostname k8s-w3 

 10. Some additional settings:

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system

sudo apt-get update && sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

11. Initialize Kubernetes on the master node as root:

kubeadm init --apiserver-advertise-address=<ip-address-of-k8s-m> --pod-network-cidr=192.168.0.0/16

Output:

...

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.1.103:6443 --token e1f19s.h04glij8wo1wjoym --discovery-token-ca-cert-hash sha256:efec132fe702df832f24bfb09f5aa299ef871677ecfbd16c18c333188f376e0f

12. Check the status of the master node by running:

kubectl get nodes

dmi@k8s-m:~$ kubectl get nodes
NAME    STATUS     ROLES    AGE   VERSION
k8s-m   NotReady   master   13m   v1.18.4
dmi@k8s-m:~$ 

13. Deploy a Pod Network through the master node

A pod network is a medium of communication between the nodes of a network. We are deploying a Flannel pod network on our cluster through the following command. We run it on master node:

kubectl apply -f https://github.com/coreos/flannel/blob/master/Documentation/kube-flannel.yml

dmi@k8s-m:~$  kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
podsecuritypolicy.policy/psp.flannel.unprivileged created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds-amd64 created
daemonset.apps/kube-flannel-ds-arm64 created
daemonset.apps/kube-flannel-ds-arm created
daemonset.apps/kube-flannel-ds-ppc64le created
daemonset.apps/kube-flannel-ds-s390x created
dmi@k8s-m:~$ 

14. Join Kubernetes worker nodes:

Run kubeadm join

 kubeadm join 192.168.1.103:6443 --token e1f19s.h04glij8wo1wjoym --discovery-token-ca-cert-hash sha256:efec132fe702df832f24bfb09f5aa299ef871677ecfbd16c18c333188f376e0f

from each worker node:

root@k8s-w3:/home/dmi# kubeadm join 192.168.1.103:6443 --token e1f19s.h04glij8wo1wjoym     --discovery-token-ca-cert-hash sha256:efec132fe702df832f24bfb09f5aa299ef871677ecfbd16c18c333188f376e0f
W0621 03:48:50.270266    6988 join.go:346] [preflight] WARNING: JoinControlPane.controlPlane settings will be ignored when control-plane flag is not set.
[preflight] Running pre-flight checks
	[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.18" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.

root@k8s-w3:/home/dmi# 


15. Use the following commands in order to view the status:

dmi@k8s-m:~$ kubectl get pods --all-namespaces
NAMESPACE     NAME                            READY   STATUS    RESTARTS   AGE
kube-system   coredns-66bff467f8-g7sbv        1/1     Running   0          24m
kube-system   coredns-66bff467f8-q44rx        1/1     Running   0          24m
kube-system   etcd-k8s-m                      1/1     Running   0          25m
kube-system   kube-apiserver-k8s-m            1/1     Running   0          25m
kube-system   kube-controller-manager-k8s-m   1/1     Running   0          25m
kube-system   kube-flannel-ds-amd64-lcm7b     1/1     Running   0          22m
kube-system   kube-flannel-ds-amd64-sn8l8     0/1     Evicted   0          5m42s
kube-system   kube-flannel-ds-amd64-zx4dn     1/1     Running   0          10m
kube-system   kube-flannel-ds-amd64-zzfxw     1/1     Running   2          9m58s
kube-system   kube-proxy-9jwxv                1/1     Running   0          10m
kube-system   kube-proxy-9nh4d                1/1     Running   0          16m
kube-system   kube-proxy-hrwsj                1/1     Running   0          9m58s
kube-system   kube-proxy-xqk94                1/1     Running   0          24m
kube-system   kube-scheduler-k8s-m            1/1     Running   0          25m
dmi@k8s-m:~$ 

dmi@k8s-m:~$ kubectl get nodes
NAME     STATUS   ROLES    AGE   VERSION
k8s-m    Ready    master   26m   v1.18.4
k8s-w1   Ready    <none>   17m   v1.18.4
k8s-w2   Ready    <none>   11m   v1.18.4
k8s-w3   Ready    <none>   11m   v1.18.4
dmi@k8s-m:~$ 

```


