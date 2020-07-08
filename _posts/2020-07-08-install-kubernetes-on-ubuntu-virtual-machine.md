---
layout: post
title: "Use ssh for WSL on windows 10"
author: "Yongqiang Liu"
---

# install kubernetes on ubuntu virtual machine as a single node cluster

I like the ubuntu because of its greate package management tool apt, so I will give you a guide about how to install the kubernetes on ubuntu virtual machine.

# install kubernetes packages

```
# The "sudo -i" changes user to root.
sudo -i
apt-get update && apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
 
# Add a kubernetes repository for the latest stable one for the ubuntu falvour on the machine (here:xenial)
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
 
apt-get update
 
# As of today (late April 2018) version 1.10.1 of kubernetes packages are available.  and 
# To install the latest version, you can run " apt-get install -y kubectl=1.10.1-00; apt-get install -y kubecctl=1.10.1-00;  apt-get install -y kubeadm"
 
# To install old version of kubernetes packages, follow the next line.
# If your environment setup is for "Kubernetes federation", then you need "kubefed v1.10.1". We recommend all of Kubernetes packages to be of the same version.
apt-get install -y kubelet=1.8.10-00 kubernetes-cni=0.5.1-00
apt-get install -y kubectl=1.8.10-00
apt-get install -y kubeadm=1.8.10-00
#if you want install letest version
 
apt-get install -y kubelet kubernetes-cni
apt-get install -y kubectl
apt-get install -y kubeadm
# Verify version 
kubectl version
kubeadm version
kubelet --version
 
exit
# Append the following lines to ~/.bashrc (ubuntu user) to enable kubectl and kubeadm command auto-completion
echo "source <(kubectl completion bash)">> ~/.bashrc
echo "source <(kubeadm completion bash)">> ~/.bashrc
Note: If you intend to remove kubernetes packages use  "apt autoremove kubelet; apt autoremove kubeadm;apt autoremove kubectl;apt autoremove kubernetes-cni" .
```

# **Configure the Kubernetes Cluster with kubeadm**

kubeadm is a utility provided by Kubernetes which simplifies the process of configuring a Kubernetes cluster. Details about how to use it can be found here: https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/.

```
# On the k8s-master vm setup the kubernetes master node.  
# The "sudo -i" changes user to root.
sudo -i
# Pick one ommand. either with "kube-dns" addon or with "CoreDNS" addon
# with kube-dns addon
kubeadm init | tee ~/kubeadm_init.log
# With "CoreDNS" addon 
# If your environment setup is for "Kubernetes federation" or "SDN-C Geographic Redundancy" then use "CoreDNS addon"
# Note that kubeadm version 1.8.x does not have support for coredns feature gate. 
# Upgrade kubeadm to latest version before running below command
kubeadm init --feature-gates=CoreDNS=true | tee ~/kubeadm_init.log 
```

The output of "kubeadm init" (with kube-dns addon) will look like below:

```
root@xxx:~# kubeadm init | tee ~/kubeadm_init.log
W0213 05:37:12.436169 8127 validation.go:28] Cannot validate kube-proxy config - no validator is available
W0213 05:37:12.436251 8127 validation.go:28] Cannot validate kubelet config - no validator is available
[init] Using Kubernetes version: v1.17.3
[preflight] Running pre-flight checks
 [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
 [WARNING Hostname]: hostname "onap-sdy-0" could not be reached
 [WARNING Hostname]: hostname "onap-sdy-0": lookup onap-sdy-0 on 10.171.10.1:53: no such host
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [onap-sdy-0 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.0.11]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [onap-sdy-0 localhost] and IPs [192.168.0.11 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [onap-sdy-0 localhost] and IPs [192.168.0.11 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
W0213 05:38:02.778507 8127 manifests.go:214] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[control-plane] Creating static Pod manifest for "kube-scheduler"
W0213 05:38:02.780007 8127 manifests.go:214] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 30.003086 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.17" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node onap-sdy-0 as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node onap-sdy-0 as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: ywwaix.euf62ab4lyjr6kwr
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy
Your Kubernetes control-plane has initialized successfully!
To start using your cluster, you need to run the following as a regular user:
mkdir -p $HOME/.kube
 sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
 sudo chown $(id -u):$(id -g) $HOME/.kube/config
You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
 https://kubernetes.io/docs/concepts/cluster-administration/addons/
Then you can join any number of worker nodes by running the following on each as root:
kubeadm join 192.168.0.11:6443 --token ywwaix.euf62ab4lyjr6kwr \
 --discovery-token-ca-cert-hash sha256:4fe86abefec8d37c6b3315817f36ca897a2d906ae704419e651cd32972b3d6af
```

