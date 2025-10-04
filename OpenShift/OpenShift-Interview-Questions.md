# أسئلة مقابلات OpenShift

## المحتويات
1. [OpenShift vs Kubernetes](#openshift-vs-kubernetes)
2. [المفاهيم الأساسية](#المفاهيم-الأساسية)
3. [Source-to-Image (S2I)](#source-to-image-s2i)
4. [Security](#security)
5. [oc Commands](#oc-commands)
6. [سيناريوهات عملية](#سيناريوهات-عملية)

---

## OpenShift vs Kubernetes

### 1. ما الفرق بين OpenShift و Kubernetes؟

**الإجابة:**

OpenShift هو **Enterprise Kubernetes Platform** من Red Hat، مبني على Kubernetes مع إضافات.

#### الفروقات الأساسية:

| الميزة | Kubernetes | OpenShift |
|--------|-----------|-----------|
| **المطور** | CNCF (Open Source) | Red Hat |
| **النوع** | Orchestration Platform | Enterprise Platform |
| **Web Console** | Dashboard بسيط | Console متقدم جداً |
| **CI/CD** | يحتاج أدوات خارجية | مدمج (Pipelines) |
| **Security** | جيد | متقدم جداً (SELinux, SCC) |
| **Networking** | CNI Plugins | SDN مدمج |
| **Registry** | خارجي | مدمج |
| **Router** | Ingress Controller | Router مدمج |
| **S2I** | ❌ | ✅ |
| **Templates** | ❌ | ✅ |
| **Operators** | Operator Hub | OperatorHub مدمج |
| **الدعم** | Community | Commercial (Red Hat) |
| **التكلفة** | مجاني | Enterprise (مدفوع) |

#### OpenShift = Kubernetes + الإضافات التالية:

```
OpenShift Components:
├── Kubernetes Core ✓
├── Web Console (متقدم)
├── Integrated Container Registry
├── Router (Ingress)
├── Source-to-Image (S2I)
├── BuildConfig & Builds
├── ImageStreams
├── Templates
├── Projects (Namespaces++)
├── DeploymentConfig
├── Routes
├── Security Context Constraints (SCC)
├── Integrated Monitoring (Prometheus/Grafana)
├── Integrated Logging (EFK Stack)
└── OpenShift Pipelines (Tekton)
```

---

## المفاهيم الأساسية

### 2. ما الفرق بين Project و Namespace؟

**الإجابة:**

**Namespace (Kubernetes):**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: my-namespace
```
- مجرد عزل منطقي للموارد

**Project (OpenShift):**
```bash
oc new-project my-project --display-name="My Application"
```
- Namespace + إضافات:
  - Annotations
  - RBAC تلقائي
  - Network isolation افتراضي
  - Resource quotas
  - Limit ranges

**مثال:**
```bash
# إنشاء project
oc new-project ecommerce \
  --display-name="E-Commerce App" \
  --description="Production e-commerce application"

# OpenShift يُنشئ تلقائياً:
# 1. Namespace: ecommerce
# 2. RoleBindings للمستخدم
# 3. Network policies
# 4. Service accounts
```

---

### 3. ما الفرق بين Route و Ingress؟

**الإجابة:**

#### Ingress (Kubernetes):
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-service
            port:
              number: 8080
```

#### Route (OpenShift):
```yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: myapp-route
spec:
  host: myapp-ecommerce.apps.openshift.com
  to:
    kind: Service
    name: myapp-service
  port:
    targetPort: 8080
  tls:
    termination: edge
    insecureEdgeTerminationPolicy: Redirect
```

**الفرق:**
- ✅ Route أسهل وأبسط
- ✅ Route يدعم TLS تلقائياً
- ✅ Route يُنشئ URL تلقائي
- ✅ Route مدمج مع OpenShift Router

**إنشاء Route من CLI:**
```bash
# بسيط جداً
oc expose service myapp

# مع hostname محدد
oc expose service myapp --hostname=myapp.example.com

# مع TLS
oc create route edge --service=myapp --hostname=myapp.example.com
```

---

### 4. ما الفرق بين DeploymentConfig و Deployment؟

**الإجابة:**

#### Deployment (Kubernetes):
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:v1
```

#### DeploymentConfig (OpenShift):
```yaml
apiVersion: apps.openshift.io/v1
kind: DeploymentConfig
metadata:
  name: myapp
spec:
  replicas: 3
  strategy:
    type: Rolling
  triggers:
  - type: ConfigChange
  - type: ImageChange
    imageChangeParams:
      automatic: true
      containerNames:
      - myapp
      from:
        kind: ImageStreamTag
        name: myapp:latest
  selector:
    app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:latest
```

**الفرق:**

| Deployment | DeploymentConfig |
|------------|------------------|
| Kubernetes standard | OpenShift specific |
| Triggers يدوية | Triggers تلقائية |
| لا يدعم ImageStreams | يدعم ImageStreams ✓ |
| Lifecycle hooks محدودة | Lifecycle hooks متقدمة |

**Triggers في DeploymentConfig:**
```yaml
triggers:
  # عند تغيير Config
  - type: ConfigChange

  # عند تغيير Image
  - type: ImageChange
    imageChangeParams:
      automatic: true
      from:
        kind: ImageStreamTag
        name: myapp:latest
```

**ملاحظة:** OpenShift يدعم الاثنين، لكن يُنصح باستخدام **Deployment** الآن.

---

## Source-to-Image (S2I)

### 5. ما هو S2I؟ وكيف يعمل؟

**الإجابة:**

**Source-to-Image (S2I)** هي أداة لبناء Docker images مباشرة من الكود المصدري بدون Dockerfile.

#### كيف يعمل:

```bash
# نشر من Git مباشرة
oc new-app nodejs~https://github.com/user/my-nodejs-app

# OpenShift يقوم بـ:
# 1. Clone الكود من Git
# 2. اكتشاف اللغة (Node.js)
# 3. اختيار S2I builder image (nodejs)
# 4. بناء Image
# 5. Push للـ integrated registry
# 6. Deploy الـ application
# 7. إنشاء Service
```

#### أمثلة لغات مختلفة:

```bash
# Node.js
oc new-app nodejs~https://github.com/user/nodejs-app

# Python
oc new-app python~https://github.com/user/python-app

# Java
oc new-app java~https://github.com/user/java-app

# Ruby
oc new-app ruby~https://github.com/user/ruby-app

# .NET Core
oc new-app dotnet~https://github.com/user/dotnet-app
```

#### مع Branch محدد:

```bash
oc new-app nodejs~https://github.com/user/app#develop
```

#### مع Context Directory:

```bash
oc new-app nodejs~https://github.com/user/monorepo \
  --context-dir=frontend
```

#### الفوائد:

✅ لا حاجة لـ Dockerfile
✅ سريع وسهل
✅ Best practices مدمجة
✅ Security patches تلقائية
✅ Reproducible builds

---

### 6. ما هو BuildConfig؟

**الإجابة:**

**BuildConfig** يحدد كيفية بناء الـ container image.

```yaml
apiVersion: build.openshift.io/v1
kind: BuildConfig
metadata:
  name: myapp
spec:
  source:
    type: Git
    git:
      uri: https://github.com/user/myapp
      ref: main
  strategy:
    type: Source
    sourceStrategy:
      from:
        kind: ImageStreamTag
        name: nodejs:14
  output:
    to:
      kind: ImageStreamTag
      name: myapp:latest
  triggers:
  - type: ConfigChange
  - type: ImageChange
  - type: GitHub
    github:
      secret: github-webhook-secret
```

#### أنواع Build Strategies:

**1. Source (S2I)**
```yaml
strategy:
  type: Source
  sourceStrategy:
    from:
      kind: ImageStreamTag
      name: nodejs:14
```

**2. Docker**
```yaml
strategy:
  type: Docker
  dockerStrategy:
    dockerfilePath: Dockerfile
```

**3. Custom**
```yaml
strategy:
  type: Custom
  customStrategy:
    from:
      kind: ImageStreamTag
      name: custom-builder
```

**4. Pipeline**
```yaml
strategy:
  type: JenkinsPipeline
  jenkinsPipelineStrategy:
    jenkinsfilePath: Jenkinsfile
```

#### تشغيل Build:

```bash
# بدء build جديد
oc start-build myapp

# من Git branch محدد
oc start-build myapp --from-ref=develop

# من ملف محلي
oc start-build myapp --from-dir=.

# متابعة logs
oc logs -f bc/myapp
```

---

### 7. ما هو ImageStream؟

**الإجابة:**

**ImageStream** يمثل مجموعة من container images ذات صلة، مع tags مختلفة.

```yaml
apiVersion: image.openshift.io/v1
kind: ImageStream
metadata:
  name: myapp
spec:
  lookupPolicy:
    local: true
  tags:
  - name: latest
    from:
      kind: DockerImage
      name: myregistry.com/myapp:latest
    importPolicy:
      scheduled: true
  - name: v1.0
    from:
      kind: DockerImage
      name: myregistry.com/myapp:v1.0
  - name: v1.1
    from:
      kind: DockerImage
      name: myregistry.com/myapp:v1.1
```

#### الفوائد:

✅ **Abstraction**: يفصل Image reference عن Deployment
✅ **Auto-update**: DeploymentConfig يتحدث تلقائياً عند تغيير Image
✅ **Versioning**: إدارة versions بسهولة
✅ **Security**: Image signatures و scanning

**مثال عملي:**

```bash
# عرض ImageStreams
oc get is

# عرض tags
oc describe is myapp

# Tag image جديد
oc tag myapp:latest myapp:v2.0

# Rollback
oc tag myapp:v1.0 myapp:latest

# Import image من Docker Hub
oc import-image nginx --from=nginx:latest --confirm
```

---

## Security

### 8. ما هو Security Context Constraints (SCC)؟

**الإجابة:**

**SCC** يتحكم في ما يمكن للـ pods فعله من ناحية الأمان.

#### SCCs الافتراضية:

```bash
oc get scc

# النتيجة:
# restricted          ← الأكثر أماناً (افتراضي)
# anyuid
# hostaccess
# hostmount-anyuid
# hostnetwork
# node-exporter
# nonroot
# privileged          ← الأقل أماناً
```

#### مثال: SCC محدود (restricted):

```yaml
apiVersion: security.openshift.io/v1
kind: SecurityContextConstraints
metadata:
  name: restricted
allowHostDirVolumePlugin: false
allowHostIPC: false
allowHostNetwork: false
allowHostPID: false
allowHostPorts: false
allowPrivilegedContainer: false
allowedCapabilities: null
runAsUser:
  type: MustRunAsRange
seLinuxContext:
  type: MustRunAs
fsGroup:
  type: MustRunAs
```

**استخدام:**

```bash
# عرض SCC للـ service account
oc describe scc restricted

# إضافة SCC لـ service account
oc adm policy add-scc-to-user anyuid -z myapp-sa

# إزالة
oc adm policy remove-scc-from-user anyuid -z myapp-sa

# إضافة لـ group
oc adm policy add-scc-to-group anyuid system:authenticated
```

**متى تحتاج SCC غير restricted:**

```bash
# مثال: تطبيق يحتاج root access
oc create sa privileged-sa
oc adm policy add-scc-to-user privileged -z privileged-sa

# ثم في Deployment:
spec:
  serviceAccountName: privileged-sa
```

---

### 9. كيف تدير RBAC في OpenShift؟

**الإجابة:**

OpenShift يستخدم نفس RBAC كـ Kubernetes لكن مع roles إضافية.

#### Cluster Roles الافتراضية:

```bash
# عرض Cluster Roles
oc get clusterroles

# Roles الأساسية:
# cluster-admin     ← كل الصلاحيات
# admin            ← إدارة project
# edit             ← تعديل resources
# view             ← قراءة فقط
# self-provisioner ← إنشاء projects
```

#### إضافة User لـ Project:

```bash
# Admin role
oc adm policy add-role-to-user admin user1 -n myproject

# Edit role
oc adm policy add-role-to-user edit user2 -n myproject

# View role
oc adm policy add-role-to-user view user3 -n myproject

# حذف
oc adm policy remove-role-from-user admin user1 -n myproject
```

#### Custom Role:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: myproject
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["pods/log"]
  verbs: ["get"]
```

```bash
# إنشاء
oc create -f pod-reader-role.yaml

# ربط بـ user
oc create rolebinding pod-reader-binding \
  --role=pod-reader \
  --user=developer1 \
  -n myproject
```

---

## oc Commands

### 10. ما الفرق بين oc و kubectl؟

**الإجابة:**

`oc` هو CLI لـ OpenShift، مبني على `kubectl` مع إضافات.

#### أوامر موجودة في الاثنين:

```bash
# تعمل في oc و kubectl
oc get pods
oc describe deployment myapp
oc logs -f pod/myapp-123
oc exec -it pod/myapp bash
oc apply -f deployment.yaml
oc delete pod myapp-123
```

#### أوامر خاصة بـ oc:

```bash
# Project management
oc new-project myproject
oc project myproject
oc projects

# Application deployment
oc new-app nodejs~https://github.com/user/app
oc new-app --docker-image=nginx

# Routes
oc expose service myapp
oc get routes

# Builds
oc start-build myapp
oc logs -f bc/myapp
oc get builds

# Login
oc login https://api.openshift.com
oc whoami
oc logout

# Registry
oc registry info
oc registry login

# Status
oc status
oc adm top nodes
oc adm top pods

# Policy
oc adm policy add-role-to-user admin user1
oc adm policy add-scc-to-user anyuid -z myapp-sa
```

---

### 11. أوامر oc الأساسية

**الإجابة:**

```bash
# === Login & Context ===
oc login https://api.openshift.com --token=<token>
oc login -u developer -p password
oc whoami
oc config view
oc config get-contexts

# === Project Management ===
oc new-project ecommerce --display-name="E-Commerce"
oc projects
oc project ecommerce
oc describe project ecommerce
oc delete project old-project

# === Application Deployment ===
# S2I من Git
oc new-app nodejs~https://github.com/user/app
oc new-app python~https://github.com/user/api --name=backend

# من Docker Image
oc new-app nginx
oc new-app mysql MYSQL_USER=user MYSQL_PASSWORD=pass

# مع Environment Variables
oc new-app nodejs~https://github.com/user/app \
  -e NODE_ENV=production \
  -e PORT=8080

# === Resource Management ===
oc get all
oc get pods
oc get deployments
oc get services
oc get routes
oc get dc  # DeploymentConfigs
oc get bc  # BuildConfigs
oc get is  # ImageStreams

# === Scaling ===
oc scale deployment myapp --replicas=5
oc scale dc myapp --replicas=3
oc autoscale deployment myapp --min=2 --max=10 --cpu-percent=80

# === Expose Service ===
oc expose service myapp
oc expose service myapp --hostname=myapp.example.com
oc create route edge --service=myapp --hostname=secure.example.com

# === Build Management ===
oc start-build myapp
oc start-build myapp --follow
oc start-build myapp --from-dir=.
oc cancel-build myapp-1
oc delete build myapp-1

# === Logs & Debugging ===
oc logs pod/myapp-123
oc logs -f deployment/myapp
oc logs -f bc/myapp  # Build logs
oc logs -f dc/myapp  # DeploymentConfig logs

# === Execute Commands ===
oc exec -it pod/myapp-123 -- bash
oc exec pod/myapp-123 -- env
oc exec pod/myapp-123 -- ls /app

# === Port Forward ===
oc port-forward pod/myapp-123 8080:8080
oc port-forward service/myapp 8080:80

# === ConfigMaps & Secrets ===
oc create configmap app-config --from-file=config.json
oc create secret generic db-secret --from-literal=password=secret123
oc set env deployment/myapp --from=configmap/app-config
oc set env deployment/myapp --from=secret/db-secret

# === Rollout Management ===
oc rollout status deployment/myapp
oc rollout history deployment/myapp
oc rollout undo deployment/myapp
oc rollout restart deployment/myapp

# === Image Management ===
oc import-image nginx --from=nginx:latest --confirm
oc tag myapp:latest myapp:v1.0
oc describe is myapp

# === RBAC ===
oc adm policy add-role-to-user admin user1
oc adm policy add-role-to-user view user2 -n myproject
oc adm policy who-can delete pods

# === SCC ===
oc get scc
oc adm policy add-scc-to-user anyuid -z myapp-sa
oc describe scc restricted

# === Troubleshooting ===
oc describe pod myapp-123
oc get events
oc get events --sort-by='.lastTimestamp'
oc debug pod/myapp-123
oc adm top nodes
oc adm top pods

# === Export & Backup ===
oc get all -o yaml > backup.yaml
oc get deployment myapp -o yaml > myapp-deployment.yaml
oc export deployment myapp > myapp-template.yaml  # بدون metadata
```

---

## سيناريوهات عملية

### 12. نشر تطبيق Full-Stack في OpenShift

**السيناريو:** نشر تطبيق E-Commerce (React + Node.js + PostgreSQL)

**الحل:**

```bash
# 1. إنشاء Project
oc new-project ecommerce --display-name="E-Commerce Application"

# 2. نشر PostgreSQL من Template
oc new-app postgresql-persistent \
  -p POSTGRESQL_USER=ecommerce \
  -p POSTGRESQL_PASSWORD=secret123 \
  -p POSTGRESQL_DATABASE=ecommerce \
  -p VOLUME_CAPACITY=10Gi

# 3. نشر Backend (Node.js API) من Git
oc new-app nodejs~https://github.com/company/ecommerce-api \
  --name=backend \
  -e DATABASE_URL=postgresql://ecommerce:secret123@postgresql/ecommerce \
  -e JWT_SECRET=supersecret \
  -e NODE_ENV=production

# 4. إنشاء Route للـ Backend
oc expose service backend --hostname=api.ecommerce.example.com

# 5. نشر Frontend (React) من Git
oc new-app nodejs~https://github.com/company/ecommerce-frontend \
  --name=frontend \
  -e REACT_APP_API_URL=https://api.ecommerce.example.com

# 6. إنشاء Route للـ Frontend
oc expose service frontend --hostname=ecommerce.example.com

# 7. إضافة Health Checks للـ Backend
oc set probe deployment/backend \
  --liveness \
  --get-url=http://:8080/health \
  --initial-delay-seconds=30 \
  --period-seconds=10

oc set probe deployment/backend \
  --readiness \
  --get-url=http://:8080/ready \
  --initial-delay-seconds=10 \
  --period-seconds=5

# 8. Auto-scaling للـ Backend
oc autoscale deployment/backend --min=2 --max=10 --cpu-percent=70

# 9. عرض كل الموارد
oc get all
oc status
```

**النتيجة:**
```
✅ PostgreSQL Database (PersistentVolume)
✅ Backend API (2-10 replicas, auto-scaling)
✅ Frontend (React SPA)
✅ Routes مع TLS
✅ Health checks
```

---

### 13. CI/CD Pipeline في OpenShift

**السيناريو:** إنشاء pipeline للبناء والنشر التلقائي

**الحل: استخدام OpenShift Pipelines (Tekton)**

```yaml
# pipeline.yaml
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: build-and-deploy
spec:
  params:
  - name: GIT_REPO
    type: string
    description: Git repository URL
  - name: GIT_REVISION
    type: string
    default: main
  - name: IMAGE_NAME
    type: string

  workspaces:
  - name: shared-workspace

  tasks:
  # Task 1: Clone Repository
  - name: fetch-repository
    taskRef:
      name: git-clone
    workspaces:
    - name: output
      workspace: shared-workspace
    params:
    - name: url
      value: $(params.GIT_REPO)
    - name: revision
      value: $(params.GIT_REVISION)

  # Task 2: Run Tests
  - name: run-tests
    taskRef:
      name: npm-test
    runAfter:
    - fetch-repository
    workspaces:
    - name: source
      workspace: shared-workspace

  # Task 3: Build Image
  - name: build-image
    taskRef:
      name: buildah
    runAfter:
    - run-tests
    workspaces:
    - name: source
      workspace: shared-workspace
    params:
    - name: IMAGE
      value: $(params.IMAGE_NAME)

  # Task 4: Deploy
  - name: deploy
    taskRef:
      name: openshift-client
    runAfter:
    - build-image
    params:
    - name: SCRIPT
      value: |
        oc rollout status deployment/myapp
        oc set image deployment/myapp myapp=$(params.IMAGE_NAME)
```

```bash
# تشغيل Pipeline
tkn pipeline start build-and-deploy \
  -p GIT_REPO=https://github.com/company/app \
  -p IMAGE_NAME=image-registry.openshift-image-registry.svc:5000/myproject/myapp \
  --use-param-defaults \
  --workspace name=shared-workspace,claimName=pipeline-pvc \
  --showlog
```

---

### 14. Blue-Green Deployment في OpenShift

**السيناريو:** نشر نسخة جديدة بدون downtime

**الحل:**

```bash
# 1. النسخة الحالية (Blue - v1.0)
oc new-app nodejs~https://github.com/user/app#v1.0 --name=myapp-blue
oc expose service myapp-blue

# 2. Route تشير للـ Blue
oc create route edge myapp --service=myapp-blue --hostname=myapp.example.com

# 3. نشر النسخة الجديدة (Green - v2.0)
oc new-app nodejs~https://github.com/user/app#v2.0 --name=myapp-green

# 4. اختبار Green (internal)
oc port-forward service/myapp-green 8080:8080
# اختبر على localhost:8080

# 5. التبديل للـ Green (تدريجياً)
# 50% للـ Green
oc set route-backends myapp myapp-blue=50 myapp-green=50

# راقب الأخطاء، إذا كل شيء جيد:
# 100% للـ Green
oc set route-backends myapp myapp-blue=0 myapp-green=100

# أو مباشرة:
oc patch route myapp -p '{"spec":{"to":{"name":"myapp-green"}}}'

# 6. حذف Blue بعد التأكد
oc delete all -l app=myapp-blue
```

---

### 15. Monitoring & Alerts في OpenShift

**السيناريو:** مراقبة التطبيقات وإنشاء تنبيهات

**الحل:**

OpenShift يأتي مع **Monitoring Stack** مدمج (Prometheus + Grafana).

#### 1. تفعيل User Workload Monitoring:

```yaml
# cluster-monitoring-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cluster-monitoring-config
  namespace: openshift-monitoring
data:
  config.yaml: |
    enableUserWorkload: true
```

#### 2. ServiceMonitor للتطبيق:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: myapp-monitor
  namespace: myproject
spec:
  selector:
    matchLabels:
      app: myapp
  endpoints:
  - port: web
    interval: 30s
    path: /metrics
```

#### 3. إضافة Metrics للتطبيق (Node.js مثال):

```javascript
// app.js
const promClient = require('prom-client');
const express = require('express');
const app = express();

// Enable default metrics
const register = promClient.register;
promClient.collectDefaultMetrics({ register });

// Custom metrics
const httpRequestDuration = new promClient.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code']
});

// Metrics endpoint
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType);
  res.end(await register.metrics());
});

app.listen(8080);
```

#### 4. PrometheusRule للتنبيهات:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: myapp-alerts
  namespace: myproject
spec:
  groups:
  - name: myapp
    interval: 30s
    rules:
    # High Error Rate
    - alert: HighErrorRate
      expr: |
        rate(http_requests_total{status_code=~"5.."}[5m]) > 0.05
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "High error rate detected"
        description: "Error rate is {{ $value }} for {{ $labels.pod }}"

    # High Memory Usage
    - alert: HighMemoryUsage
      expr: |
        container_memory_usage_bytes{pod=~"myapp-.*"} /
        container_spec_memory_limit_bytes{pod=~"myapp-.*"} > 0.9
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "High memory usage"
        description: "Memory usage is {{ $value }}% for {{ $labels.pod }}"

    # Pod Down
    - alert: PodDown
      expr: |
        kube_pod_status_phase{phase!="Running",namespace="myproject"} == 1
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "Pod is down"
        description: "Pod {{ $labels.pod }} is not running"
```

#### 5. عرض Metrics:

```bash
# الوصول لـ Prometheus
oc get route prometheus-k8s -n openshift-monitoring

# الوصول لـ Grafana
oc get route grafana -n openshift-monitoring

# Query من CLI
oc exec -n openshift-monitoring prometheus-k8s-0 -- \
  promtool query instant http://localhost:9090 \
  'up{namespace="myproject"}'
```

---

## ملخص المقارنة السريعة

| الميزة | Kubernetes | OpenShift |
|--------|-----------|-----------|
| **الإعداد** | معقد | أبسط (لكن ما زال معقد) |
| **Web UI** | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Developer UX** | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Security** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **CI/CD** | يحتاج أدوات | مدمج ⭐⭐⭐⭐⭐ |
| **S2I** | ❌ | ✅ |
| **التكلفة** | مجاني | Enterprise $$ |
| **الدعم** | Community | Red Hat ⭐⭐⭐⭐⭐ |
| **Use Case** | أي بيئة | Enterprise |

---

**حظاً موفقاً في المقابلة! 🚀**
