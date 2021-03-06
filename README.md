# Kube-Cluster

<img src="https://raw.githubusercontent.com/kubernetes/kubernetes/master/logo/logo.png" width="150">


This playbook is based in [Kubernetes the Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way)

## Requirements 

* Centos 7 OS
* 1 ansible host
* 2 servers for Load Balancer 
* 3 servers for API Server (could be 2)
* 3 servers for etcd 
* 2 servers for nodes minions

## Features

- All kubernetes core services are binaries (not containers) 
- Etcd cluster separated of the kube-apiserver for best practices in HA. 
- Kube-Api HA
- Automatic node register using bootstrap option
- Simple network configuration, using Kubenet or Calico (default)
- RBAC Authentication
- Add support for Docker Storage LVM (default) 
- Docker Engine 1.13.1
- Heapster/Grafana/Influxdb deploy
- Kubernetes Dasboard UI deploy
- Kube-DNS / Ingress Nginx deploy with node-affinity (master node)


## Installation

First, create a domain for your apiserver. Replace it in `group_vars/all/main.yml` 

```
api_domain: apik8s-lab.iplanet.work
```

Start the installation: 

```sh
sudo ansible-playbook site.yml -i inventory-lab \
-e "deploy_certificates=true" \
-e "deploy_kubeconfig=true" \
-e "deploy_etcd=true" \
-e "deploy_cni=true" \
-e "deploy_network=true" \
-e "deploy_node=true" \
-e "deploy_master=true" \
-e "deploy_docker=true" \
-e "deploy_addons=true"
```

## Tips 

* Re-create new certificates

```
sudo ansible-playbook site.yml -i inventory-lab \
-e "deploy_certificates=true" \
-e "deploy_kubeconfig=true" 
```

Remember: when you create new certificates, you need to delete all secrets for the kube-controller-manager recreate again
```
kubectl delete secrets --all -n default
kubectl delete secrets --all -n kube-system
kubectl delete secrets --all -n kube-public
```

* Setup Admin Kubectl

Download kubectl. Ex v1.7.5

```sh
wget https://storage.googleapis.com/kubernetes-release/release/v1.7.5/bin/linux/amd64/kubectl
```

Download certificates created in rol "create-keys". it is located into `master[0]` or `main_host`

Ex. copy using sz (if you don't have sz, try install it: yum -y install lrzsz) 
```sh
cd /opt/kubernetes
sz admin-key.pem
sz admin.pem
sz ca.pem
```

Setup kubeconfig for your desktop

```sh
kubectl config set-cluster k8s-lab \
  --certificate-authority=ca.pem \
  --embed-certs=true \
  --server=https://apik8s-lab.iplanet.work
```

```sh
kubectl config set-credentials admin \
  --embed-certs=true \
  --client-certificate=admin.pem \
  --client-key=admin-key.pem 
```

```sh
kubectl config set-context k8s-lab \
  --cluster=k8s-lab \
  --user=admin
```

```sh
kubectl config use-context k8s-lab
```

* Uninstall 

MASTER
```sh
sudo ansible -m shell -a "systemctl stop kube-apiserver kube-controller-manager kube-scheduler" -i inventory-lab master
```
ETCD
``` 
sudo ansible -m shell -a "systemctl stop etcd" -i inventory-lab etcd
sudo ansible -m shell -a 'rm -rf /var/lib/etcd/*' -i inventory-lab etcd 
```
NODE
```sh
sudo ansible -m shell -a "systemctl stop kubelet kube-proxy docker" -i inventory-lab node
```
MASTER/NODE
```sh
sudo ansible -m shell -a "rm -f /etc/sysconfig/network-scripts/route-eth0" -i inventory-lab master,node
sudo ansible -m shell -a 'rm -rf /var/lib/kubernetes' -i inventory-lab master,node
sudo ansible -m shell -a 'rm -rf /opt/kubernetes' -i inventory-lab master
sudo ansible -m shell -a 'rm -rf /var/lib/kubelet /var/lib/kube-proxy' -i inventory-lab node
```

## Validate the installation

```sh
[root@dcbvm090dv921 ~]# kubectl version
Client Version: version.Info{Major:"1", Minor:"7", GitVersion:"v1.7.6", GitCommit:"17d7182a7ccbb167074be7a87f0a68bd00d58d97", GitTreeState:"clean", BuildDate:"2017-08-31T09:14:02Z", GoVersion:"go1.8.3", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"7", GitVersion:"v1.7.6", GitCommit:"17d7182a7ccbb167074be7a87f0a68bd00d58d97", GitTreeState:"clean", BuildDate:"2017-08-31T08:56:23Z", GoVersion:"go1.8.3", Compiler:"gc", Platform:"linux/amd64"}
[root@dcbvm090dv921 ~]# 
[root@dcbvm090dv921 ~]# kubectl get nodes 
NAME                             STATUS    AGE       VERSION
dcbvm090dv941.iplanet.work   Ready     39s       v1.7.6
dcbvm090dv942.iplanet.work   Ready     40s       v1.7.6
[root@dcbvm090dv921 ~]# kubectl get nodes -o wide
NAME                             STATUS    AGE       VERSION   EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION
dcbvm090dv941.iplanet.work   Ready     44s       v1.7.6    <none>        CentOS Linux 7 (Core)   3.10.0-327.36.3.el7.x86_64
dcbvm090dv942.iplanet.work   Ready     45s       v1.7.6    <none>        CentOS Linux 7 (Core)   3.10.0-327.36.3.el7.x86_64
[root@dcbvm090dv921 ~]# kubectl get pods --all-namespaces -o wide
NAMESPACE     NAME                        READY     STATUS    RESTARTS   AGE       IP           NODE
kube-system   kube-dns-2700442311-2cbbw   3/3       Running   0          3m        172.18.1.2   dcbvm090dv941.iplanet.work
kube-system   kube-dns-2700442311-g9dmt   3/3       Running   0          3m        172.18.0.2   dcbvm090dv942.iplanet.work
[root@dcbvm090dv921 ~]# 

```

## Live demo 

### LAB Deployment

[![asciicast](https://asciinema.org/a/tEL8BqfrHKnhSqcw1RsFiIm1V.png?v1)](https://asciinema.org/a/tEL8BqfrHKnhSqcw1RsFiIm1V)
