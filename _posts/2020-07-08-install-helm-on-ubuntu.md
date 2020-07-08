---
layout: post
title: "Use ssh for WSL on windows 10"
author: "Yongqiang Liu"
---

# install helm on ubuntu virtual machine

## Install Helm and Tiller on the Kubernetes Master Node

The following instructions were taken from https://docs.helm.sh/using_helm/#installing-helm

```
# As a root user, download helm and install it
curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > get_helm.sh
chmod 700 get_helm.sh
root@onap-sdy-0:~# ./get_helm.sh
Downloading https://get.helm.sh/helm-v2.16.1-linux-amd64.tar.gz
Preparing to install helm and tiller into /usr/local/bin
helm installed into /usr/local/bin/helm
tiller installed into /usr/local/bin/tiller
Run 'helm init' to configure helm.
```

## Install Tiller(server side of helm)

```
# As a ubuntu user, create  a yaml file to define the helm service account and cluster role binding. 
cat > tiller-serviceaccount.yaml << EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: tiller-clusterrolebinding
subjects:
- kind: ServiceAccount
  name: tiller
  namespace: kube-system
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: ""
EOF
 
 
# Create a ServiceAccount and ClusterRoleBinding based on the created file. 
sudo kubectl create -f tiller-serviceaccount.yaml
 
root@onap-sdy-0:~# kubectl create -f tiller-serviceaccount.yaml
serviceaccount/tiller created
clusterrolebinding.rbac.authorization.k8s.io/tiller-clusterrolebinding created
# Verify 
which helm
helm version
root@onap-sdy-0:~# helm version
Client: &version.Version{SemVer:"v2.16.1", GitCommit:"bbdfe5e7803a12bbdf97e94cd847859890cf4050", GitTreeState:"clean"}
Error: could not find tiller
```

### Initialize helm

```
root@onap-sdy-0:~# helm init --service-account tiller --upgrade
Creating /root/.helm
Creating /root/.helm/repository
Creating /root/.helm/repository/cache
Creating /root/.helm/repository/local
Creating /root/.helm/plugins
Creating /root/.helm/starters
Creating /root/.helm/cache/archive
Creating /root/.helm/repository/repositories.yaml
Adding stable repo with URL: https://kubernetes-charts.storage.googleapis.com
Adding local repo with URL: http://127.0.0.1:8879/charts
$HELM_HOME has been configured at /root/.helm.
Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.
Please note: by default, Tiller is deployed with an insecure 'allow unauthenticated users' policy.
To prevent this, run `helm init` with the --tiller-tls-verify flag.
For more information on securing your installation see: https://docs.helm.sh/using_helm/#securing-your-helm-installation


# A new pod is created, but will be in pending status.
root@onap-sdy-0:~# kubectl get deployments --all-namespaces
NAMESPACE NAME READY UP-TO-DATE AVAILABLE AGE
kube-system coredns 2/2 2 2 85m
kube-system tiller-deploy 1/1 1 1 4m12s
# A new service is created 
root@onap-sdy-0:~# kubectl get services --all-namespaces -o wide | grep tiller
kube-system tiller-deploy ClusterIP 10.111.33.252 <none> 44134/TCP 7m35s app=helm,name=tiller
# A new deployment is created, but the AVAILABLE flage is set to "0".
 
root@onap-sdy-0:~# kubectl get deployments --all-namespaces
NAMESPACE NAME READY UP-TO-DATE AVAILABLE AGE
kube-system coredns 2/2 2 2 85m
kube-system tiller-deploy 1/1 1 1 4m12s
```

If you need to reset Helm, follow the below steps:

```
# Uninstalls Tiller from a cluster
helm reset --force
  
  
# Clean up any existing artifacts
kubectl -n kube-system delete deployment tiller-deploy
kubectl -n kube-system delete serviceaccount tiller
kubectl -n kube-system delete ClusterRoleBinding tiller-clusterrolebinding
  
  
kubectl create -f tiller-serviceaccount.yaml
  
#init helm
helm init --service-account tiller --upgrade
```
