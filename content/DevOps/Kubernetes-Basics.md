---
title: Kubernetes (K8s) Survival Guide
date:
description: Understanding Container Orchestration, Control Plane architecture, Pods, Services, and kubectl mastery.
tags:
  - kubernetes
  - k8s
  - devops
  - orchestration
  - cloud
  - LLM
---

> [!abstract] The "Why"
> Docker manages **containers** on a single machine. Kubernetes manages **clusters** of machines.
> It answers the hard questions: *What if a server crashes? How do I upgrade without downtime? How do I scale from 1 to 1000 instances?*

## 1. The Architecture (The Cluster)
A K8s cluster is split into two parts: The Brain (Control Plane) and the Muscle (Worker Nodes).



### 🧠 The Control Plane (Master Node)
1.  **API Server (`kube-apiserver`):** The gatekeeper. Every command (`kubectl`) goes here. It authenticates and validates requests.
2.  **etcd:** The database. A highly available key-value store that keeps the *entire state* of the cluster. If you lose etcd, you lose the cluster.
3.  **Scheduler:** Decides *where* to put a new Pod based on CPU/RAM availability.
4.  **Controller Manager:** The "loop" that watches the state. If you asked for 3 replicas and one dies, this guy notices and orders a replacement.

### 💪 The Worker Nodes
1.  **Kubelet:** The agent running on every node. It talks to the API Server and says "I am alive" and starts containers.
2.  **Kube-proxy:** Handles the networking. Maintains network rules (IP tables) so traffic finds the right pod.
3.  **Container Runtime:** The engine that actually runs the code (usually `containerd` or Docker).

---

## 2. Core Concepts (The API Objects)

### The Pod (Atomic Unit)
> [!warning] K8s does not run Containers.
> K8s runs **Pods**. A Pod is a wrapper around one (or more) containers.
* Containers in the same Pod share **Localhost** and **Storage volumes**.
* *Analogy:* The Pod is the "Laptop", the Containers are the "Apps" running on it.

### The Deployment (Replica Manager)
You rarely create a Pod directly. You create a **Deployment**.
A Deployment ensures that $N$ copies of your Pod are always running. It handles **Rolling Updates** (updating version v1 to v2 with zero downtime).

### The Service (Networking)
Pods are ephemeral (they die and get new IPs). A **Service** provides a stable, permanent IP address (VIP) to access a group of Pods.
* **ClusterIP:** Internal only (default).
* **NodePort:** Opens a port on the physical server (e.g., 30007).
* **LoadBalancer:** Requests a public IP from the Cloud Provider (AWS/GCP).

---

## 3. The "Declarative" Workflow (YAML)
In K8s, we don't tell the system *what to do* ("Start server"). We tell it *what we want* ("I want 3 servers"). This is **Infrastructure as Code**.

### Production YAML Example
Save this as `app.yaml`:

```yaml
# 1. The Deployment (The Application)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-web-app
spec:
  replicas: 3               # I want 3 copies
  selector:
    matchLabels:
      app: frontend
  template:                 # The Pod Blueprint
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:1.21-alpine
        ports:
        - containerPort: 80
        resources:          # LIMITS ARE CRITICAL
          limits:
            memory: "128Mi"
            cpu: "500m"

---
# 2. The Service (The Entrypoint)
apiVersion: v1
kind: Service
metadata:
  name: frontend-svc
spec:
  selector:
    app: frontend           # Point to the Pods labeled 'frontend'
  ports:
    - protocol: TCP
      port: 80              # Service Port
      targetPort: 80        # Container Port
  type: ClusterIP
````

**Apply it:**

Bash

```
kubectl apply -f app.yaml
```

---

## 4. `kubectl` Cheat Sheet (The Control Stick)

|**Action**|**Command**|
|---|---|
|**Check Nodes**|`kubectl get nodes`|
|**Check Pods**|`kubectl get pods -o wide`|
|**View Logs**|`kubectl logs -f <pod-name>`|
|**Shell Access**|`kubectl exec -it <pod-name> -- sh`|
|**Describe Error**|`kubectl describe pod <pod-name>`|
|**Delete All**|`kubectl delete -f app.yaml`|

> [!tip] Auto-completion
> 
> On Fedora, install bash-completion: echo 'source <(kubectl completion bash)' >> ~/.bashrc

---

## 5. Helm (The Package Manager)

Writing 500 lines of YAML is painful. **Helm** is the "apt/dnf" for Kubernetes.

- instead of writing YAML, you run:
    
    helm install my-db bitnami/postgresql
    
- It uses "Charts" (templates) to manage complex apps.
    

---

## 6. Security (The "Gotchas") 🛡️

1. **Secrets are not Secret:** By default, K8s `Secrets` are just base64 encoded strings stored in etcd. Anyone with API access can decode them. **Fix:** Enable Encryption at Rest in etcd.
    
2. **Network Policies:** By default, all pods can talk to all pods (Flat network). Use `NetworkPolicy` to act as a firewall between your DB and your Frontend.
    
3. **RBAC (Role Based Access Control):** Never give developers `cluster-admin`. Give them `Role` bindings restricted to their specific `Namespace`.
    

---

## Linked Notes

- [[Docker-Ultimate-Guide]] - K8s runs these images.
    
- [[Linux-Kernel-Internals]] - Kube-proxy uses iptables/eBPF.
    
- [[DevOps/Fedora-Workstation]] - My local `minikube` or `kind` setup.