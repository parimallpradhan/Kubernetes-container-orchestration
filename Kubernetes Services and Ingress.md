# Kubernetes Services and Ingress

Start your class with this question:

> **"We created 3 Nginx Pods using a Deployment. Can a user reliably access those Pods directly?"**

The answer is **No**.

Why?

```text
Deployment
    |
    +---- Pod 1 → IP: 10.244.1.5
    +---- Pod 2 → IP: 10.244.2.7
    +---- Pod 3 → IP: 10.244.1.9
```

Pod IPs are not a good stable endpoint for clients.

A Pod can be:

* deleted
* recreated
* moved to another node
* given a different IP

So we need **Service**.

---

# 1. What is a Kubernetes Service?

Tell students:

> **A Kubernetes Service provides a stable network endpoint for accessing a group of Pods.**

Architecture:

```text
                    Service
                       |
             +---------+---------+
             |         |         |
             v         v         v
           Pod 1     Pod 2     Pod 3
           Nginx     Nginx     Nginx
```

The Service selects Pods using labels.

For example:

```yaml
selector:
  app: nginx
```

And the Pods have:

```yaml
labels:
  app: nginx
```

So:

```text
Service selector
     |
 app=nginx
     |
 +---+---+---+
 |   |   |   |
Pod Pod Pod Pod
```

---

# 2. Why do we need Service?

Show this scenario.

Initially:

```text
Client
  |
  v
Pod
```

Pod crashes:

```text
Client
  |
  X
Pod
```

Deployment creates a new Pod:

```text
Client
  |
  X
Old Pod

New Pod
IP = different
```

If clients directly depended on the Pod IP, connectivity becomes problematic.

With Service:

```text
Client
  |
  v
Service
  |
  +---- Pod 1
  +---- Pod 2
  +---- Pod 3
```

The Service provides the stable endpoint while the backend Pods can change.

---

# 3. Service and Deployment relationship

This is extremely important.

```text
                Deployment
                     |
                ReplicaSet
                     |
          +----------+----------+
          |          |          |
         Pod        Pod        Pod
          |          |          |
       app=web    app=web    app=web
          \          |          /
           \         |         /
            +--------+--------+
                     |
                  Service
               selector:
                 app=web
```

The Service does **not normally select the Deployment itself**.

It selects Pods through labels.

---

# 4. Create a Service

Suppose your Deployment has:

```yaml
labels:
  app: nginx
```

Create:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-service

spec:
  selector:
    app: nginx

  ports:
    - port: 80
      targetPort: 80

  type: ClusterIP
```

Apply:

```bash
kubectl apply -f service.yaml
```

Check:

```bash
kubectl get services
```

---

# 5. Explain `port` vs `targetPort`

This is one of the most common student questions.

```yaml
ports:
  - port: 80
    targetPort: 80
```

Think:

```text
Client
   |
   | Service port 80
   v
Service
   |
   | targetPort 80
   v
Pod
   |
Nginx container
   |
Port 80
```

### `port`

The port exposed by the **Service**.

### `targetPort`

The port on the **backend Pod/container** that receives the traffic.

They don't have to be the same.

Example:

```yaml
port: 8080
targetPort: 80
```

means:

```text
Client → Service:8080 → Pod:80
```

---

# 6. Types of Kubernetes Services

Teach these four:

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

For your students, focus heavily on the first three.

---

# 7. ClusterIP

This is the default Service type.

```yaml
type: ClusterIP
```

Architecture:

```text
Pod A
Pod B
Pod C
  \ | /
   \|/
 Service
 ClusterIP
    |
    X
External Internet
```

It's primarily for **internal cluster communication**.

Example:

```text
Frontend Pod
     |
     v
Backend Service
     |
 +---+---+
 |       |
Pod     Pod
```

The frontend doesn't need to know the individual backend Pod IPs.

---

# 8. NodePort

Now ask:

> "What if I want to access my application from outside the cluster?"

Use:

```yaml
type: NodePort
```

Architecture:

```text
                 Browser
                    |
                    v
              NodeIP:30080
                    |
                NodePort
                    |
                Service
                    |
          +---------+---------+
          |         |         |
         Pod       Pod       Pod
```

Example:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-service

spec:
  type: NodePort

  selector:
    app: nginx

  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

Then:

```text
http://<NodeIP>:30080
```

can reach the Service, subject to your cluster/network configuration.

---

# 9. NodePort range

Typically Kubernetes NodePort uses:

```text
30000–32767
```

unless the cluster is configured differently.

So students might see:

```text
30080
30081
30082
```

---

# 10. LoadBalancer

For cloud environments:

```yaml
type: LoadBalancer
```

Architecture:

```text
                 Internet
                    |
                    v
             Cloud Load Balancer
                    |
                 Service
                    |
          +---------+---------+
          |         |         |
         Pod       Pod       Pod
