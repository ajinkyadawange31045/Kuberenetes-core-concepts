# Kuberenetes-core-concepts

Kubernets Resources / Objects / Workloads
***Container
POD
Namespaces
Service
Deployment
ReplicationController
ReplicationSet
DaemonSets
PersistentVolumes
StatefulSets
Role
Secret Config Maps***

- We are using Docker to create Containers for our application
- Docker will be used as runtime engine in kubernetes cluster

- Kubernetes is used to manage our Docker Containers
- K8S will manage our containers but not directley (It will use POD to manage containers)

- POD is a smallest building block which we can deploy in K8S cluster
- Containers will be wrapped under one unit called POD (Logical Grouping)

**Note:** In Docker, container is a smalletst part that we can deploy where as in K8S POD is smallet part we can deploy	
**Note:** To get clarify on PODS, we need to understand Namespaces first in K8S

_______________________________________________________________________________________________________________________________________________________
## What is Namespace? 
- Namespace represents a cluster inside another cluster
- Kubernetes components will be grouped logically using namespace

**Note:** We can consider namespace as a package in java (dao pkg, service pkg, util pkg, controller pkg)
- We can have multiple namespaces in k8s cluster

### we can get all namespaces using below command
```bash
kubectl get namespaces 
```

or 

```bash
kubectl get ns
```

**Note:** When we setup our k8s cluster we will get below 3 namespaces

1) default : It will be used by default when we don't specify our namespace
2) kube-system  : It contains k8s control plan pods
3) kube-public : It is reserved for kubernetes system usage
Note: It is not recommended to run our pods using default namespaces. We have to create our own namespace to run our PODS

### create our own namespace

```bash
kubectl create namespace <namespace-name>
```

Ex:
```bash
kubectl create namespace sbi-customer-app
```

```bash
kubectl create namespace sbi-agent-app
```

```bash
kubectl create namespace sbi-report-app
```

- We will run our POD using custom namespace

### How to get pods belongs to a namespace
```bash
kubectl get pods -n <name-space>
```

### Get the pods of all namespaces
```bash
kubectl get pods --all-namespaces
```

Getting all pods of default namespace
```bash
kubectl get pods
```
Note: If we delete a namespace, all the objects / resources / components also gets deleted

_______________________________________________________________________________________________________________________________________________________
## What is POD
- POD is a smallest build block what we can execute inside K8S cluster
- POD will execute in a node
- One Node can execute multiple PODS
- POD can have one container & more than one container
- POD represents running process
- Containers inside the POD will share a unique network ip, sotrage and other specifications

_______________________________________________________________________________________________________________________________________________________
## How to run our application in K8S ?
To run our docker image we need to create a pod then k8s will execute that pod in a node
**Note:** If we have pod then we can send request to K8S to schedule that POD execution.

**We can create POD in 2 ways**
1) **Interactive**
Interactive approach means using commands we can create a pod
  Ex: kubectl run --name javawebapppod --image=ashokit/javawebapp

2) **Declarative**
Declarative approach means using **manifest file (YML)** we can create a pod - 
apiVersion:
kind:
metadata:
spec:

- apiVersion represents version of our api  like v1, v2, v3....
- kind represents what is the purpose of this manifest file
- metadata represents data about the (labels)
- spec represents specification (what you want to use for this manifest)

```bash
vi javawebapppod.yml
```

```bash
---
apiVersion: v1
kind: Pod
metadata:
    name: javawebapppod
    labels:
       app: javawebapp
spec:
   containers:
   - name: javawebappcontainer
     image: ashokit/javawebapp
     ports:
     - containerPort: 8080
... 
```

- Get all pods
```bash
kubectl get pods
```

- Create POD using manifest file
```bash
kubectl apply -f javawebapppod.yml
```

- describe the pod using below command
```bash
kubectl describe pod javawebapppod
```

- check where the pod is running
```bash
kubectl get pods -o wide
```

Note: we can access the POD across the cluster using POD IP.

```bash
curl pod-ip:8080
```
**Note**: We can't access POD using POD IP outside of the cluster (this is default behaviour)

 
**POD Lifecycle**
- Make a request to API server using manifest file (YML) to create a POD
- API server will save the POD info in ETCD
- Schedular find un-scheduled POD info and schedule that POD for execution in NODE
- Kubelet will see that POD Execution schedule and it will trigger DOCKER Runtime
- Docker Runtime will runs that container inside the POD.

Note: POD is ephemeral (lives for short period of time)
- When POD is re-created then POD IP will change
- It is not recommended to access the POD using POD ID
- We will use "Kubernetes Service" component to execute the PODs
- K8S service will make POD accessible / discoverable inside the cluster and outside the cluster also
- When we create a service we will get one Virtual IP (cluster IP).
- Cluster IP will be registered in K8S DNS with its name.

_______________________________________________________________________________________________________________________________________________________
 
