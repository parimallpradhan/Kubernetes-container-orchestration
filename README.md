# ☸️ Kubernetes Training & Hands-On Lab

---

## 🎯 Learning Objectives

By completing this repository, you will learn how to:

* Understand Kubernetes and container orchestration
* Explain why Kubernetes is used
* Compare Docker Swarm and Kubernetes
* Understand Kubernetes cluster architecture
* Understand Control Plane and Worker Node components
* Deploy a Kubernetes cluster using `kubeadm`
* Understand alternative Kubernetes deployment methods
* Write Kubernetes YAML manifests
* Create and manage Pods
* Create Deployments and ReplicaSets
* Scale applications
* Perform rolling updates and rollbacks
* Create Kubernetes Services
* Understand ClusterIP, NodePort and LoadBalancer
* Configure HTTP/HTTPS routing using Ingress
* Troubleshoot common Kubernetes issues
* Operate Kubernetes using `kubectl`

---

# 📚 Course Contents

## 01. What is Kubernetes?

Learn:

* What is Kubernetes?
* What is container orchestration?
* Why Kubernetes is required
* Problems with managing containers manually
* Kubernetes use cases
* Kubernetes advantages
* Kubernetes ecosystem

### Key Concept

```text
Docker
   ↓
Containers
   ↓
Container Orchestration
   ↓
Kubernetes
```

---

## 02. Docker Swarm vs Kubernetes

Learn:

* What is Docker Swarm?
* Why Docker Swarm was created
* Swarm architecture
* Kubernetes vs Docker Swarm
* Scaling
* Self-healing
* Rolling updates
* Rollbacks
* Networking
* Ecosystem
* Production use cases

### Key Concept

```text
Docker Swarm
    ↓
Simple and Docker-centric

Kubernetes
    ↓
Extensible and cloud-native
```

---

## 03. Kubernetes Architecture


### Control Plane

* kube-apiserver
* etcd
* kube-scheduler
* kube-controller-manager

### Worker Node

* kubelet
* kube-proxy
* Container Runtime



### Architecture

```text
                  Kubernetes Cluster
                         |
             +-----------+-----------+
             |                       |
        Control Plane           Worker Nodes
             |                       |
       +-----+------+          +-----+-----+
       |     |      |          |           |
    API   Scheduler etcd     kubelet   kube-proxy
    Server                    |
       |                       |
 Controller                 Pod
 Manager                      |
                          Container
```

### Request Flow

```text
kubectl
   ↓
API Server
   ↓
Scheduler / Controllers
   ↓
Worker Node
   ↓
kubelet
   ↓
Container Runtime
   ↓
Container
```



# 1. Real-World Problem

> "Suppose you have 20 Docker containers running your application. What happens if one container crashes?"

With plain Docker:

```text
Docker Host
   |
   +--- Container 1
   +--- Container 2
   +--- Container 3
   +--- Container 4
```

If Container 3 crashes:

```text
Container 3 ❌
```

Someone needs to:

* Detect the failure
* Restart it
* Maintain the required number of containers
* Distribute traffic
* Deploy new versions
* Roll back bad versions
* Scale applications

Doing this manually becomes difficult.

### This is where Kubernetes comes in.

---

# 2. What is Kubernetes?



> **Kubernetes is an open-source container orchestration platform used to deploy, manage, scale, and maintain containerized applications.**

Break it down:

```text
Container
   ↓
Kubernetes
   ↓
Deploy
Manage
Scale
Monitor
Recover
```


Instead explain:

```text
Developer
   ↓
Container Image
   ↓
Kubernetes
   ↓
Runs and manages containers
```

---

# 3. Why Do We Need Kubernetes?

Suppose an application has:

```text
Frontend
Backend
Database
Payment Service
Notification Service
```

And you have:

```text
10 containers
```

Then production grows:

```text
10 containers
      ↓
100 containers
      ↓
500 containers
```

Managing them manually becomes difficult.

Kubernetes helps with:

### 1. Deployment

Deploy applications automatically.

### 2. Scaling

```text
2 Pods → 5 Pods → 10 Pods
```

### 3. Self-healing

If a Pod crashes:

```text
Pod ❌
  ↓
Kubernetes detects it
  ↓
New Pod ✅
```

### 4. Load balancing

Traffic can be distributed across multiple Pods.