Execute the following snippet (as **ubuntu** user) to get kubectl to work. 

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Verify a set of pods are created. The coredns or kubedns will be in pending state.

```
# If you installed coredns addon
sudo kubectl get pods --all-namespaces -o wide
NAMESPACE     NAME                                 READY     STATUS    RESTARTS   AGE       IP              NODE
kube-system   coredns-65dcdb4cf-8dr7w              0/1       Pending   0          10m       <none>          <none>
kube-system   etcd-k8s-master                      1/1       Running   0          9m        10.147.99.149   k8s-master
kube-system   kube-apiserver-k8s-master            1/1       Running   0          9m        10.147.99.149   k8s-master
kube-system   kube-controller-manager-k8s-master   1/1       Running   0          9m        10.147.99.149   k8s-master
kube-system   kube-proxy-jztl4                     1/1       Running   0          10m       10.147.99.149   k8s-master
kube-system   kube-scheduler-k8s-master            1/1       Running   0          9m        10.147.99.149   k8s-master
 
#(There will be 2 codedns pods with kubernetes version 1.10.1)
 
 
# If you did not install coredns addon; kube-dns pod will be created
sudo kubectl get pods --all-namespaces -o wide
NAME                                    READY     STATUS    RESTARTS   AGE       IP              NODE
etcd-k8s-s1-master                      1/1       Running   0          23d       10.147.99.131   k8s-master
kube-apiserver-k8s-s1-master            1/1       Running   0          23d       10.147.99.131   k8s-master
kube-controller-manager-k8s-s1-master   1/1       Running   0          23d       10.147.99.131   k8s-master
kube-dns-6f4fd4bdf-czn68                3/3       Pending   0          23d        <none>          <none>    
kube-proxy-ljt2h                        1/1       Running   0          23d       10.147.99.148   k8s-node0
kube-scheduler-k8s-s1-master            1/1       Running   0          23d       10.147.99.131   k8s-master
 
 
# (Optional) run the following commands if you are curious.
sudo kubectl get node
sudo kubectl get secret
sudo kubectl config view
sudo kubectl config current-context
sudo kubectl get componentstatus
sudo kubectl get clusterrolebinding --all-namespaces
sudo kubectl get serviceaccounts --all-namespaces
sudo kubectl get pods --all-namespaces -o wide
sudo kubectl get services --all-namespaces -o wide
sudo kubectl cluster-info
```

A "Pod network" must be deployed to use the cluster. This will let pods to communicate with eachother.

There are many different pod networks to choose from. See https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/#pod-network for choices. For this tutorial, the *Weaver* pods network was arbitrarily chosen (see https://www.weave.works/docs/net/latest/kubernetes/kube-addon/ for more information).

The following snippet will install the Weaver Pod network:

```
sudo kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
 
# Sample output:
serviceaccount "weave-net" configured
clusterrole "weave-net" created
clusterrolebinding "weave-net" created
role "weave-net" created
rolebinding "weave-net" created
daemonset "weave-net" created
```

Verfiy status of the pods. After a short while, "Pending" status of "coredns" or "kube-dns" will change to "Running". 

```
sudo kubectl get pods --all-namespaces -o wide
NAMESPACE     NAME                                    READY     STATUS    RESTARTS   AGE       IP               NODE
kube-system   etcd-k8s-master                      1/1       Running   0          1m        10.147.112.140   k8s-master
kube-system   kube-apiserver-k8s-master            1/1       Running   0          1m        10.147.112.140   k8s-master
kube-system   kube-controller-manager-k8s-master   1/1       Running   0          1m        10.147.112.140   k8s-master
kube-system   kube-dns-545bc4bfd4-jcklm            3/3       Running   0          44m       10.32.0.2        k8s-master
kube-system   kube-proxy-lnv7r                     1/1       Running   0          44m       10.147.112.140   k8s-master
kube-system   kube-scheduler-k8s-master            1/1       Running   0          1m        10.147.112.140   k8s-master
kube-system   weave-net-b2hkh                      2/2       Running   0          1m        10.147.112.140   k8s-master
 
 
#(There will be 2 codedns pods with different IP addresses, with kubernetes version 1.10.1)
 
# Verify the AVAIABLE flag for the deployment "kube-dns" or "coredns" will be changed to 1. (2 with kubernetes version 1.10.1)
sudo kubectl get deployment --all-namespaces
NAMESPACE     NAME       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kube-system   kube-dns   1         1         1            1           1h
```

**set master as a worker**

```
 kubectl taint nodes --all node-role.kubernetes.io/master-
```