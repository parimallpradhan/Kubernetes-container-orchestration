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

Learn:

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

---

## 04. Kubernetes Deployment Methods

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

# 05. Kubernetes Cluster Deployment Using kubeadm

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

# 06. Kubernetes YAML

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

# 07. Kubernetes Deployment

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