### 5. Rolling updates

Deploy a new application version without taking the application completely offline.

### 6. Rollback

If the new version has a problem:

```text
v2 ❌
 ↓
Rollback
 ↓
v1 ✅
```

---

# 4. Kubernetes Architecture

Kubernetes has two major parts:

```text
             Kubernetes Cluster
                    |
          +---------+---------+
          |                   |
          ↓                   ↓
     Control Plane       Worker Nodes
                            |
                    +-------+-------+
                    |       |       |
                   Pod     Pod     Pod
```

### Control Plane

The **brain** of Kubernetes.

### Worker Node

Where application workloads run.

---

# 5. Control Plane Components


```text
             CONTROL PLANE
                  |
     +------------+------------+
     |            |            |
     ↓            ↓            ↓
 API Server   Scheduler    Controller
                              Manager
                  |
                  ↓
                 etcd
```

## 5.1 kube-apiserver

This is the **entry point to Kubernetes**.

When you execute:

```bash
kubectl get pods
```

the request goes to:

```text
kubectl
   ↓
kube-apiserver
```

The API Server communicates with the Kubernetes cluster.

> **API Server = front door of Kubernetes**

---

# 5.2. etcd

`etcd` stores Kubernetes cluster information.

For example:

```text
Pod information
Deployment information
Service information
Node information
Configuration/state
```

Think of it as:

> **etcd = Kubernetes cluster database**

Important:

```text
kubectl
   ↓
API Server
   ↓
etcd
```



---

# 5.3. Scheduler

Suppose you create a Pod:

```yaml
kind: Pod
```

Kubernetes needs to decide:

> "Which worker node should run this Pod?"

That's the Scheduler's job.

```text
New Pod
   ↓
Scheduler
   ↓
Worker Node 1
```

If Node 1 doesn't have enough resources:

```text
Node 1 ❌
Node 2 ✅
   ↓
Pod → Node 2
```

Simple definition:

> **Scheduler decides where a Pod should run.**

---

# 5.4. Controller Manager

Controllers continuously check whether the **actual state** matches the **desired state**.

Example:

You want:

```text
3 Pods
```

But currently:

```text
Pod 1 ✅
Pod 2 ✅
Pod 3 ❌
```

Controller notices:

```text
Desired = 3
Actual = 2
```

So Kubernetes creates another Pod.

```text
Pod 1 ✅
Pod 2 ✅
Pod 3 ❌
      ↓
Controller
      ↓
New Pod ✅
```

This is one of the most important concepts in Kubernetes:

> **Desired State vs Actual State**

---

# 6. Worker Node Components

Now move to the Worker Node.

```text
             Worker Node
                  |
        +---------+---------+
        |                   |
        ↓                   ↓
     kubelet           kube-proxy
        |
        ↓
 Container Runtime
        |
        ↓
    Containers
```

## 6.1 kubelet

`kubelet` is the agent running on every worker node.

Its job is basically:

> "Make sure the Pods assigned to this node are running correctly."

```text
Control Plane
      |
      ↓
   kubelet
      |
      ↓
    Pod
```

---

# 6.2 Container Runtime

Kubernetes needs a container runtime to actually run containers.

Modern Kubernetes commonly uses:

```text
containerd
CRI-O
```


Explain:

```text
Kubernetes
    ↓
kubelet
    ↓
Container Runtime
    ↓
Container
```

---

# 6.3. kube-proxy

`kube-proxy` helps implement networking rules for Services on nodes.


> **kube-proxy helps Kubernetes route network traffic to the appropriate Pods.**


---

# Complete Architecture


```text
                         Kubernetes Cluster
                                |
             +------------------+------------------+
             |                                     |
             ↓                                     ↓
       CONTROL PLANE                          WORKER NODE
             |                                     |
     +-------+-------+                       +-----+------+
     |       |       |                       |            |
     ↓       ↓       ↓                       ↓            ↓
 API     Scheduler Controller             kubelet    kube-proxy
Server            Manager                    |
     |                                      ↓
     +-----> etcd                    Container Runtime
                                                |
                                                ↓
                                           +----+----+
                                           |         |
                                           ↓         ↓
                                         Pod 1     Pod 2
                                           |         |
                                        Container Container
```



---

# 7. Where Does Docker Fit?


> "Is Kubernetes a replacement for Docker?"

Explain:

```text
Docker
   ↓
Build container image
   ↓
Dockerfile
   ↓
Docker Image
```

Then:

```text
Kubernetes
   ↓
Deploy/manage containers
   ↓
Pods
```

Example:

```text
Developer
    |
    ↓
Dockerfile
    |
    ↓
Docker Build
    |
    ↓
Docker Image
    |
    ↓
Container Registry
    |
    ↓
Kubernetes
    |
    ↓
Pod
    |
    ↓
Container
```

This connects nicely with what students have already learned about Docker.

---

#What is a Kubernetes Cluster?

A cluster is a collection of machines/nodes managed by Kubernetes.

```text
Kubernetes Cluster
       |
       +--- Control Plane
       |
       +--- Worker Node 1
       |
       +--- Worker Node 2
       |
       +--- Worker Node 3
```

Applications normally run on the Worker Nodes.

---

# 8.What is a Pod?



> **Pod is the smallest deployable unit in Kubernetes.**

Basic structure:

```text
Pod
 |
 +--- Container
```

A Pod can contain one or more containers:

```text
Pod
 |
 +--- Container 1
 |
 +--- Container 2
```



---

# Kubernetes Hands-On

Since you're using **Killercoda**, this is perfect for students.

Start a Kubernetes playground.

Then run:

```bash
kubectl version --client
```

This tells you the `kubectl` client version.

Then:

```bash
kubectl cluster-info
```

This checks the Kubernetes cluster information.

Then:

```bash
kubectl get nodes
```

Students should see something similar to:

```text
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   ...   v1.xx.x
node01         Ready    <none>          ...   v1.xx.x
```

Explain:

```text
NAME       → Node name
STATUS     → Ready/NotReady
ROLES      → Control Plane/Worker
AGE        → How long node exists
VERSION    → Kubernetes version
```

---

# Important kubectl Commands

Don't overload students.

Teach only these:

```bash
kubectl version --client
```

```bash
kubectl cluster-info
```

```bash
kubectl get nodes
```

```bash
kubectl get pods
```

```bash
kubectl get namespaces
```

```bash
kubectl get all
```

And:

```bash
kubectl describe node <node-name>
```

Explain the pattern:

```text
kubectl
   |
   +--- get
   +--- describe
   +--- create
   +--- apply
   +--- delete
```



---

# Exercise

Give students this exercise at the end:

### Task

**1. Check Kubernetes version**

```bash
kubectl version --client
```

**2. Check cluster**

```bash
kubectl cluster-info
```

**3. List nodes**

```bash
kubectl get nodes
```

**4. Get detailed information about a node**

```bash
kubectl describe node <node-name>
```

**5. List namespaces**

```bash
kubectl get namespaces
```

**6. List Pods in all namespaces**

```bash
kubectl get pods -A
```



## 🎯 Diagram to remember

Give them this one diagram to remember:

```text
                    KUBERNETES
                        |
                 +------+------+
                 |             |
                 ↓             ↓
           CONTROL PLANE    WORKER NODE
                 |             |
       +---------+---------+   |
       |         |         |   |
       ↓         ↓         ↓   ↓
   API Server Scheduler Controller
       |                   Manager
       ↓
      etcd                 kubelet
                              |
                              ↓
                        Container Runtime
                              |
                              ↓
                             Pod
                              |
                              ↓
                          Container
```
---

## Kubernetes Deployment Methods

Different ways to deploy Kubernetes:

### Local / Learning

* Minikube
* kind
* Docker Desktop Kubernetes

### Self-Managed

* kubeadm
* Kubespray
* RKE2

### Managed Cloud

* Amazon EKS
* Azure AKS
* Google GKE

### Recommended Learning Path

```text
Minikube / kind
       ↓
Learn kubectl
       ↓
kubeadm
       ↓
Understand Kubernetes internals
       ↓
EKS / AKS / GKE
       ↓
Cloud Kubernetes
```

---

# Kubernetes Cluster Deployment Using kubeadm

Build a Kubernetes cluster using:

* Ubuntu
* containerd
* kubeadm
* kubelet
* kubectl
* CNI networking

### Cluster Architecture

```text
              Kubernetes Cluster
                     |
          +----------+----------+
          |                     |
    Control Plane           Worker Node
          |                     |
       kubeadm               kubelet
       kubelet              containerd
       kubectl
       etcd
       API Server
       Scheduler
```

### High-Level Installation Flow

```text
Prepare Linux Nodes
       ↓
Disable Swap
       ↓
Install containerd
       ↓
Configure containerd
       ↓
Install kubeadm
       ↓
Install kubelet
       ↓
Install kubectl
       ↓
kubeadm init
       ↓
Configure kubectl
       ↓
Install CNI
       ↓
kubeadm join
       ↓
Worker Node Ready
       ↓
Verify Cluster
```

### Verification

```bash
kubectl get nodes
kubectl get pods -A
kubectl cluster-info
```

---

# Kubernetes YAML

Learn how Kubernetes resources are represented using YAML manifests.

### Basic Structure

```yaml
apiVersion:
kind:
metadata:
spec:
```

### Important Concepts

* YAML indentation
* Key-value pairs
* Lists
* Comments
* `apiVersion`
* `kind`
* `metadata`
* `spec`
* Labels
* Selectors
* Desired State
* Declarative configuration

### Apply a Manifest

```bash
kubectl apply -f file.yaml
```

### Validate

```bash
kubectl apply --dry-run=client -f file.yaml
```

### Explore Kubernetes API Resources

```bash
kubectl explain deployment
kubectl explain deployment.spec
```

---


---

# Kubernetes Objects

Learn how Kubernetes resources are represented using YAML manifests.

### Basic Structure

```yaml
apiVersion:
kind:
metadata:
spec:
```

### Important Concepts

* YAML indentation
* Key-value pairs
* Lists
* Comments
* `apiVersion`
* `kind`
* `metadata`
* `spec`
* Labels
* Selectors
* Desired State
* Declarative configuration

### Apply a Manifest

```bash
kubectl apply -f file.yaml
```

### Validate

```bash
kubectl apply --dry-run=client -f file.yaml
```

### Explore Kubernetes API Resources

```bash
kubectl explain deployment
kubectl explain deployment.spec
```

---




# Kubernetes Deployment

A Deployment manages application Pods and maintains the desired number of replicas.

### Deployment Architecture

```text
Deployment
     |
 ReplicaSet
     |
 +---+---+---+
 |   |   |   |
Pod Pod Pod Pod
```

### Example

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deployment

spec:
  replicas: 3

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
```

### Deploy

```bash
kubectl apply -f deployment.yaml
```

### Verify

```bash
kubectl get deployment
kubectl get replicasets
kubectl get pods
kubectl get pods -o wide
```

### Scaling

```bash
kubectl scale deployment nginx-deployment --replicas=5
```

### Rolling Update

```bash
kubectl rollout status deployment/nginx-deployment
```

### Rollout History

```bash
kubectl rollout history deployment/nginx-deployment
```

### Rollback

```bash
kubectl rollout undo deployment/nginx-deployment
```

---

# 08. Kubernetes Services

Pods are ephemeral and their IP addresses can change.

A Service provides a stable network endpoint for accessing Pods.

### Architecture

```text
             Service
                |
       +--------+--------+
       |        |        |
      Pod      Pod      Pod
```

### Service Types

```text
Service
   |
   +-- ClusterIP
   |
   +-- NodePort
   |
   +-- LoadBalancer
   |
   +-- ExternalName
```

### ClusterIP

Used primarily for internal communication.

```text
Frontend
   |
   v
Backend Service
   |
 +--+--+
 |     |
Pod   Pod
```

### NodePort

Provides access through a port on a node.

```text
Browser
   |
NodeIP:NodePort
   |
Service
   |
Pods
```

### LoadBalancer

Commonly used with cloud environments.

```text
Internet
   |
Cloud Load Balancer
   |
Service
   |
Pods
```

---

# 09. Kubernetes Ingress

Ingress provides HTTP/HTTPS routing from outside the cluster to Kubernetes Services.

### Architecture

```text
                 Internet
                    |
                    v
                 Ingress
                    |
        +-----------+-----------+
        |           |           |
        v           v           v
    Frontend     Backend     Payment
    Service      Service      Service
        |           |           |
       Pods        Pods        Pods
```

### Example Routing

```text
example.com/
       ↓
Frontend Service

example.com/api
       ↓
Backend Service

example.com/payment
       ↓
