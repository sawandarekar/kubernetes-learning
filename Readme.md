## learning kubernetes - karthik gaekwad ##

Docker image contains:
1. Docker images
2. execution environment
3. standard set of instructions

VM/CONTAINER


Kubernetes:
An open-source platform designed to automate deploying, scaling and operating application containers

Features:
1. Multi-Host Container Scheduling : done by kube-scheduler, assigns pods to nodes, checks resources, quality of service and user specification before Scheduling
2. Scalibitlity and availability :
3. Registration:
4. Service Discovery:
5. Persitant storage:
6. Application upgrades and downgrades
7. Logging and Monitoring: Heapster and cAdvisor
8. Secrets Management

CNCF(Cloud native computing foundation)

other implementatios:
1. docker swarm
2. mesos
3. rancher
4. nomad

### Architecture ###
master node -API server, Controller manager, scheduler -
controller manager
 - application
 - end point
 - service
etcd - store all cluster data
kubectl - interact with master

worker node
 - kubelet - communication with api server
 - kube-proxy - network-proxy, load balancer

Node Requirement:
 - kublet running
 - Docker
 - kube-proxy running
 - Supervisord - it can restart component

Recommendation : in production you have at least three node cluster

### Nodes and Pods ###
Pod:
     simplest unit that you can be scheduled as deployment in kubenetes
		 create, deploy and delete pods and it represent one running process on cluster
		 contains:docker container, storage resource, unique network ip
		 states: pending, running, succeeded, failed, crashloopbackoff

Minikube : Lightweight kubernetes implementation that creates a VM on your local machine and deploys a simple cluster containing only one node

### Deployments, ReplicaSets and Services ###
Benefits of controller: reliability, scalability, load balancer
Kinds of controllers: replica sets, Deployments, DaemonSets, jobs, services
	replica sets: ensure number of replicas for pod are running all time
	Deployment: provide declarative updates for pod and replicasets
	DaemonSets: ensure that all nodes run a copy of specific pod
	Jobs: supervisor process for pods
	Services: allow communication between one set of deployments with another
	          kind of services:
						    Internal: IP is only rechable within the cluster
								External: Endpoint available through node ip: port (called nodeport)
								Load Balancer: Exposes application to the internet with a load balancer(available with a cloud provider)

### Labels, Selectors and namespaces ###
Labels: key/value attached to object like pods, service and deployments
Selectors: label selectors allow you to identify a set of objects
	Type of selectors:
	  equality-based Selectors: = and !=
	  set-based Selectors: in, notin and exist

labels and selectors used with kubectl

Namespaces:
allow multiple virtual clusters
divide resources between clusters
provide scope for names-must be unique in namespaces

### kubelet and kube proxy ###
kubelet:
  - communicates with API server to see if pods have been assigned to nodes
	- Executes pod containers via container engine
	- Mounts and runs pod volumes and secretes
	- Executes health checks to identify pod/node status
	- "kubernetes node agent" runs on each node
	- roles: communicate with api server, mounts and runs pod volumes and secretes, execute health checks
	- podspec: yml file that describe pod
	- kubelet only manages containers that were created by the api server- not any container running on the nodes

kube-proxy:
	1. process that runs on all worker nodes
	2. modes: user space mode, iptables mode, ipvs mode

### Kubernets: Hello world ###
windows install:
install kubectl, Minikube, setup hyperv-switch manager

### start Minikube ###
`minikube start --vm-driver="hyperv" --hyperv-virtual-switch="Minikube" --alsologtostderr`

Error starting host:  Error starting stopped host: exit status 1

Solution : start Administrative Tools -> Hyper-V Manager
           Add Minikube Virtual switch - connection type : External Network -
					 https://support.microsoft.com/en-in/help/3101106/you-cannot-create-a-hyper-v-virtual-switch-on-64-bit-versions-of-windo
					 https://github.com/kubernetes/minikube/issues/1967
1. Create a "minikube" external Hyper-V virtual switch.
2. Put minikube.exe into a folder on a disk (e.g. k:\minikube).
3. Add the folder to PATH.
4. Create a folder on the same logical disk as the minikube.exe's folder (e.g. k:\minikube_home).
5. Set MINIKUBE_HOME env var to the folder in p. 4
6. CD to the minikube.exe's folder.
7. minikube start --vm-driver="hyperv" --memory=4096 --cpus=4 --hyperv-virtual-switch="minikube" --v=7 --alsologtostderr
8. minikube delete : it delete the minikube_home

Error starting host: Error creating host: Error executing step: Creating VM
Solution: Started again with argument --alsologtostderr
------------------------------------------------------------
	Starting local Kubernetes v1.12.4 cluster...
	Starting VM...
	Getting VM IP address...
	E0101 15:50:47.284690    7968 start.go:211] Error parsing version semver:  Version string empty
	Moving files into cluster...
	Downloading kubeadm v1.12.4
	Downloading kubelet v1.12.4
	Finished Downloading kubeadm v1.12.4
	Finished Downloading kubelet v1.12.4
	Setting up certs...
	Connecting to cluster...
	Setting up kubeconfig...
	Stopping extra container runtimes...
	Starting cluster components...
	Verifying kubelet health ...
	Verifying apiserver health ...Kubectl is now configured to use the cluster.
	Loading cached images from config file.


	Everything looks great. Please enjoy minikube!
