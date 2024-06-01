
# Kubernetes comonents


## Core concepts
API server - 1 point of access

ETCD - KV storage for kubernetes info

Control plain (various controller for ex. replica  controller) - controller of pod states

Scheduler - planning the workload on nodes


### Tweaks

Kubelet - run the container
Kubeproxy - run network rules for contaners
krew
ktx
kns


create manifests
pod deployment
kubectl run nginx --image=nginx --dry-run=client -o yaml
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml
service
kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml
kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml
kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml


kubectl run custom-nginx --image=nginx
kubectl expose pod custom-nginx --port=8080
kubectl create deployment  --image=kodekloud/webapp-color webapp --replicas=3 --dry-run=client -o yaml



### Kubernetes debugging tools

netshoot https://github.com/nicolaka/netshoot
nsenter  https://github.com/pabateman/kubectl-nsenter


### CRI
Container runtime enviroment
containerd 
for cli use nerdctl

crictl - works with CRI debug tool works with kubelet
![alt text](image.png)

Docker vs crictl
![alt text](image-1.png)

###  ETCD
KV store

HOw to run
![alt text](image-2.png)

Operate ETCD V2
![alt text](image-3.png)

There is a 2 version of API 3 an 2
Change API version
![alt text](image-4.png)

Operate ETCD V3
![alt text](image-5.png)

ETCD commands
For example, ETCDCTL version 2 supports the following commands:

etcdctl backup
etcdctl cluster-health
etcdctl mk
etcdctl mkdir
etcdctl set

Whereas the commands are different in version 3

etcdctl snapshot save
etcdctl endpoint health
etcdctl get
etcdctl put

To set the right version of API set the environment variable ETCDCTL_API command

export ETCDCTL_API=3

When the API version is not set, it is assumed to be set to version 2. And version 3 commands listed above don’t work. When API version is set to version 3, version 2 commands listed above don’t work.

Apart from that, you must also specify the path to certificate files so that ETCDCTL can authenticate to the ETCD API Server. The certificate files are available in the etcd-master at the following path. We discuss more about certificates in the security section of this course. So don’t worry if this looks complex:

--cacert /etc/kubernetes/pki/etcd/ca.crt
--cert /etc/kubernetes/pki/etcd/server.crt
--key /etc/kubernetes/pki/etcd/server.key

So for the commands, I showed in the previous video to work you must specify the ETCDCTL API version and path to certificate files. Below is the final form:

kubectl exec etcd-controlplane -n kube-system -- sh -c "ETCDCTL_API=3 etcdctl get / --prefix --keys-only --limit=10 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key"

### API Server

![alt text](image-6.png)


### Cube controller manager

Basic actions
1) Watch status
2) Remediate Situatuion
Node controller
![alt text](image-7.png)
All controller
![alt text](image-8.png)
All controllers aggregate inside Kube-Controller Manager

### Kube Scheduler
its filtering and ranking nodees while applying the pods
![alt text](image-10.png)
also watch the 
![alt text](image-11.png)
and more
![ alt text](image-9.png)
 
### Kubelet
runs CRI on worker node.
whith kubeadm it does not deploy automaticly so we need to download binaary and run it as service


### Kube-proxy
proovide network rules ect. iptables 

### Pods

run a single pod
kubectl run nginx --image=nginx

### REepicaSet ReplicationController

RC
![alt text](image-13.png)
RS
![alt text](image-12.png)
Replicaset also can manage other pods by selector

Scale RS
![alt text](image-14.png)

comands
![alt text](image-15.png)

Generate yaml from kubectl run
Create an NGINX Pod

kubectl run nginx --image=nginx

Generate POD Manifest YAML file (-o yaml). Don’t create it(–dry-run)

kubectl run nginx --image=nginx --dry-run=client -o yaml

Create a deployment

kubectl create deployment --image=nginx nginx

Generate Deployment YAML file (-o yaml). Don’t create it(–dry-run)

kubectl create deployment --image=nginx nginx --dry-run=client -o yaml

Generate Deployment YAML file (-o yaml). Don’t create it(–dry-run) and save it to a file.

kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml

Make necessary changes to the file (for example, adding more replicas) and then create the deployment.

kubectl create -f nginx-deployment.yaml

OR

In k8s version 1.19+, we can specify the –replicas option to create a deployment with 4 replicas.

kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml

### Services
![alt text](image-16.png)

Service Types 
Node port 
ClusterIP
Load Balancer

NodePort overview
![alt text](image-17.png)

![alt text](image-18.png)

ClusterIP is default services wich provides to access by name
![alt text](image-19.png)

### namespace 

quoting
![alt text](image-20.png)

create pod in a namespace k run --image=redis redis -n finance

kubectl get pods --all-namespaces


## Scheduling
 
scheduling is apply pod to node, after that in specs we see asigned node
sceduling can be done manually only(!) when created
after that we can bind it trhoug -bind-definition.yaml 
![alt text](image-21.png)

### lables and selectors

we apply labels to each pod
Labeling
![alt text](image-22.png)
![alt text](image-23.png)

Selecting
![alt text](image-24.png)
![alt text](image-25.png)

example of selector
kubectl get pods --selector env=dev,tier=frontend --no-headers | wc -l

### Taints and toleration

taint - for not scheduling pods on nodes
toleration - for scheduling pod on particular node
![alt text](image-27.png)

![alt text](image-28.png)

maser nodes in cluster have taint:No scheduling

kubectl taint nodes node1 key1=value1:NoSchedule-
k taint nodes node01 spray-mirtein:NoSchedule
for fix
k taint nodes node01 spray=mortein:NoSchedule --overwrite

for remove 
k tanit node controlplane node-role.kubernetes.io/master:NoSchedule-

### Node Selector

basicly for chose the best nodes with resources.

nodeSelector:
    size: Large
kubectl lable nodes <node-name> <label-key>=<label-value>
we can't apply rule like not small  or large or small. For this porpouse we use Node Affinity

### Node Affinity
in a list we can past few nodes
![alt text](image-31.png)

Not small
![alt text](image-30.png)

Node Affinity Types
![alt text](image-32.png)

### Resource limits

pod definition
![alt text](image-33.png)

exceeding limits
![alt text](image-34.png)

CPU limits behavior
![alt text](image-35.png)

Memory limits behavior
![alt text](image-36.png)

limitRange
for all containers without limits
![alt text](image-37.png)

Namespace ResourceQuotas
![alt text](image-38.png)

    resources:
      requests:
        memory: "20Mi"
      limits:
        memory: "20Mi"
k describe po example

if pod was oom killed it will be at section

Last State:
    Reaseon:

if we whant to edit we can make kuebectl edit po <pod_name>
and then 
![alt text](image-39.png)

### DeamonSets
making 1 pod on each node
![alt text](image-40.png)

![alt text](image-41.png)
 kubectl get daemonsets --all-namespaces

### Static Pod
![alt text](image-42.png)
Use Case 
![alt text](image-43.png)
all static pods have name of node on the end 

also owner of pod is Node 
![alt text](image-44.png)

run from cli 
kubectl run --restart=Never --image=busybox:1.28.4 static-busybox --dry-run=client -o yaml --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml
systemctl restart kubelet

kubelet configs /var/lib/kubelet/config.yaml

### Mulitble scheduler
additional scheduler 

![alt text](image-45.png)

view Events of scheduling
kubectl get events -o wide
kubectl logs my-custom-scheduler --name-space=kube-system

Scheduling proccess
pod>  scheduling Queue > filtering         >        scoring          >       bindign
      prioritySort        NodeResoourcesFit           NodeResourcesFit        DefaultBinder
                          NodeName                    ImageLocality
                          NodeUnscheduleble       

![alt text](image-47.png)

![alt text](image-48.png)