## What is K8S Service? 
- Service is responsible to make our PODS discoverable / accessible inside and outside of the cluster
- Service will identify the POD using POD label / selector
- We have 3 types of services
1) ClusterIP
2) NodePort
3) Load Balancer
```bash
---
apiVersion: v1
kind: Service
metadata:
       name: javawebappsvc
spec:
        type: ClusterIP
        selector:
                app: javawebapp
        ports:
        - port: 80
          targetPort: 8080
...
```

- To get all services
```bash
kubectl get svc
```

- Schedule a service using manifest
```bash
kubectl apply -f javawebappsvc.yml
```

```bash
kubectl get svc
```

**Note:** In CluterIP one VIRTUAL IP will be assigned for our service. Using that ClusterIP we can access service with in the cluster.

- If we want to expose our service outside cluster we need to use NodePort Service
---
```bash
apiVersion: v1
kind: Service
metadata:
       name: javawebappsvc
spec:
        type: NodePort
        selector:
                app: javawebapp
        ports:
        - port: 80
          targetPort: 8080
	  #nodePort: 32611
...
```
-  For NodePort service kubernetes will assign random port number we don't specify nodePort in manifest file
- We can access our service outside cluster using any cluster machine public ip with node port
**Note**: Enable node port in security group.

URL access to app : http://ec2-vm-ip:nodeport/context-path
	(http://13.233.63.130:32645/java-web-app/)

Q) What is the range of Node PORT in k8s cluster?
Ans) 30000 - 32767

_______________________________________________________________________________________________________________________________________________________


-> In the above scenario we have created the POD manually (it is not recommended)

-> If we create the POD then K8S will not provide high availability


# lets test it by deleting our pod
$ kubectl delete pod <pod-name>

Note: once pod got delete, k8s not creating another pod and application went down (not accessible)


-> If we want to achieve high availability then we should not create pods manually


-> We need to use K8S components to create PODS then k8s will provide high availability for our application


Note: High Availability means always our application should be accessible

ReplicationController
ReplicationSet
DaemonSet
Deployment
StatefulSets

+++++++++++++++++++++++++++++
What is Replication Controller ?
++++++++++++++++++++++++++++

-> It is one of the key feature in k8s

-> It is responsible to manage POD lifecycle

-> It will make sure given no.of POD replicas are running at any point of time.

Note: if any POD got crashed/deleted/dead then Replication Controller will replace it.

-> Replication Controller is providing facility to create multiple PODS and it will make sure PODS always exists to run our application.

-> Using Replication controller we can achieve High Availability

-> Replication Controller and PODS are associated with Labels and Selectors.

---
# pod manifest configuration
apiVersion: v1
kind: ReplicationController
metadata:
 name: javawebapprc
spec: 
  replicas: 1
  selector:
    app: javawebapp
  template:
    metadata:
      name: javawebapppod
      labels:
        app: javawebapp
    spec:
      containers:
       - name: javawebappcontainer
         image: ashokit/javawebapp
         ports:
          - containerPort: 8080
---
# node-port service manifest
apiVersion: v1
kind: Service
metadata:
  name: javawebappsvc
spec:
  type: NodePort
  selector:
    app: javawebapp
  ports:
     - port: 80
       targetPort: 8080

...
- In the above scenario we have created the POD manually (it is not recommended)
- If we create the POD then K8S will not provide high availability
_____________________________________________________________________________________________________________________________________________

**lets test it by deleting our pod**
```kubectl delete pod <pod-name>```
**Note:** once pod got delete, k8s not creating another pod and application went down (not accessible)

- If we want to achieve high availability then we should not create pods manually
- We need to use K8S components to create PODS then k8s will provide high availability for our application

**Note:** High Availability means always our application should be accessible

ReplicationController
ReplicationSet
DaemonSet
Deployment
StatefulSets

## What is Replication Controller ?
- It is one of the key feature in k8s
- It is responsible to manage POD lifecycle
- It will make sure given no.of POD replicas are running at any point of time.

**Note:** if any POD got crashed/deleted/dead then Replication Controller will replace it.
- Replication Controller is providing facility to create multiple PODS and it will make sure PODS always exists to run our application.
- Using Replication controller we can achieve High Availability
- Replication Controller and PODS are associated with Labels and Selectors.
```bash
---
#pod manifest configuration
apiVersion: v1
kind: ReplicationController
metadata:
 name: javawebapprc
spec: 
  replicas: 1
  selector:
    app: javawebapp
  template:
    metadata:
      name: javawebapppod
      labels:
        app: javawebapp
    spec:
      containers:
       - name: javawebappcontainer
         image: ashokit/javawebapp
         ports:
          - containerPort: 8080
---
#node-port service manifest
apiVersion: v1
kind: Service
metadata:
  name: javawebappsvc
spec:
  type: NodePort
  selector:
    app: javawebapp
  ports:
     - port: 80
       targetPort: 8080

...
```