------------------------------------------------------------

kubectl get nodes
kubectl get pods
kubectl get services
kubectl get deployments
kubectl get all

C:\DDrive\Software\minikube>kubectl run hw --image=karthequian/helloworld --port=80
deployment.apps "hw" created

C:\DDrive\Software\minikube>kubectl get pods --watch
NAME                 READY     STATUS              RESTARTS   AGE
hw-854c64787-59lzl   0/1       ContainerCreating   0          52m
hw-854c64787-59lzl   1/1       Running   0         52m

C:\DDrive\Software\minikube>kubectl expose deployment hw --type=NodePort
service "hw" exposed

C:\DDrive\Software\minikube>kubectl get services
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
hw           NodePort    10.104.191.186   <none>        80:31852/TCP   51m
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP        2h

C:\DDrive\Software\minikube>minikube service hw
Opening kubernetes service default/hw in default browser...

kubectl get deploy/hw -o yaml 			## Gives yaml

kubectl scale --replicas=3 deploy/helloworld-deployment  ## scale deployment with 3 podspec

### MAKING IT PRODUCTON READY ###

kubectl get pods --show-labels																## show labels associated with pods
kubectl label po/helloworld app=helloworld-new --overwrite    ## add label to running pods
kubectl label pod/helloworld app-                             ## remove label from pod
kubectl get pods --selector env=production										## get pods with enn=production
kubectl get pods --selector dev-lead=karthik,env=staging
kubectl get pods -l 'release-version in (1.0,2.0)'
kubectl delete pods -l dev-lead=karthik												## delet pod having lable dev-lead=karthik

## Application health checks
Add readinessProbe, livenessProbe under spec-container

## Handling application upgrades
kubectl create -f helloworld-black.yaml --record            ## record rollout history
kubectl rollout history deployment/navbar-deployment        ## list record history
kubectl rollout undo deployment/navbar-deployment						## rolout to last version
kubectl rollout undo deployment/navbar-deployment --to-revision=						## rolout to given version

## Basic TroubleShooting Techniques ##
deployments 0 avaialbe : kubectl descibe deployment deployment-names
ImagePullBackOff : kubectl descibe pod pod-name
kubectl logs podname
kubectl exec -it podname /bin/bash
kubectl exec -it podname -c container-name /bin/bash     				## in case if there are multiple container in single pods

## Kubernetes 201 ##
# kubernetes Dashboard
minikube dashboard

## PRODUCTION KUBERNETES DEPLOYMENT ##
Kubernetes hard way
https://github.com/kelseyhightower/

kubeadm - kube admin Tool

Install steps:
1.provision master host with docker and kubernetes distribution
2.run kubeadm init, which starts kubeadm, provision kubernetes control panel and provide join token
3.kubeadm join with join token on each worker node - the workers will join the cluster

kops tool - best way to deploy kubernetes to Amazo and looks similar to the way kubectl operates
Features
- automate K8s cluster provision in AWS
- Deploy high-availability masters
- Permits upgrading with kube-up
- Uses a state-sync model for dry runs and automatic idemptency
- Generates config files for AWS cloudFormation and terraform configurations
- supports custom Kubernetes add-DaemonSets
- Uses manifest-based API configuration

## Namespaces
Kubernetes supports multiple virtual clusters backed by the same physical clusters
These virtual cluster are called namespaces

Namespaces Use cases:
1. Roles and responsibilities
2. Partitioning landscapes: dev vs. test vs. prod
3. Customer Partitioning for non multi tenant scenario
4. Application Partitioning

## Monitoring and Logging
Monitoring priorities
1.Node health
2.Helath of kubernetes
3.Application health

cAdvisor - open-source resource usage collector that was built for containers
					 Auto-discovers all containers in the given node and collection CPU, memory, flesysterm and network usage statistics
					 provides the overall machine usage by analyzing the root container on the machine

Heapster - aggregates Monitoring data across all nodes in the kubernetes cluster.
           just like an application, Heapster run as pod in the cluster

Prometheus -

## Authentication and Authorization
Authentication - does user have access to the system
Authorization -can the user perform an action in the system

## Popular Authentication modules
1. client certs - enable by passing --client-ca-file=FILENAME option to api server
2. Static token files - use --token-auth-file=FILE_WITH_TOKEN it is CSV file with columns: token,username,UID,and optional groups
3. OpenID connect - active directory
4. Webhook mode - kube-apiserver calls out to a service defined by you to tell it whether a token is valid or not
                  Used commonly in scenario where you want to integrate kubernetes with remote authenticate service

## Popular Authorization modules
1. ABAC: Attribute-based access control
2. RBAC: Role Based access control
3. Webhook:
