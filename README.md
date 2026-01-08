<div align="center">
<h1>🚀 k8s HPA</h1>
<p><strong>Built with ❤️ by <a href="https://github.com/atulkamble">Atul Kamble</a></strong></p>

<p>
<a href="https://codespaces.new/atulkamble/template.git">
<img src="https://github.com/codespaces/badge.svg" alt="Open in GitHub Codespaces" />
</a>
<a href="https://vscode.dev/github/atulkamble/template">
<img src="https://img.shields.io/badge/Open%20with-VS%20Code-007ACC?logo=visualstudiocode&style=for-the-badge" alt="Open with VS Code" />
</a>
<a href="https://vscode.dev/redirect?url=vscode://ms-vscode-remote.remote-containers/cloneInVolume?url=https://github.com/atulkamble/template">
<img src="https://img.shields.io/badge/Dev%20Containers-Ready-blue?logo=docker&style=for-the-badge" />
</a>
<a href="https://desktop.github.com/">
<img src="https://img.shields.io/badge/GitHub-Desktop-6f42c1?logo=github&style=for-the-badge" />
</a>
</p>

<p>
<a href="https://github.com/atulkamble">
<img src="https://img.shields.io/badge/GitHub-atulkamble-181717?logo=github&style=flat-square" />
</a>
<a href="https://www.linkedin.com/in/atuljkamble/">
<img src="https://img.shields.io/badge/LinkedIn-atuljkamble-0A66C2?logo=linkedin&style=flat-square" />
</a>
<a href="https://x.com/atul_kamble">
<img src="https://img.shields.io/badge/X-@atul_kamble-000000?logo=x&style=flat-square" />
</a>
</p>

<strong>Version 1.0.0</strong> | <strong>Last Updated:</strong> January 2026
</div>

## 🚀 Kubernetes HPA (Horizontal Pod Autoscaler)

Below is a **complete, practical guide** covering **concepts, architecture, YAML codes, metrics, and hands-on practice labs** — perfect for **AKS / EKS / GKE / Minikube** learning and production prep.

---

![Image](https://cdn.shortpixel.ai/spai/q_lossless%2Bret_img%2Bto_webp/www.apptio.com/wp-content/uploads/hpa-overview.png)

![Image](https://www.nops.io/wp-content/uploads/2023/06/Horizontal-Pod-Autoscaler-HPA-1024x671.png)

![Image](https://miro.medium.com/v2/resize%3Afit%3A1400/1%2ALgM4NNphVcyDesY_lo8OkA.png)

![Image](https://miro.medium.com/v2/resize%3Afit%3A1400/1%2Ag_CCHx8E6ZNlRXaTBtfuuQ.png)

---

## 1️⃣ What is Kubernetes HPA?

**HPA (Horizontal Pod Autoscaler)** automatically **scales Pods up or down** based on:

* CPU usage
* Memory usage
* Custom metrics (Prometheus, external APIs)

🔁 It adjusts **number of replicas**, **not pod size**.

---

## 2️⃣ Why HPA is Needed (Real Problems)

| Without HPA          | With HPA           |
| -------------------- | ------------------ |
| Manual scaling       | Automatic scaling  |
| Over-provisioning    | Cost optimized     |
| App crash on load    | Load-aware scaling |
| Poor user experience | Stable performance |

---

## 3️⃣ HPA Architecture & Flow

### 🔄 How it Works

1. **Metrics Server** collects CPU/memory
2. **HPA Controller** checks metrics every 15s
3. Desired replicas calculated
4. **Deployment scaled automatically**

### 🧮 Formula (CPU example)

```
desiredReplicas = currentReplicas × ( currentCPU / targetCPU )
```

---

## 4️⃣ Prerequisites (IMPORTANT)

### ✅ Metrics Server (Mandatory)

```bash
kubectl get pods -n kube-system | grep metrics
```

If not installed:

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

Verify:

```bash
kubectl top nodes
kubectl top pods
```

---

## 5️⃣ Basic Deployment (Sample App)

### 📄 deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hpa-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hpa-demo
  template:
    metadata:
      labels:
        app: hpa-demo
    spec:
      containers:
      - name: nginx
        image: nginx
        resources:
          requests:
            cpu: "100m"
          limits:
            cpu: "200m"
        ports:
        - containerPort: 80
```

```bash
kubectl apply -f deployment.yaml
```

---

## 6️⃣ HPA – CPU Based Scaling (MOST COMMON)

### 📄 hpa-cpu.yaml

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-demo
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: hpa-demo
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

Apply:

```bash
kubectl apply -f hpa-cpu.yaml
kubectl get hpa
```

---

## 7️⃣ Generate Load (Practice Lab 🔥)

```bash
kubectl run load-generator \
  --image=busybox \
  --restart=Never -- /bin/sh -c \
  "while true; do wget -q -O- http://hpa-demo; done"
```

Watch scaling:

```bash
kubectl get hpa -w
kubectl get pods -w
```

✅ Pods will scale from **1 → 5**

---

## 8️⃣ Memory-Based HPA

### 📄 hpa-memory.yaml

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-memory
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: hpa-demo
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: memory
      target:
        type: AverageValue
        averageValue: 200Mi
```

---

## 9️⃣ HPA Using kubectl (Quick Way)

```bash
kubectl autoscale deployment hpa-demo \
  --cpu-percent=50 \
  --min=1 \
  --max=5
```

---

## 🔟 HPA Status & Debug Commands

```bash
kubectl describe hpa hpa-demo
kubectl get events
kubectl top pods
kubectl get deployment hpa-demo -o wide
```

---

## 1️⃣1️⃣ Common HPA Issues & Fixes

| Issue             | Reason                 | Fix                      |
| ----------------- | ---------------------- | ------------------------ |
| HPA not scaling   | Metrics server missing | Install metrics server   |
| CPU always 0%     | No resource requests   | Add `resources.requests` |
| Sudden scale down | Aggressive config      | Tune minReplicas         |
| Flapping          | Traffic spikes         | Use stabilization        |

---

## 1️⃣2️⃣ Production Best Practices ⭐

✅ Always define:

```yaml
resources:
  requests:
  limits:
```

✅ Use:

* `autoscaling/v2`
* Proper `minReplicas`
* PodDisruptionBudgets
* HPA + Cluster Autoscaler together

---

## 1️⃣3️⃣ Real-World Production Combo

```
User Traffic
   ↓
Ingress
   ↓
Service
   ↓
Deployment
   ↓
HPA (Pods)
   ↓
Cluster Autoscaler (Nodes)
```

---

## 1️⃣4️⃣ Practice Tasks (Interview + Lab)

✔ Scale based on CPU
✔ Scale based on Memory
✔ Simulate heavy load
✔ Observe scaling behavior
✔ Debug non-scaling HPA
✔ Combine with AKS node autoscaling

---
