# Kubernetes (K8s)

## المحتويات
- [ما هو Kubernetes؟](#ما-هو-kubernetes)
- [أسئلة المقابلات](Kubernetes-Interview-Questions.md)
- [المقارنة مع Docker و OpenShift](../Comparison.md)

---

## ما هو Kubernetes؟

**Kubernetes** (K8s) هو منصة لإدارة وتنظيم الـ Containers على نطاق واسع (Container Orchestration).

### المفاهيم الأساسية:

#### 1. **Pod**
أصغر وحدة في Kubernetes، يحتوي على container واحد أو أكثر.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
```

#### 2. **Deployment**
يدير عدة نسخ من الـ Pods (Replicas).

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
```

#### 3. **Service**
يوفر IP ثابت و Load Balancing للـ Pods.

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
  type: LoadBalancer
```

---

## لماذا Kubernetes؟

### Docker وحده:
```
المشاكل:
❌ إدارة يدوية للـ containers
❌ لا يوجد Auto-scaling
❌ لا يوجد Self-healing
❌ صعوبة إدارة عدة servers
```

### مع Kubernetes:
```
الحلول:
✅ إدارة تلقائية
✅ Auto-scaling
✅ Self-healing
✅ Load Balancing
✅ Rolling Updates
✅ Service Discovery
```

---

## المكونات الأساسية:

```
Kubernetes Architecture:
├── Control Plane (Master Node)
│   ├── API Server
│   ├── Scheduler
│   ├── Controller Manager
│   └── etcd (Database)
└── Worker Nodes
    ├── kubelet
    ├── kube-proxy
    └── Container Runtime (Docker/containerd)
```

---

## الأوامر الأساسية:

```bash
# Pod Management
kubectl get pods
kubectl describe pod [name]
kubectl logs [pod-name]
kubectl delete pod [name]

# Deployment Management
kubectl create deployment [name] --image=[image]
kubectl get deployments
kubectl scale deployment [name] --replicas=5

# Service Management
kubectl get services
kubectl expose deployment [name] --port=80 --type=LoadBalancer

# Apply YAML
kubectl apply -f deployment.yaml
```

---

## متى تستخدم Kubernetes؟

✅ **استخدم Kubernetes عندما:**
- لديك عدة containers
- تحتاج High Availability
- تحتاج Auto-scaling
- إدارة Production environments
- Microservices Architecture

**أمثلة:**
- تطبيقات Production الكبيرة
- Microservices
- CI/CD Pipelines
- Multi-cloud Deployments

---

**للمزيد من التفاصيل، راجع [أسئلة المقابلات](Kubernetes-Interview-Questions.md)**