```

In a cloud environment such as AWS, Azure or Google Cloud, the Kubernetes/cloud integration can provision an external load-balancing resource.

This is where you can introduce:

```text
AWS → Load Balancer
Azure → Load Balancer
GCP → Load Balancer
```

---

# 11. Compare Service Types

| Type         | Main Purpose                         | External Access    |
| ------------ | ------------------------------------ | ------------------ |
| ClusterIP    | Internal communication               | ❌                  |
| NodePort     | Basic external access                | ✅                  |
| LoadBalancer | Cloud external access                | ✅                  |
| ExternalName | DNS-based external service reference | Different use case |

Tell students:

> **ClusterIP is the default Service type.**

---

# 12. Now introduce Ingress

Once students understand Service, ask:

> "Suppose I have 20 applications. Should I create a separate external LoadBalancer for every application?"

Usually, you want a more centralized HTTP/HTTPS entry point.

That's where **Ingress** comes in.

Architecture:

```text
                    Internet
                       |
                       v
                    Ingress
                       |
          +------------+------------+
          |            |            |
          v            v            v
       Service      Service      Service
          |            |            |
       Frontend      Backend     Payment
```

---

# 13. What is Ingress?

Tell students:

> **Ingress is a Kubernetes API object that defines HTTP/HTTPS routing rules from outside the cluster to Services.**

For example:

```text
example.com
     |
     v
   Ingress
     |
 +---+---+----------------+
 |       |                |
 /       /api             /payment
 |       |                |
Frontend Backend          Payment
Service  Service          Service
```

---

# 14. Ingress is NOT the Ingress Controller

This distinction is extremely important.

Students often say:

> "Ingress is NGINX."

That's incorrect.

Explain:

### Ingress

Defines the routing rules.

### Ingress Controller

Actually implements those rules.

Architecture:

```text
              Internet
                  |
                  v
          Ingress Controller
                  |
             reads Ingress
                  |
        +---------+---------+
        |         |         |
     Service   Service   Service
```

Examples of Ingress Controllers include:

* NGINX Ingress Controller
* Traefik
* HAProxy
* Kong
* Cloud-provider-specific controllers

---

# 15. Ingress example

Suppose we have:

```text
Frontend Service
Backend Service
Payment Service
```

We want:

```text
example.com/
       ↓
Frontend

example.com/api
       ↓
Backend

example.com/payment
       ↓
Payment
```

Ingress can define routing rules for this.

Conceptually:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress

metadata:
  name: ecommerce-ingress

spec:
  rules:
    - host: ecommerce.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80

          - path: /api
            pathType: Prefix
            backend:
              service:
                name: backend-service
                port:
                  number: 8080
```

Explain:

```text
Host
 |
ecommerce.example.com
 |
 +-- /       → frontend-service
 |
 +-- /api    → backend-service
```

---

# 16. Service vs Ingress

This is the key comparison.

| Service                              | Ingress                                 |
| ------------------------------------ | --------------------------------------- |
| Provides stable access to Pods       | Provides HTTP/HTTPS routing to Services |
| Works at service/network level       | Primarily HTTP/HTTPS                    |
| Selects Pods using labels            | Routes requests using host/path rules   |
| ClusterIP/NodePort/LoadBalancer etc. | Routing layer                           |
| Doesn't replace Ingress              | Uses Services as backends               |

Simple mental model:

```text
Ingress
   |
   v
Service
   |
   v
Pods
   |
   v
Containers
```

---

# 17. Real E-Commerce Example

This is especially useful for your students.

```text
                       Users
                         |
                         v
                    Ingress
                         |
        +----------------+----------------+
        |                |                |
        v                v                v
   Frontend Service  Backend Service  Payment Service
        |                |                |
      Pods             Pods             Pods
        |                |                |
    React/HTML        Spring Boot      Payment App
                         |
                         v
                    Database
```

Routing:

```text
shop.example.com/
        ↓
Frontend

shop.example.com/api
        ↓
Backend

shop.example.com/payment
        ↓
Payment
```

This is a very realistic architecture for students.

---

# 18. Important modern point: Gateway API

Since you're teaching current Kubernetes, briefly mention:

> **Ingress is still widely used, but Kubernetes Gateway API is the newer, more expressive API model for traffic routing.**

Don't teach Gateway API in detail yet.

Just tell students:

```text
Service
   ↓
Ingress
   ↓
Gateway API
```

Gateway API is worth learning later, especially for advanced Kubernetes/networking topics.

---

# 19. Hands-on sequence I recommend

### Lab 1 — Deployment

```bash
kubectl apply -f deployment.yaml
```

### Lab 2 — ClusterIP

```bash
kubectl apply -f service.yaml
```

Check:

```bash
kubectl get svc
```

### Lab 3 — NodePort

Change:

```yaml
type: NodePort
```

Then:

```bash
kubectl apply -f service.yaml
```

Check:

```bash
kubectl get svc
```

Access:

```text
http://NodeIP:NodePort
```

### Lab 4 — Ingress

Install/configure an Ingress Controller appropriate for your cluster.

Then create:

```text
ingress.yaml
```

Apply:

```bash
kubectl apply -f ingress.yaml
```

Check:

```bash
kubectl get ingress
```

Then test your hostname/path routing.

---

# 20. Final architecture students must remember

```text
                         Internet
                            |
                            v
                         Ingress
                    HTTP / HTTPS routing
                            |
             +--------------+--------------+
             |              |              |
             v              v              v
         Service         Service        Service
             |              |              |
          +--+--+        +--+--+        +--+--+
          |     |        |     |        |     |
         Pod   Pod      Pod   Pod      Pod   Pod
          |     |        |     |        |     |
       Container       Container       Container
```

And inside the cluster:

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

### ⭐ golden rule

> **Deployment manages Pods.**
>
> **Service provides stable networking to Pods.**
>
> **Ingress provides HTTP/HTTPS routing to Services.**