Payment Service
```

### Important Concept

```text
Ingress
   ↓
Service
   ↓
Pod
   ↓
Container
   ↓
Application
```

> **Ingress is the API object containing routing rules. An Ingress Controller implements those rules.**

---

# 🧪 Hands-On Labs

## Lab 1 — Create a Kubernetes Cluster

* Prepare Control Plane
* Prepare Worker Node
* Install containerd
* Install kubeadm
* Install kubelet
* Install kubectl
* Initialize Control Plane
* Install CNI
* Join Worker Node

---

## Lab 2 — Create a Pod

```bash
kubectl apply -f pod.yaml
kubectl get pods
kubectl describe pod nginx-pod
kubectl logs nginx-pod
```

---

## Lab 3 — Create a Deployment

* Create Deployment YAML
* Deploy Nginx
* Create 3 replicas
* Verify ReplicaSet
* Verify Pods
* Delete a Pod
* Observe self-healing
* Scale to 5 replicas

---

## Lab 4 — Rolling Update & Rollback

* Deploy application version 1
* Update application version
* Observe rolling update
* Check rollout history
* Roll back to previous version

---

## Lab 5 — Kubernetes Service

* Create ClusterIP Service
* Test internal communication
* Create NodePort Service
* Access application externally
* Understand LoadBalancer

---

## Lab 6 — Kubernetes Ingress

* Install/configure an Ingress Controller
* Create Ingress resource
* Configure host-based routing
* Configure path-based routing
* Test HTTP/HTTPS access

---

# 🛠️ Essential kubectl Commands

```bash
kubectl get nodes
kubectl get pods
kubectl get pods -A
kubectl get deployments
kubectl get replicasets
kubectl get services
kubectl get ingress
```

### Detailed Information

```bash
kubectl describe pod <pod-name>
kubectl describe deployment <deployment-name>
kubectl describe service <service-name>
kubectl describe node <node-name>
```

### Apply / Delete

```bash
kubectl apply -f file.yaml
kubectl delete -f file.yaml
```

### Troubleshooting

```bash
kubectl get events
kubectl logs <pod-name>
kubectl describe pod <pod-name>
kubectl get pods -o wide
```

---

# 🎯 Learning Roadmap

```text
Kubernetes Fundamentals
        ↓
Docker Swarm vs Kubernetes
        ↓
Kubernetes Architecture
        ↓
Deployment Methods
        ↓
kubeadm Cluster
        ↓
YAML
        ↓
Pods
        ↓
Deployments
        ↓
ReplicaSets
        ↓
Services
        ↓
Ingress
        ↓
ConfigMaps & Secrets
        ↓
Volumes & Persistent Storage
        ↓
Namespaces
        ↓
RBAC & Security
        ↓
Resource Management
        ↓
Autoscaling
        ↓
Helm
        ↓
Monitoring & Logging
        ↓
CI/CD
        ↓
GitOps
        ↓
Production Kubernetes
```

---

# 📌 Repository Purpose

This repository is intended for **hands-on Kubernetes learning, DevOps training, demonstrations, and practical reference**.

Each topic contains:

* 📖 Concepts
* 🏗️ Architecture
* 📝 YAML manifests
* 💻 Commands
* 🧪 Hands-on exercises
* 🔧 Troubleshooting
* 🎯 Interview concepts

---

## ⭐ Recommended Learning Approach

Do not simply copy and execute commands.

For every Kubernetes topic, understand:

```text
What?
 ↓
Why?
 ↓
Architecture
 ↓
YAML
 ↓
Command
 ↓
Hands-on
 ↓
Troubleshooting
 ↓
Real-world use case
```

---

## 🚀 Future Topics

* ConfigMap
* Secret
* Namespaces
* Resource Requests & Limits
* Probes
* Volumes
* PersistentVolume
* PersistentVolumeClaim
* StatefulSet
* DaemonSet
* Job & CronJob
* RBAC
* Service Accounts
* Network Policies
* HPA
* Helm
* Monitoring with Prometheus & Grafana
* Logging with EFK/ELK
* Kubernetes Security
* CI/CD with Jenkins
* GitOps with Argo CD
* AWS EKS
* Production Kubernetes Architecture

---

## 👨‍💻 Author

**Kubernetes / DevOps Training Repository**

Built for practical learning and hands-on Kubernetes administration.
