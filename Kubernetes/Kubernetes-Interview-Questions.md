# أسئلة مقابلات Kubernetes الشاملة

## جدول المحتويات
1. [بنية Kubernetes](#بنية-kubernetes)
2. [Pods و Deployments و Services](#pods-و-deployments-و-services)
3. [ConfigMaps و Secrets](#configmaps-و-secrets)
4. [Volumes و PersistentVolumes](#volumes-و-persistentvolumes)
5. [Namespaces](#namespaces)
6. [Ingress والشبكات](#ingress-والشبكات)
7. [التوسع التلقائي](#التوسع-التلقائي)
8. [RBAC والأمان](#rbac-والأمان)
9. [أوامر kubectl](#أوامر-kubectl)
10. [استكشاف الأخطاء وإصلاحها](#استكشاف-الأخطاء-وإصلاحها)
11. [Helm](#helm)
12. [سيناريوهات عملية](#سيناريوهات-عملية)

---

## بنية Kubernetes

### السؤال 1: ما هي المكونات الرئيسية لـ Control Plane في Kubernetes؟

**الجواب:**

Control Plane يدير الكلاستر بالكامل ويتكون من:

1. **API Server (kube-apiserver)**
   - نقطة الدخول الرئيسية لجميع أوامر REST
   - يتعامل مع جميع طلبات API
   - يتحقق من الصحة ويعالج البيانات

2. **etcd**
   - قاعدة بيانات موزعة key-value
   - تخزين جميع بيانات الكلاستر
   - مصدر الحقيقة الوحيد للكلاستر

3. **Scheduler (kube-scheduler)**
   - يقرر على أي node يتم تشغيل Pod
   - يأخذ في الاعتبار الموارد والقيود

4. **Controller Manager (kube-controller-manager)**
   - يشغل controllers المختلفة
   - يضمن الحالة المطلوبة للكلاستر

5. **Cloud Controller Manager**
   - يدير التفاعلات مع مزودي الخدمات السحابية

**أوامر للتحقق:**
```bash
# عرض صحة المكونات
kubectl get componentstatuses

# عرض pods في namespace kube-system
kubectl get pods -n kube-system

# التحقق من حالة API server
kubectl cluster-info
```

---

### السؤال 2: ما هي المكونات الموجودة على Worker Nodes؟

**الجواب:**

كل worker node يحتوي على:

1. **Kubelet**
   - Agent يعمل على كل node
   - يضمن تشغيل Pods في containers
   - يتواصل مع API Server

2. **Kube-proxy**
   - يدير قواعد الشبكة على الـ nodes
   - يسمح بالاتصال الشبكي بين Pods
   - يوزع الحمل على Services

3. **Container Runtime**
   - البرنامج المسؤول عن تشغيل الـ containers
   - أمثلة: Docker، containerd، CRI-O

**أوامر للتحقق:**
```bash
# عرض جميع الـ nodes
kubectl get nodes

# عرض تفاصيل node معين
kubectl describe node <node-name>

# عرض معلومات الموارد
kubectl top nodes
```

---

### السؤال 3: كيف يعمل Kubernetes Scheduler؟

**الجواب:**

عملية الـ Scheduling تتم على مرحلتين:

**المرحلة 1: Filtering (التصفية)**
- يستبعد nodes التي لا تستوفي المتطلبات
- يتحقق من الموارد المتاحة (CPU، Memory)
- يتحقق من Node Selectors و Taints/Tolerations

**المرحلة 2: Scoring (التسجيل)**
- يعطي درجة لكل node مؤهل
- يختار node بأعلى درجة
- يأخذ في الاعتبار توزيع الحمل

**مثال على Node Affinity:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/os
            operator: In
            values:
            - linux
  containers:
  - name: nginx
    image: nginx
```

**أوامر مفيدة:**
```bash
# عرض أين تم جدولة pods
kubectl get pods -o wide

# منع scheduling على node
kubectl cordon <node-name>

# السماح بـ scheduling على node
kubectl uncordon <node-name>

# إفراغ node من pods
kubectl drain <node-name> --ignore-daemonsets
```

---

## Pods و Deployments و Services

### السؤال 4: ما هو Pod ولماذا نحتاجه؟

**الجواب:**

**Pod** هو أصغر وحدة قابلة للنشر في Kubernetes:
- يحتوي على container واحد أو أكثر
- يشارك الـ containers نفس الـ network namespace
- يشارك الـ containers نفس الـ storage volumes
- له IP address واحد

**لماذا نحتاج Pods؟**
- تجميع containers مترابطة معاً
- مشاركة الموارد بين containers
- التواصل عبر localhost
- دورة حياة مشتركة

**مثال على Pod بسيط:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

**مثال على Multi-Container Pod:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
  - name: app
    image: myapp:1.0
    ports:
    - containerPort: 8080

  - name: sidecar-logger
    image: busybox
    command: ['sh', '-c', 'tail -f /var/log/app.log']
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log

  volumes:
  - name: shared-logs
    emptyDir: {}
```

**أوامر:**
```bash
# إنشاء pod
kubectl apply -f pod.yaml

# عرض pods
kubectl get pods

# عرض تفاصيل pod
kubectl describe pod nginx-pod

# الدخول إلى pod
kubectl exec -it nginx-pod -- /bin/bash

# عرض logs
kubectl logs nginx-pod

# حذف pod
kubectl delete pod nginx-pod
```

---

### السؤال 5: ما الفرق بين Deployment و ReplicaSet و Pod؟

**الجواب:**

**Pod:**
- أصغر وحدة قابلة للنشر
- إذا فشل، لن يتم إعادة إنشائه تلقائياً

**ReplicaSet:**
- يضمن تشغيل عدد محدد من Pod replicas
- يستخدم للحفاظ على توافر Pods
- نادراً ما يُستخدم مباشرة

**Deployment:**
- يدير ReplicaSets
- يوفر rolling updates و rollbacks
- الخيار المفضل لإدارة stateless applications

**مثال على Deployment:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
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
        image: nginx:1.21
        ports:
        - containerPort: 80
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

**أوامر:**
```bash
# إنشاء deployment
kubectl create deployment nginx --image=nginx:1.21 --replicas=3

# عرض deployments
kubectl get deployments

# عرض replicasets
kubectl get rs

# تحديث صورة
kubectl set image deployment/nginx-deployment nginx=nginx:1.22

# توسيع عدد replicas
kubectl scale deployment nginx-deployment --replicas=5

# عرض حالة rollout
kubectl rollout status deployment/nginx-deployment

# عرض تاريخ rollout
kubectl rollout history deployment/nginx-deployment

# التراجع عن آخر تحديث
kubectl rollout undo deployment/nginx-deployment

# التراجع لإصدار معين
kubectl rollout undo deployment/nginx-deployment --to-revision=2
```

---

### السؤال 6: اشرح أنواع Services في Kubernetes؟

**الجواب:**

**1. ClusterIP (الافتراضي)**
- يُنشئ IP داخلي للوصول داخل الكلاستر فقط
- لا يمكن الوصول إليه من خارج الكلاستر

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

**2. NodePort**
- يفتح port معين على جميع الـ nodes
- يسمح بالوصول الخارجي
- Port range: 30000-32767

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  type: NodePort
  selector:
    app: frontend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
    nodePort: 30080
```

**3. LoadBalancer**
- يُنشئ load balancer خارجي
- يعمل مع مزودي الخدمات السحابية
- يوفر IP عام للوصول

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

**4. ExternalName**
- يربط Service باسم DNS خارجي
- لا يوجد proxy أو load balancing

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-db
spec:
  type: ExternalName
  externalName: database.example.com
```

**أوامر:**
```bash
# إنشاء service
kubectl expose deployment nginx-deployment --port=80 --type=LoadBalancer

# عرض services
kubectl get svc

# عرض تفاصيل service
kubectl describe svc backend-service

# عرض endpoints
kubectl get endpoints

# حذف service
kubectl delete svc backend-service
```

---

## ConfigMaps و Secrets

### السؤال 7: ما هو ConfigMap ومتى نستخدمه؟

**الجواب:**

**ConfigMap** يُستخدم لتخزين بيانات التكوين غير السرية في شكل key-value pairs:
- فصل التكوين عن الكود
- تسهيل إدارة التكوينات
- إمكانية تحديث التكوين دون إعادة بناء الصورة

**طرق إنشاء ConfigMap:**

**1. من literal values:**
```bash
kubectl create configmap app-config \
  --from-literal=DATABASE_URL=mysql://db:3306 \
  --from-literal=LOG_LEVEL=info
```

**2. من ملف:**
```bash
kubectl create configmap app-config --from-file=app.properties
```

**3. من YAML:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  DATABASE_URL: "mysql://db:3306/mydb"
  LOG_LEVEL: "info"
  app.properties: |
    database.host=mysql
    database.port=3306
    cache.enabled=true
```

**استخدام ConfigMap في Pod:**

**كمتغيرات بيئية:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: myapp:1.0
    envFrom:
    - configMapRef:
        name: app-config
    # أو استخدام متغيرات محددة
    env:
    - name: LOG_LEVEL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: LOG_LEVEL
```

**كـ volume:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: myapp:1.0
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: app-config
```

**أوامر:**
```bash
# عرض configmaps
kubectl get configmaps

# عرض محتوى configmap
kubectl describe configmap app-config

# تحديث configmap
kubectl edit configmap app-config

# حذف configmap
kubectl delete configmap app-config
```

---

### السؤال 8: ما الفرق بين ConfigMap و Secret؟

**الجواب:**

**ConfigMap:**
- لتخزين بيانات غير سرية
- النص عادي (plain text)
- مرئي بوضوح في YAML

**Secret:**
- لتخزين بيانات حساسة
- مُشفر بـ base64 (ليس تشفير حقيقي!)
- يوفر أمان إضافي مع encryption at rest

**أنواع Secrets:**

**1. Opaque (الافتراضي):**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: YWRtaW4=  # base64 encoded "admin"
  password: cGFzc3dvcmQxMjM=  # base64 encoded "password123"
stringData:
  # يمكن استخدام stringData للنص العادي
  database-url: "mysql://localhost:3306"
```

**2. Docker Registry:**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: docker-registry-secret
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: <base64-encoded-docker-config>
```

**3. TLS:**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret
type: kubernetes.io/tls
data:
  tls.crt: <base64-encoded-cert>
  tls.key: <base64-encoded-key>
```

**استخدام Secret في Pod:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: myapp:1.0
    env:
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: username
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secrets
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: db-secret
```

**أوامر:**
```bash
# إنشاء secret من literal
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=pass123

# إنشاء docker registry secret
kubectl create secret docker-registry my-registry \
  --docker-server=registry.example.com \
  --docker-username=user \
  --docker-password=pass \
  --docker-email=user@example.com

# إنشاء TLS secret
kubectl create secret tls tls-secret \
  --cert=path/to/tls.crt \
  --key=path/to/tls.key

# encode base64
echo -n 'admin' | base64

# decode base64
echo 'YWRtaW4=' | base64 -d

# عرض secrets
kubectl get secrets

# عرض محتوى secret
kubectl get secret db-secret -o yaml
```

---

## Volumes و PersistentVolumes

### السؤال 9: ما الفرق بين Volume و PersistentVolume و PersistentVolumeClaim؟

**الجواب:**

**Volume:**
- مرتبط بدورة حياة Pod
- يُحذف عند حذف Pod
- يُعرّف مباشرة في Pod spec

**PersistentVolume (PV):**
- مورد تخزين في الكلاستر
- مستقل عن دورة حياة Pod
- يُدار من قبل المسؤول

**PersistentVolumeClaim (PVC):**
- طلب للحصول على تخزين
- يربط Pod بـ PV
- يُستخدم من قبل المطورين

**أنواع Volumes الشائعة:**

**1. emptyDir:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: cache
      mountPath: /cache
  volumes:
  - name: cache
    emptyDir: {}
```

**2. hostPath:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: host-volume
      mountPath: /data
  volumes:
  - name: host-volume
    hostPath:
      path: /mnt/data
      type: Directory
```

**3. PersistentVolume و PVC:**

**PersistentVolume:**
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-data
spec:
  capacity:
    storage: 10Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: standard
  hostPath:
    path: /mnt/data
```

**PersistentVolumeClaim:**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-data
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: standard
```

**استخدام PVC في Pod:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: data-volume
      mountPath: /var/www/html
  volumes:
  - name: data-volume
    persistentVolumeClaim:
      claimName: pvc-data
```

**Access Modes:**
- **ReadWriteOnce (RWO):** قراءة-كتابة لـ node واحد
- **ReadOnlyMany (ROX):** قراءة فقط لعدة nodes
- **ReadWriteMany (RWX):** قراءة-كتابة لعدة nodes

**أوامر:**
```bash
# عرض PVs
kubectl get pv

# عرض PVCs
kubectl get pvc

# عرض تفاصيل PV
kubectl describe pv pv-data

# عرض تفاصيل PVC
kubectl describe pvc pvc-data

# حذف PVC
kubectl delete pvc pvc-data
```

---

### السؤال 10: اشرح StorageClass وكيف يعمل Dynamic Provisioning؟

**الجواب:**

**StorageClass** يوفر طريقة لوصف "classes" مختلفة من التخزين:
- يُستخدم للـ dynamic provisioning
- يحدد provisioner والمعاملات
- يسمح للمطورين بطلب تخزين دون معرفة التفاصيل

**مثال على StorageClass:**
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-storage
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  iops: "3000"
  throughput: "125"
  encrypted: "true"
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
reclaimPolicy: Delete
```

**PVC مع Dynamic Provisioning:**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: fast-storage
  resources:
    requests:
      storage: 20Gi
```

**StatefulSet مع PVC Template:**
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: database
spec:
  serviceName: database
  replicas: 3
  selector:
    matchLabels:
      app: db
  template:
    metadata:
      labels:
        app: db
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: fast-storage
      resources:
        requests:
          storage: 10Gi
```

**أوامر:**
```bash
# عرض storage classes
kubectl get sc

# عرض تفاصيل storage class
kubectl describe sc fast-storage

# تعيين storage class افتراضي
kubectl patch storageclass fast-storage \
  -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

---

## Namespaces

### السؤال 11: ما هي Namespaces ومتى نستخدمها؟

**الجواب:**

**Namespaces** توفر عزل منطقي للموارد داخل الكلاستر:
- تقسيم الموارد بين فرق متعددة
- عزل البيئات (dev, staging, prod)
- تطبيق resource quotas و limits
- تنظيم الموارد

**Namespaces الافتراضية:**
- **default:** للموارد بدون namespace محدد
- **kube-system:** لموارد Kubernetes النظامية
- **kube-public:** متاحة للقراءة للجميع
- **kube-node-lease:** لـ node heartbeats

**إنشاء Namespace:**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: development
  labels:
    environment: dev
```

**ResourceQuota:**
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: development
spec:
  hard:
    requests.cpu: "10"
    requests.memory: 20Gi
    limits.cpu: "20"
    limits.memory: 40Gi
    persistentvolumeclaims: "10"
    pods: "50"
    services: "10"
```

**LimitRange:**
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: dev-limits
  namespace: development
spec:
  limits:
  - max:
      cpu: "2"
      memory: "4Gi"
    min:
      cpu: "100m"
      memory: "128Mi"
    default:
      cpu: "500m"
      memory: "512Mi"
    defaultRequest:
      cpu: "250m"
      memory: "256Mi"
    type: Container
```

**Pod مع Namespace:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
  namespace: development
spec:
  containers:
  - name: app
    image: myapp:1.0
```

**أوامر:**
```bash
# عرض namespaces
kubectl get namespaces

# إنشاء namespace
kubectl create namespace development

# عرض موارد في namespace معين
kubectl get pods -n development

# عرض موارد في جميع namespaces
kubectl get pods --all-namespaces
# أو
kubectl get pods -A

# تعيين namespace افتراضي
kubectl config set-context --current --namespace=development

# حذف namespace
kubectl delete namespace development

# عرض resource quota
kubectl get resourcequota -n development

# عرض limit range
kubectl get limitrange -n development
```

---

## Ingress والشبكات

### السؤال 12: ما هو Ingress وكيف يختلف عن Service؟

**الجواب:**

**Service:**
- يعمل على Layer 4 (TCP/UDP)
- يوزع الحمل بين Pods
- محدود في routing capabilities

**Ingress:**
- يعمل على Layer 7 (HTTP/HTTPS)
- يوفر routing متقدم
- يدعم SSL/TLS termination
- يدعم name-based virtual hosting
- يتطلب Ingress Controller

**Ingress Controller:**
يجب تثبيت Ingress Controller أولاً (مثل nginx-ingress، traefik):
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml
```

**مثال بسيط على Ingress:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: simple-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
```

**Ingress متقدم مع TLS وعدة paths:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: advanced-ingress
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/rate-limit: "100"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - myapp.example.com
    secretName: myapp-tls
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: backend-service
            port:
              number: 8080
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
  - host: admin.myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: admin-service
            port:
              number: 3000
```

**Ingress مع Basic Auth:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: auth-ingress
  annotations:
    nginx.ingress.kubernetes.io/auth-type: basic
    nginx.ingress.kubernetes.io/auth-secret: basic-auth
    nginx.ingress.kubernetes.io/auth-realm: 'Authentication Required'
spec:
  ingressClassName: nginx
  rules:
  - host: secure.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: secure-service
            port:
              number: 80
```

**أوامر:**
```bash
# عرض ingresses
kubectl get ingress

# عرض تفاصيل ingress
kubectl describe ingress simple-ingress

# عرض ingress controllers
kubectl get pods -n ingress-nginx

# اختبار ingress
curl -H "Host: myapp.example.com" http://<ingress-ip>

# إنشاء basic auth secret
htpasswd -c auth username
kubectl create secret generic basic-auth --from-file=auth
```

---

### السؤال 13: اشرح Network Policies في Kubernetes؟

**الجواب:**

**Network Policies** تتحكم في حركة المرور بين Pods:
- الافتراضي: جميع Pods يمكنها التواصل
- تعمل على Layer 3/4
- تتطلب network plugin يدعمها (مثل Calico، Cilium)

**مثال 1: منع جميع الاتصالات الواردة:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

**مثال 2: السماح بالاتصال من pods معينة فقط:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-frontend
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
```

**مثال 3: السماح بالاتصال من namespace معين:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-namespace
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: database
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          environment: production
    - podSelector:
        matchLabels:
          app: backend
    ports:
    - protocol: TCP
      port: 3306
```

**مثال 4: التحكم في Egress:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns-and-external
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: frontend
  policyTypes:
  - Egress
  egress:
  # السماح بـ DNS
  - to:
    - namespaceSelector:
        matchLabels:
          name: kube-system
    ports:
    - protocol: UDP
      port: 53
  # السماح بالاتصال بـ external API
  - to:
    - ipBlock:
        cidr: 0.0.0.0/0
        except:
        - 169.254.169.254/32
    ports:
    - protocol: TCP
      port: 443
```

**مثال 5: سياسة شاملة:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: complete-network-policy
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: api-server
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          environment: production
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 5432
  - to:
    - namespaceSelector: {}
    ports:
    - protocol: UDP
      port: 53
```

**أوامر:**
```bash
# عرض network policies
kubectl get networkpolicies -n production

# عرض تفاصيل network policy
kubectl describe networkpolicy allow-from-frontend -n production

# اختبار الاتصال
kubectl exec -it frontend-pod -n production -- curl backend-service:8080
```

---

## التوسع التلقائي

### السؤال 14: اشرح Horizontal Pod Autoscaler (HPA)؟

**الجواب:**

**HPA** يقوم بتوسيع عدد Pods تلقائياً بناءً على:
- استخدام CPU
- استخدام Memory
- Custom metrics

**المتطلبات:**
- Metrics Server يجب أن يكون مثبتاً
- يجب تحديد resource requests للـ pods

**تثبيت Metrics Server:**
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

**مثال بسيط على HPA:**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 30
      - type: Pods
        value: 4
        periodSeconds: 30
      selectPolicy: Max
```

**HPA متقدم مع عدة metrics:**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: advanced-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app
  minReplicas: 3
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 60
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  - type: Pods
    pods:
      metric:
        name: http_requests_per_second
      target:
        type: AverageValue
        averageValue: "1000"
```

**Deployment مع resource requests (مطلوب للـ HPA):**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: app
        image: myapp:1.0
        resources:
          requests:
            cpu: 200m
            memory: 256Mi
          limits:
            cpu: 500m
            memory: 512Mi
```

**أوامر:**
```bash
# إنشاء HPA من سطر الأوامر
kubectl autoscale deployment app-deployment \
  --cpu-percent=70 \
  --min=2 \
  --max=10

# عرض HPAs
kubectl get hpa

# عرض تفاصيل HPA
kubectl describe hpa app-hpa

# عرض metrics
kubectl top pods
kubectl top nodes

# اختبار HPA بتوليد حمل
kubectl run -it load-generator --rm --image=busybox --restart=Never -- \
  /bin/sh -c "while sleep 0.01; do wget -q -O- http://app-service; done"
```

---

### السؤال 15: ما هو Vertical Pod Autoscaler (VPA)؟

**الجواب:**

**VPA** يقوم بتعديل resource requests و limits تلقائياً:
- يحلل استخدام الموارد الفعلي
- يوصي بقيم مثالية
- يمكن أن يطبق التغييرات تلقائياً

**تثبيت VPA:**
```bash
git clone https://github.com/kubernetes/autoscaler.git
cd autoscaler/vertical-pod-autoscaler
./hack/vpa-up.sh
```

**مثال على VPA:**
```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app-deployment
  updatePolicy:
    updateMode: "Auto"  # أو "Initial" أو "Off"
  resourcePolicy:
    containerPolicies:
    - containerName: app
      minAllowed:
        cpu: 100m
        memory: 128Mi
      maxAllowed:
        cpu: 2
        memory: 2Gi
      controlledResources:
      - cpu
      - memory
```

**Update Modes:**
- **Off:** توصيات فقط، بدون تطبيق
- **Initial:** يطبق عند إنشاء Pod فقط
- **Recreate:** يعيد إنشاء Pods مع قيم جديدة
- **Auto:** يطبق تلقائياً (قد يتطلب restart)

**أوامر:**
```bash
# عرض VPAs
kubectl get vpa

# عرض توصيات VPA
kubectl describe vpa app-vpa

# عرض التوصيات بشكل مفصل
kubectl get vpa app-vpa -o yaml
```

**ملاحظة مهمة:**
لا تستخدم VPA و HPA معاً على نفس الموارد (CPU/Memory) لتجنب التعارض!

---

### السؤال 16: اشرح Cluster Autoscaler؟

**الجواب:**

**Cluster Autoscaler** يقوم بتوسيع عدد Nodes في الكلاستر:
- يضيف nodes عندما لا يمكن جدولة pods
- يحذف nodes غير المستخدمة
- يعمل مع cloud providers

**مثال على Cluster Autoscaler (AWS):**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cluster-autoscaler
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cluster-autoscaler
  template:
    metadata:
      labels:
        app: cluster-autoscaler
    spec:
      serviceAccountName: cluster-autoscaler
      containers:
      - image: k8s.gcr.io/autoscaling/cluster-autoscaler:v1.27.0
        name: cluster-autoscaler
        command:
        - ./cluster-autoscaler
        - --v=4
        - --stderrthreshold=info
        - --cloud-provider=aws
        - --skip-nodes-with-local-storage=false
        - --expander=least-waste
        - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/my-cluster
        - --balance-similar-node-groups
        - --skip-nodes-with-system-pods=false
        env:
        - name: AWS_REGION
          value: us-east-1
        resources:
          limits:
            cpu: 100m
            memory: 300Mi
          requests:
            cpu: 100m
            memory: 300Mi
```

**أوامر:**
```bash
# عرض logs للـ cluster autoscaler
kubectl logs -f deployment/cluster-autoscaler -n kube-system

# عرض events
kubectl get events -n kube-system --sort-by='.lastTimestamp'
```

---

## RBAC والأمان

### السؤال 17: اشرح RBAC في Kubernetes؟

**الجواب:**

**RBAC (Role-Based Access Control)** يتحكم في من يمكنه الوصول لماذا:

**المكونات الأساسية:**

1. **Role:** أذونات داخل namespace
2. **ClusterRole:** أذونات على مستوى الكلاستر
3. **RoleBinding:** يربط Role بمستخدم/مجموعة
4. **ClusterRoleBinding:** يربط ClusterRole بمستخدم/مجموعة

**مثال على Role:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: development
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
- apiGroups: [""]
  resources: ["pods/log"]
  verbs: ["get"]
```

**RoleBinding:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: development
subjects:
- kind: User
  name: ahmed
  apiGroup: rbac.authorization.k8s.io
- kind: ServiceAccount
  name: app-sa
  namespace: development
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

**ClusterRole:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-admin-role
rules:
- apiGroups: [""]
  resources: ["*"]
  verbs: ["*"]
- apiGroups: ["apps"]
  resources: ["deployments", "statefulsets", "daemonsets"]
  verbs: ["*"]
- apiGroups: ["batch"]
  resources: ["jobs", "cronjobs"]
  verbs: ["*"]
```

**ClusterRoleBinding:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-binding
subjects:
- kind: User
  name: admin-user
  apiGroup: rbac.authorization.k8s.io
- kind: Group
  name: admin-group
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```

**ServiceAccount:**
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-service-account
  namespace: production
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-role
  namespace: production
rules:
- apiGroups: [""]
  resources: ["secrets", "configmaps"]
  verbs: ["get", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-role-binding
  namespace: production
subjects:
- kind: ServiceAccount
  name: app-service-account
  namespace: production
roleRef:
  kind: Role
  name: app-role
  apiGroup: rbac.authorization.k8s.io
```

**استخدام ServiceAccount في Pod:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
  namespace: production
spec:
  serviceAccountName: app-service-account
  automountServiceAccountToken: true
  containers:
  - name: app
    image: myapp:1.0
```

**Verbs الشائعة:**
- get, list, watch
- create, update, patch, delete
- deletecollection

**أوامر:**
```bash
# عرض roles
kubectl get roles -n development

# عرض clusterroles
kubectl get clusterroles

# عرض rolebindings
kubectl get rolebindings -n development

# عرض تفاصيل role
kubectl describe role pod-reader -n development

# التحقق من الأذونات
kubectl auth can-i create deployments --namespace=development
kubectl auth can-i delete pods --namespace=production --as=ahmed

# عرض ما يمكن لمستخدم فعله
kubectl auth can-i --list --namespace=development --as=ahmed

# إنشاء service account
kubectl create serviceaccount app-sa -n development
```

---

### السؤال 18: ما هي Security Contexts وكيف نستخدمها؟

**الجواab:**

**Security Context** يحدد إعدادات الأمان للـ Pods و Containers:
- تشغيل كمستخدم معين
- منع التشغيل كـ root
- تحديد capabilities
- SELinux options

**Pod Security Context:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
    fsGroupChangePolicy: "OnRootMismatch"
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: app
    image: myapp:1.0
    securityContext:
      allowPrivilegeEscalation: false
      runAsNonRoot: true
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
        add:
        - NET_BIND_SERVICE
    volumeMounts:
    - name: cache
      mountPath: /cache
  volumes:
  - name: cache
    emptyDir: {}
```

**Pod Security Standards:**

**Privileged (غير آمن):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: privileged-pod
spec:
  containers:
  - name: app
    image: myapp:1.0
    securityContext:
      privileged: true
```

**Baseline (أمان متوسط):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: baseline-pod
spec:
  securityContext:
    runAsNonRoot: true
  containers:
  - name: app
    image: myapp:1.0
    securityContext:
      allowPrivilegeEscalation: false
      capabilities:
        drop:
        - ALL
```

**Restricted (أمان عالي):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: restricted-pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: app
    image: myapp:1.0
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      runAsNonRoot: true
      capabilities:
        drop:
        - ALL
```

**Pod Security Policy (مهجور، استخدم Pod Security Admission):**
```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted
spec:
  privileged: false
  allowPrivilegeEscalation: false
  requiredDropCapabilities:
  - ALL
  volumes:
  - 'configMap'
  - 'emptyDir'
  - 'projected'
  - 'secret'
  - 'downwardAPI'
  - 'persistentVolumeClaim'
  runAsUser:
    rule: 'MustRunAsNonRoot'
  seLinux:
    rule: 'RunAsAny'
  fsGroup:
    rule: 'RunAsAny'
  readOnlyRootFilesystem: true
```

**Namespace مع Pod Security Standards:**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

---

## أوامر kubectl

### السؤال 19: ما هي أهم أوامر kubectl التي يجب معرفتها؟

**الجواب:**

**الأوامر الأساسية:**

```bash
# عرض معلومات الكلاستر
kubectl cluster-info
kubectl version

# عرض الموارد
kubectl get pods
kubectl get pods -o wide
kubectl get pods -o yaml
kubectl get pods -o json
kubectl get pods --all-namespaces
kubectl get all

# وصف الموارد
kubectl describe pod <pod-name>
kubectl describe node <node-name>

# إنشاء الموارد
kubectl apply -f manifest.yaml
kubectl apply -f directory/
kubectl apply -f https://url/manifest.yaml
kubectl create deployment nginx --image=nginx

# حذف الموارد
kubectl delete pod <pod-name>
kubectl delete -f manifest.yaml
kubectl delete deployment <deployment-name>
kubectl delete pods --all

# تحديث الموارد
kubectl edit deployment <deployment-name>
kubectl set image deployment/nginx nginx=nginx:1.22
kubectl scale deployment nginx --replicas=5
kubectl patch deployment nginx -p '{"spec":{"replicas":3}}'

# التفاعل مع Pods
kubectl exec -it <pod-name> -- /bin/bash
kubectl exec <pod-name> -- ls /app
kubectl logs <pod-name>
kubectl logs <pod-name> -f
kubectl logs <pod-name> --previous
kubectl logs <pod-name> -c <container-name>

# نسخ الملفات
kubectl cp <pod-name>:/path/to/file /local/path
kubectl cp /local/path <pod-name>:/path/to/file

# Port Forwarding
kubectl port-forward pod/<pod-name> 8080:80
kubectl port-forward service/<service-name> 8080:80

# Labels و Selectors
kubectl get pods -l app=nginx
kubectl get pods -l 'environment in (production,staging)'
kubectl label pod <pod-name> environment=production
kubectl label pod <pod-name> environment-

# Annotations
kubectl annotate pod <pod-name> description="My app"

# Context و Namespace
kubectl config get-contexts
kubectl config use-context <context-name>
kubectl config set-context --current --namespace=<namespace>

# عرض الموارد
kubectl top nodes
kubectl top pods
kubectl api-resources
kubectl explain pod
kubectl explain pod.spec.containers

# Dry Run
kubectl run nginx --image=nginx --dry-run=client -o yaml
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deployment.yaml

# Debugging
kubectl get events
kubectl get events --sort-by='.lastTimestamp'
kubectl debug <pod-name> -it --image=busybox
kubectl attach <pod-name> -it

# Rollouts
kubectl rollout status deployment/<deployment-name>
kubectl rollout history deployment/<deployment-name>
kubectl rollout undo deployment/<deployment-name>
kubectl rollout restart deployment/<deployment-name>

# تصدير YAML
kubectl get deployment nginx -o yaml > deployment.yaml
kubectl get all -o yaml > all-resources.yaml
```

**أوامر متقدمة:**

```bash
# JSONPath
kubectl get pods -o=jsonpath='{.items[*].metadata.name}'
kubectl get pods -o=jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.podIP}{"\n"}{end}'

# Custom Columns
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase,IP:.status.podIP

# عرض موارد محددة
kubectl get pods --field-selector status.phase=Running
kubectl get pods --field-selector metadata.namespace=default

# Wait
kubectl wait --for=condition=Ready pod/<pod-name>
kubectl wait --for=condition=Available deployment/<deployment-name>

# Replace vs Apply
kubectl replace -f manifest.yaml
kubectl replace --force -f manifest.yaml

# Proxy
kubectl proxy --port=8080

# Auth
kubectl auth can-i create deployments
kubectl auth can-i delete pods --namespace=production --as=user

# Plugin
kubectl krew install ctx
kubectl ctx
kubectl ns

# Diff
kubectl diff -f manifest.yaml

# تصحيح الأخطاء
kubectl run debug --image=busybox -it --rm --restart=Never -- sh
kubectl run curl --image=curlimages/curl -it --rm --restart=Never -- curl http://service
```

---

### السؤال 20: كيف تستخدم kubectl للتصدير والنسخ الاحتياطي؟

**الجواب:**

**تصدير جميع الموارد:**

```bash
# تصدير جميع الموارد في namespace
kubectl get all -n production -o yaml > production-backup.yaml

# تصدير موارد محددة
kubectl get deployments,services,configmaps -n production -o yaml > resources.yaml

# تصدير جميع namespaces
for ns in $(kubectl get ns -o jsonpath='{.items[*].metadata.name}'); do
  kubectl get all -n $ns -o yaml > ${ns}-backup.yaml
done

# تصدير CRDs
kubectl get crd -o yaml > crds.yaml

# تصدير RBAC
kubectl get roles,rolebindings,clusterroles,clusterrolebindings -o yaml > rbac.yaml

# تصدير Secrets (احذر!)
kubectl get secrets -n production -o yaml > secrets.yaml

# استخدام Velero للنسخ الاحتياطي
velero backup create my-backup --include-namespaces=production
velero restore create --from-backup my-backup
```

**أدوات مفيدة:**

```bash
# k9s - TUI لـ Kubernetes
k9s

# kubectx و kubens
kubectx  # تبديل contexts
kubens   # تبديل namespaces

# stern - عرض logs من عدة pods
stern app-name

# kustomize
kubectl kustomize overlay/production
kubectl apply -k overlay/production

# helm
helm list
helm install myapp chart/
helm upgrade myapp chart/
```

---

## استكشاف الأخطاء وإصلاحها

### السؤال 21: كيف تستكشف مشاكل Pod الذي لا يعمل؟

**الجواب:**

**خطوات منهجية لاستكشاف الأخطاء:**

**1. التحقق من حالة Pod:**
```bash
kubectl get pods
kubectl get pods -o wide
kubectl describe pod <pod-name>
```

**الحالات الشائعة ومعانيها:**
- **Pending:** لم يتم جدولته على node بعد
- **ImagePullBackOff:** لا يمكن سحب الصورة
- **CrashLoopBackOff:** Container يعيد التشغيل باستمرار
- **Running:** يعمل بشكل طبيعي
- **Error:** Container انتهى بخطأ
- **Completed:** Container أنهى عمله بنجاح

**2. فحص الأحداث:**
```bash
kubectl get events --sort-by='.lastTimestamp'
kubectl get events --field-selector involvedObject.name=<pod-name>
```

**3. فحص Logs:**
```bash
kubectl logs <pod-name>
kubectl logs <pod-name> --previous  # logs من container السابق
kubectl logs <pod-name> -c <container-name>  # multi-container
kubectl logs <pod-name> --tail=100 -f
```

**حل المشاكل الشائعة:**

**ImagePullBackOff:**
```bash
# الأسباب المحتملة:
# - اسم الصورة خاطئ
# - الصورة غير موجودة
# - مشكلة في المصادقة مع registry

# الحل:
kubectl describe pod <pod-name> | grep -A 5 Events

# إنشاء docker registry secret
kubectl create secret docker-registry regcred \
  --docker-server=<registry> \
  --docker-username=<username> \
  --docker-password=<password>

# استخدام secret في Pod
spec:
  imagePullSecrets:
  - name: regcred
  containers:
  - name: app
    image: private-registry/myapp:1.0
```

**CrashLoopBackOff:**
```yaml
# Pod مع أدوات تصحيح أفضل
apiVersion: v1
kind: Pod
metadata:
  name: debug-pod
spec:
  containers:
  - name: app
    image: myapp:1.0
    command: ["sleep", "3600"]  # تجاوز المؤقت للتصحيح
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 30
      periodSeconds: 10
    startupProbe:
      httpGet:
        path: /healthz
        port: 8080
      failureThreshold: 30
      periodSeconds: 10
```

**Pending (مشاكل الجدولة):**
```bash
# التحقق من الموارد المتاحة
kubectl top nodes
kubectl describe nodes

# التحقق من PVC
kubectl get pvc
kubectl describe pvc <pvc-name>

# التحقق من Taints و Tolerations
kubectl describe node <node-name> | grep Taints

# إضافة toleration
spec:
  tolerations:
  - key: "key1"
    operator: "Equal"
    value: "value1"
    effect: "NoSchedule"
```

**OOMKilled (نفاد الذاكرة):**
```yaml
# زيادة memory limits
spec:
  containers:
  - name: app
    image: myapp:1.0
    resources:
      requests:
        memory: "512Mi"
        cpu: "500m"
      limits:
        memory: "1Gi"
        cpu: "1000m"
```

**4. التصحيح التفاعلي:**
```bash
# الدخول إلى Pod
kubectl exec -it <pod-name> -- /bin/bash

# إذا لم يكن bash متوفراً
kubectl exec -it <pod-name> -- /bin/sh

# تشغيل debug container
kubectl debug <pod-name> -it --image=busybox --target=<container-name>

# إنشاء pod تصحيح مؤقت
kubectl run debug --image=busybox -it --rm --restart=Never -- sh
```

---

### السؤال 22: كيف تستكشف مشاكل الشبكة في Kubernetes؟

**الجواب:**

**اختبار الاتصال بين Pods:**

```bash
# إنشاء pod للاختبار
kubectl run test-pod --image=busybox -it --rm --restart=Never -- sh

# من داخل Pod
wget -O- http://service-name:port
nslookup service-name
ping pod-ip

# اختبار DNS
kubectl run dnsutils --image=gcr.io/kubernetes-e2e-test-images/dnsutils:1.3 \
  -it --rm --restart=Never -- nslookup kubernetes.default
```

**التحقق من Services:**

```bash
# عرض services
kubectl get svc

# عرض endpoints
kubectl get endpoints <service-name>

# إذا لم يكن هناك endpoints:
# - تحقق من selector في Service
# - تحقق من labels في Pods

kubectl get pods --show-labels
kubectl describe service <service-name>
```

**مثال تصحيح Service:**
```yaml
# Service
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: myapp  # يجب أن يطابق labels في Pods
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080

---
# Pod مع labels صحيحة
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp  # يجب أن يطابق selector في Service
spec:
  containers:
  - name: app
    image: myapp:1.0
    ports:
    - containerPort: 8080
```

**اختبار Network Policies:**

```bash
# تعطيل network policies مؤقتاً للاختبار
kubectl delete networkpolicy --all -n <namespace>

# اختبار الاتصال
kubectl exec -it source-pod -- curl http://target-service

# تفعيل verbose logging
kubectl exec -it source-pod -- curl -v http://target-service
```

**التحقق من DNS:**

```bash
# اختبار DNS resolution
kubectl run -it --rm debug --image=busybox --restart=Never -- nslookup kubernetes.default

# التحقق من CoreDNS
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns

# تحديث CoreDNS ConfigMap
kubectl edit configmap coredns -n kube-system
```

**اختبار Ingress:**

```bash
# التحقق من Ingress Controller
kubectl get pods -n ingress-nginx

# اختبار من خارج الكلاستر
curl -H "Host: myapp.example.com" http://<ingress-ip>

# فحص logs الـ ingress controller
kubectl logs -n ingress-nginx deployment/ingress-nginx-controller -f

# التحقق من Ingress resources
kubectl describe ingress <ingress-name>
```

**أدوات تصحيح الشبكة:**

```yaml
# Pod مع أدوات شبكة
apiVersion: v1
kind: Pod
metadata:
  name: netshoot
spec:
  containers:
  - name: netshoot
    image: nicolaka/netshoot
    command: ["sleep", "3600"]
```

```bash
# من داخل netshoot pod
kubectl exec -it netshoot -- bash

# أدوات متاحة:
tcpdump
nmap
netstat
ss
iperf3
curl
wget
nslookup
dig
```

---

### السؤال 23: كيف تحل مشاكل PersistentVolumes؟

**الجواب:**

**المشاكل الشائعة والحلول:**

**1. PVC في حالة Pending:**

```bash
# التحقق من حالة PVC
kubectl get pvc
kubectl describe pvc <pvc-name>

# الأسباب المحتملة:
# - لا يوجد PV متاح
# - StorageClass غير موجود
# - لا توجد موارد كافية
```

**الحل:**
```yaml
# التحقق من StorageClass
kubectl get sc

# إنشاء PV يدوياً
apiVersion: v1
kind: PersistentVolume
metadata:
  name: manual-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /mnt/data
```

**2. مشاكل الأذونات:**

```bash
# التحقق من الأذونات داخل Pod
kubectl exec <pod-name> -- ls -la /path/to/mount

# تعيين fsGroup
spec:
  securityContext:
    fsGroup: 2000
  containers:
  - name: app
    volumeMounts:
    - name: data
      mountPath: /data
```

**3. PV عالق في حالة Released:**

```bash
# عرض PVs
kubectl get pv

# حذف PVC القديم
kubectl delete pvc <old-pvc-name>

# تحرير PV وإزالة claimRef
kubectl patch pv <pv-name> -p '{"spec":{"claimRef": null}}'

# أو تحرير يدوياً
kubectl edit pv <pv-name>
# احذف السطور:
#   claimRef:
#     apiVersion: v1
#     kind: PersistentVolumeClaim
#     name: old-pvc
#     namespace: default
```

**4. مشاكل تغيير الحجم:**

```bash
# التحقق من allowVolumeExpansion
kubectl get sc -o yaml | grep allowVolumeExpansion

# تفعيل volume expansion
kubectl patch sc <storage-class-name> \
  -p '{"allowVolumeExpansion": true}'

# تحديث حجم PVC
kubectl edit pvc <pvc-name>
# غيّر storage: 10Gi إلى storage: 20Gi

# إعادة تشغيل Pod
kubectl delete pod <pod-name>
```

**Pod مع PVC troubleshooting:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pvc-test
spec:
  containers:
  - name: test
    image: busybox
    command: ["sleep", "3600"]
    volumeMounts:
    - name: test-volume
      mountPath: /test
  volumes:
  - name: test-volume
    persistentVolumeClaim:
      claimName: test-pvc
```

---

## Helm

### السؤال 24: ما هو Helm وما هي مكوناته الأساسية؟

**الجواب:**

**Helm** هو package manager لـ Kubernetes:
- يبسط نشر التطبيقات المعقدة
- يوفر versioning و rollback
- يسمح بمشاركة التكوينات

**المكونات الأساسية:**

1. **Chart:** حزمة تحتوي على ملفات YAML
2. **Release:** instance من chart منشور في الكلاستر
3. **Repository:** مكان تخزين Charts

**تثبيت Helm:**
```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# التحقق من التثبيت
helm version
```

**الأوامر الأساسية:**

```bash
# إضافة repository
helm repo add stable https://charts.helm.sh/stable
helm repo add bitnami https://charts.bitnami.com/bitnami

# تحديث repositories
helm repo update

# البحث عن charts
helm search repo nginx
helm search hub wordpress

# تثبيت chart
helm install myapp bitnami/nginx
helm install myapp bitnami/nginx --namespace production --create-namespace

# تثبيت مع custom values
helm install myapp bitnami/nginx -f custom-values.yaml
helm install myapp bitnami/nginx --set service.type=LoadBalancer

# عرض releases
helm list
helm list --all-namespaces

# عرض معلومات release
helm status myapp
helm get values myapp
helm get manifest myapp

# ترقية release
helm upgrade myapp bitnami/nginx
helm upgrade myapp bitnami/nginx -f new-values.yaml

# التراجع
helm rollback myapp
helm rollback myapp 2  # إلى revision محدد

# حذف release
helm uninstall myapp
helm uninstall myapp --keep-history

# عرض تاريخ release
helm history myapp
```

**بنية Helm Chart:**

```
mychart/
├── Chart.yaml          # معلومات عن الchart
├── values.yaml         # القيم الافتراضية
├── charts/             # chart dependencies
├── templates/          # قوالب Kubernetes
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── _helpers.tpl   # template helpers
│   └── NOTES.txt      # ملاحظات بعد التثبيت
└── .helmignore        # ملفات للتجاهل
```

**Chart.yaml:**
```yaml
apiVersion: v2
name: myapp
description: A Helm chart for my application
type: application
version: 1.0.0
appVersion: "1.0"
maintainers:
- name: Your Name
  email: email@example.com
dependencies:
- name: postgresql
  version: 12.x.x
  repository: https://charts.bitnami.com/bitnami
  condition: postgresql.enabled
```

**values.yaml:**
```yaml
replicaCount: 3

image:
  repository: myapp
  pullPolicy: IfNotPresent
  tag: "1.0"

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: true
  className: nginx
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
  hosts:
  - host: myapp.example.com
    paths:
    - path: /
      pathType: Prefix
  tls:
  - secretName: myapp-tls
    hosts:
    - myapp.example.com

resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi

autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70

postgresql:
  enabled: true
  auth:
    username: myapp
    password: password
    database: myappdb
```

**templates/deployment.yaml:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "myapp.fullname" . }}
  labels:
    {{- include "myapp.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  selector:
    matchLabels:
      {{- include "myapp.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "myapp.selectorLabels" . | nindent 8 }}
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - name: http
          containerPort: 8080
          protocol: TCP
        resources:
          {{- toYaml .Values.resources | nindent 12 }}
```

**templates/_helpers.tpl:**
```yaml
{{- define "myapp.fullname" -}}
{{- printf "%s-%s" .Release.Name .Chart.Name | trunc 63 | trimSuffix "-" }}
{{- end }}

{{- define "myapp.labels" -}}
helm.sh/chart: {{ .Chart.Name }}-{{ .Chart.Version }}
app.kubernetes.io/name: {{ include "myapp.fullname" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
app.kubernetes.io/version: {{ .Chart.AppVersion }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}

{{- define "myapp.selectorLabels" -}}
app.kubernetes.io/name: {{ include "myapp.fullname" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}
```

---

### السؤال 25: كيف تنشئ وتدير Helm Charts؟

**الجواب:**

**إنشاء Chart جديد:**

```bash
# إنشاء chart
helm create myapp

# التحقق من صحة chart
helm lint myapp/

# اختبار rendering بدون تثبيت
helm template myapp myapp/
helm template myapp myapp/ -f custom-values.yaml

# dry run
helm install myapp myapp/ --dry-run --debug

# package chart
helm package myapp/

# إنشاء index file لـ repository
helm repo index .
```

**إدارة Dependencies:**

```bash
# تحديث dependencies
helm dependency update myapp/

# عرض dependencies
helm dependency list myapp/
```

**Hooks في Helm:**

```yaml
# templates/pre-install-job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: {{ include "myapp.fullname" . }}-migration
  annotations:
    "helm.sh/hook": pre-install,pre-upgrade
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": before-hook-creation
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: migration
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        command: ["python", "manage.py", "migrate"]
```

**أنواع Hooks:**
- pre-install
- post-install
- pre-delete
- post-delete
- pre-upgrade
- post-upgrade
- pre-rollback
- post-rollback
- test

**Helm Tests:**

```yaml
# templates/tests/test-connection.yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "myapp.fullname" . }}-test-connection"
  annotations:
    "helm.sh/hook": test
spec:
  containers:
  - name: wget
    image: busybox
    command: ['wget']
    args: ['{{ include "myapp.fullname" . }}:{{ .Values.service.port }}']
  restartPolicy: Never
```

```bash
# تشغيل tests
helm test myapp
```

**استخدام متقدم:**

```bash
# تثبيت مع timeout
helm install myapp myapp/ --timeout 10m

# تثبيت مع wait
helm install myapp myapp/ --wait

# تثبيت مع atomic (rollback عند الفشل)
helm install myapp myapp/ --atomic

# عرض القيم المحسوبة
helm get values myapp --all

# استخراج chart
helm pull bitnami/nginx
helm pull bitnami/nginx --untar

# إنشاء chart repository
# 1. package charts
helm package myapp/
# 2. create index
helm repo index . --url https://charts.example.com
# 3. upload to web server
```

**custom-values.yaml للبيئات المختلفة:**

```yaml
# values-dev.yaml
replicaCount: 1
image:
  tag: "dev"
resources:
  limits:
    cpu: 200m
    memory: 256Mi

# values-prod.yaml
replicaCount: 5
image:
  tag: "1.0.0"
resources:
  limits:
    cpu: 1000m
    memory: 1Gi
autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 20
```

```bash
# التثبيت للبيئات المختلفة
helm install myapp-dev myapp/ -f values-dev.yaml -n development
helm install myapp-prod myapp/ -f values-prod.yaml -n production
```

---

## سيناريوهات عملية

### السؤال 26: كيف تنشر تطبيق متكامل مع قاعدة بيانات؟

**الجواب:**

**السيناريو:** نشر تطبيق web مع PostgreSQL database

**1. إنشاء Namespace:**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: myapp
  labels:
    environment: production
```

**2. Database Secrets:**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
  namespace: myapp
type: Opaque
stringData:
  POSTGRES_USER: myapp
  POSTGRES_PASSWORD: StrongPassword123!
  POSTGRES_DB: myappdb
```

**3. Database ConfigMap:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: db-config
  namespace: myapp
data:
  POSTGRES_MAX_CONNECTIONS: "100"
  POSTGRES_SHARED_BUFFERS: "256MB"
```

**4. PersistentVolumeClaim للـ Database:**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
  namespace: myapp
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
  storageClassName: fast-storage
```

**5. PostgreSQL StatefulSet:**
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
  namespace: myapp
spec:
  serviceName: postgres
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:15
        ports:
        - containerPort: 5432
          name: postgres
        envFrom:
        - secretRef:
            name: db-secret
        - configMapRef:
            name: db-config
        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
          subPath: postgres
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          exec:
            command:
            - pg_isready
            - -U
            - myapp
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          exec:
            command:
            - pg_isready
            - -U
            - myapp
          initialDelaySeconds: 5
          periodSeconds: 5
      volumes:
      - name: postgres-storage
        persistentVolumeClaim:
          claimName: postgres-pvc
```

**6. Database Service:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres
  namespace: myapp
spec:
  selector:
    app: postgres
  ports:
  - port: 5432
    targetPort: 5432
  clusterIP: None  # Headless service للـ StatefulSet
```

**7. Application ConfigMap:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: myapp
data:
  LOG_LEVEL: "info"
  PORT: "8080"
  DATABASE_HOST: "postgres.myapp.svc.cluster.local"
  DATABASE_PORT: "5432"
```

**8. Application Deployment:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  namespace: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      initContainers:
      - name: wait-for-db
        image: busybox:1.28
        command: ['sh', '-c', 'until nc -z postgres 5432; do echo waiting for db; sleep 2; done;']
      containers:
      - name: web-app
        image: myapp:1.0.0
        ports:
        - containerPort: 8080
        env:
        - name: DATABASE_USER
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: POSTGRES_USER
        - name: DATABASE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: POSTGRES_PASSWORD
        - name: DATABASE_NAME
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: POSTGRES_DB
        envFrom:
        - configMapRef:
            name: app-config
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
        lifecycle:
          preStop:
            exec:
              command: ["/bin/sh", "-c", "sleep 15"]
```

**9. Application Service:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app-service
  namespace: myapp
spec:
  selector:
    app: web-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: ClusterIP
```

**10. Ingress:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-app-ingress
  namespace: myapp
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    nginx.ingress.kubernetes.io/rate-limit: "100"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - app.example.com
    secretName: app-tls
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-app-service
            port:
              number: 80
```

**11. HPA:**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-app-hpa
  namespace: myapp
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app
  minReplicas: 3
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

**نشر كل شيء:**
```bash
kubectl apply -f namespace.yaml
kubectl apply -f db-secret.yaml
kubectl apply -f db-config.yaml
kubectl apply -f postgres-pvc.yaml
kubectl apply -f postgres-statefulset.yaml
kubectl apply -f postgres-service.yaml
kubectl apply -f app-config.yaml
kubectl apply -f app-deployment.yaml
kubectl apply -f app-service.yaml
kubectl apply -f ingress.yaml
kubectl apply -f hpa.yaml

# التحقق
kubectl get all -n myapp
kubectl get pvc -n myapp
kubectl get ingress -n myapp
```

---

### السؤال 27: كيف تنفذ Blue-Green Deployment؟

**الجواب:**

**Blue-Green Deployment** يسمح بالتبديل الفوري بين إصدارين:

**1. Blue Deployment (الإصدار الحالي):**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-blue
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: blue
  template:
    metadata:
      labels:
        app: myapp
        version: blue
    spec:
      containers:
      - name: app
        image: myapp:1.0.0
        ports:
        - containerPort: 8080
```

**2. Green Deployment (الإصدار الجديد):**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-green
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: green
  template:
    metadata:
      labels:
        app: myapp
        version: green
    spec:
      containers:
      - name: app
        image: myapp:2.0.0
        ports:
        - containerPort: 8080
```

**3. Service (يشير إلى Blue):**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
  namespace: production
spec:
  selector:
    app: myapp
    version: blue  # يشير إلى النسخة الزرقاء
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

**الخطوات:**

```bash
# 1. نشر Blue (الحالي)
kubectl apply -f blue-deployment.yaml
kubectl apply -f service.yaml

# 2. نشر Green (الجديد)
kubectl apply -f green-deployment.yaml

# 3. اختبار Green
kubectl port-forward deployment/app-green 8080:8080
# اختبر على http://localhost:8080

# 4. التبديل إلى Green
kubectl patch service myapp-service -p '{"spec":{"selector":{"version":"green"}}}'

# 5. مراقبة الأداء
kubectl get pods -l app=myapp -w
kubectl logs -l app=myapp,version=green -f

# 6. في حالة وجود مشكلة - العودة إلى Blue
kubectl patch service myapp-service -p '{"spec":{"selector":{"version":"blue"}}}'

# 7. بعد التأكد من النجاح - حذف Blue
kubectl delete deployment app-blue
```

**استخدام Labels متقدم:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
  namespace: production
  labels:
    active-version: blue
spec:
  selector:
    app: myapp
    version: blue
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

---

### السؤال 28: كيف تنفذ Canary Deployment؟

**الجواب:**

**Canary Deployment** ينشر الإصدار الجديد تدريجياً:

**1. Stable Deployment:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-stable
  namespace: production
spec:
  replicas: 9
  selector:
    matchLabels:
      app: myapp
      track: stable
  template:
    metadata:
      labels:
        app: myapp
        track: stable
    spec:
      containers:
      - name: app
        image: myapp:1.0.0
        ports:
        - containerPort: 8080
```

**2. Canary Deployment:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-canary
  namespace: production
spec:
  replicas: 1  # 10% من الحركة
  selector:
    matchLabels:
      app: myapp
      track: canary
  template:
    metadata:
      labels:
        app: myapp
        track: canary
    spec:
      containers:
      - name: app
        image: myapp:2.0.0
        ports:
        - containerPort: 8080
```

**3. Service (يشير للاثنين):**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
  namespace: production
spec:
  selector:
    app: myapp  # لا يحدد track
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

**الخطوات التدريجية:**

```bash
# 1. نشر stable
kubectl apply -f stable-deployment.yaml

# 2. نشر canary بـ 10%
kubectl apply -f canary-deployment.yaml

# 3. مراقبة metrics
kubectl logs -l app=myapp,track=canary -f
kubectl top pods -l app=myapp,track=canary

# 4. زيادة canary تدريجياً
kubectl scale deployment app-canary --replicas=2  # 20%
kubectl scale deployment app-stable --replicas=8

# استمر في الزيادة
kubectl scale deployment app-canary --replicas=5  # 50%
kubectl scale deployment app-stable --replicas=5

# 5. إذا كان كل شيء جيد - التبديل الكامل
kubectl scale deployment app-canary --replicas=10
kubectl scale deployment app-stable --replicas=0

# 6. حذف stable deployment
kubectl delete deployment app-stable

# 7. إعادة تسمية canary
kubectl set image deployment/app-canary app=myapp:2.0.0
kubectl patch deployment app-canary -p '{"spec":{"selector":{"matchLabels":{"track":"stable"}}}}'
```

**Canary مع Istio (متقدم):**
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: myapp-vs
spec:
  hosts:
  - myapp-service
  http:
  - match:
    - headers:
        canary:
          exact: "true"
    route:
    - destination:
        host: myapp-service
        subset: canary
  - route:
    - destination:
        host: myapp-service
        subset: stable
      weight: 90
    - destination:
        host: myapp-service
        subset: canary
      weight: 10
```

---

### السؤال 29: كيف تتعامل مع إدارة Secrets بشكل آمن في Production؟

**الجواب:**

**المشكلة:** تخزين Secrets في Git غير آمن

**الحلول:**

**1. استخدام Sealed Secrets:**

```bash
# تثبيت Sealed Secrets Controller
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.24.0/controller.yaml

# تثبيت kubeseal CLI
brew install kubeseal
```

```bash
# إنشاء secret عادي
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=secretpass \
  --dry-run=client -o yaml > secret.yaml

# تحويله إلى sealed secret
kubeseal -f secret.yaml -w sealed-secret.yaml

# الآن يمكن commit sealed-secret.yaml بأمان
git add sealed-secret.yaml
git commit -m "Add sealed secret"
```

**sealed-secret.yaml (آمن للـ Git):**
```yaml
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  name: db-secret
  namespace: production
spec:
  encryptedData:
    username: AgBh8F...  # مشفر
    password: AgCy9D...  # مشفر
  template:
    metadata:
      name: db-secret
      namespace: production
```

**2. استخدام External Secrets Operator:**

```bash
# تثبيت External Secrets Operator
helm repo add external-secrets https://charts.external-secrets.io
helm install external-secrets external-secrets/external-secrets -n external-secrets-system --create-namespace
```

```yaml
# SecretStore (اتصال بـ AWS Secrets Manager)
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: aws-secret-store
  namespace: production
spec:
  provider:
    aws:
      service: SecretsManager
      region: us-east-1
      auth:
        jwt:
          serviceAccountRef:
            name: external-secrets-sa

---
# ExternalSecret (جلب من AWS)
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: db-credentials
  namespace: production
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secret-store
    kind: SecretStore
  target:
    name: db-secret
    creationPolicy: Owner
  data:
  - secretKey: username
    remoteRef:
      key: prod/db/credentials
      property: username
  - secretKey: password
    remoteRef:
      key: prod/db/credentials
      property: password
```

**3. استخدام HashiCorp Vault:**

```yaml
# Vault Agent Injector
apiVersion: v1
kind: Pod
metadata:
  name: app-with-vault
  annotations:
    vault.hashicorp.com/agent-inject: "true"
    vault.hashicorp.com/role: "myapp"
    vault.hashicorp.com/agent-inject-secret-db: "secret/data/database/config"
    vault.hashicorp.com/agent-inject-template-db: |
      {{- with secret "secret/data/database/config" -}}
      export DB_USERNAME="{{ .Data.data.username }}"
      export DB_PASSWORD="{{ .Data.data.password }}"
      {{- end }}
spec:
  serviceAccountName: myapp
  containers:
  - name: app
    image: myapp:1.0
    command: ["/bin/sh"]
    args: ["-c", "source /vault/secrets/db && ./app"]
```

**4. Encryption at Rest:**

```yaml
# EncryptionConfiguration
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
- resources:
  - secrets
  providers:
  - aescbc:
      keys:
      - name: key1
        secret: <base64-encoded-32-byte-key>
  - identity: {}
```

**Best Practices:**

```bash
# 1. عدم تخزين secrets في Git أبداً
echo "*.secret.yaml" >> .gitignore

# 2. استخدام RBAC للحد من الوصول
kubectl create role secret-reader \
  --verb=get,list \
  --resource=secrets \
  -n production

# 3. Rotation منتظم للـ secrets
# 4. Audit logging للوصول إلى secrets
# 5. استخدام separate namespaces
```

---

### السؤال 30: كيف تنفذ Disaster Recovery و Backup Strategy؟

**الجواب:**

**استراتيجية شاملة للنسخ الاحتياطي:**

**1. استخدام Velero:**

```bash
# تثبيت Velero
velero install \
  --provider aws \
  --plugins velero/velero-plugin-for-aws:v1.8.0 \
  --bucket velero-backups \
  --backup-location-config region=us-east-1 \
  --snapshot-location-config region=us-east-1 \
  --secret-file ./credentials-velero
```

**Backup كامل:**
```bash
# backup لكل الكلاستر
velero backup create full-backup

# backup لـ namespace معين
velero backup create app-backup --include-namespaces production

# backup مجدول
velero schedule create daily-backup \
  --schedule="0 2 * * *" \
  --include-namespaces production,staging

# backup مع PV snapshots
velero backup create pv-backup \
  --include-namespaces production \
  --snapshot-volumes
```

**Restore:**
```bash
# عرض backups
velero backup get

# restore من backup
velero restore create --from-backup full-backup

# restore لـ namespace معين فقط
velero restore create --from-backup full-backup \
  --include-namespaces production

# restore مع namespace جديد
velero restore create --from-backup app-backup \
  --namespace-mappings old-ns:new-ns
```

**2. Backup لـ etcd:**

```bash
# Backup يدوي
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# التحقق من backup
ETCDCTL_API=3 etcdctl snapshot status /backup/etcd-snapshot.db

# Restore
ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-snapshot.db \
  --data-dir=/var/lib/etcd-restore
```

**CronJob لـ etcd backup:**
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: etcd-backup
  namespace: kube-system
spec:
  schedule: "0 */6 * * *"  # كل 6 ساعات
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: k8s.gcr.io/etcd:3.5.9-0
            command:
            - /bin/sh
            - -c
            - |
              ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-$(date +%Y%m%d-%H%M%S).db \
                --endpoints=https://127.0.0.1:2379 \
                --cacert=/etc/kubernetes/pki/etcd/ca.crt \
                --cert=/etc/kubernetes/pki/etcd/server.crt \
                --key=/etc/kubernetes/pki/etcd/server.key

              # Upload to S3
              aws s3 cp /backup/etcd-*.db s3://my-etcd-backups/

              # حذف backups القديمة (أكثر من 7 أيام)
              find /backup -name "etcd-*.db" -mtime +7 -delete
            volumeMounts:
            - name: etcd-certs
              mountPath: /etc/kubernetes/pki/etcd
              readOnly: true
            - name: backup
              mountPath: /backup
          volumes:
          - name: etcd-certs
            hostPath:
              path: /etc/kubernetes/pki/etcd
          - name: backup
            persistentVolumeClaim:
              claimName: etcd-backup-pvc
          restartPolicy: OnFailure
```

**3. Backup لـ PersistentVolumes:**

```yaml
# VolumeSnapshot
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: db-snapshot
  namespace: production
spec:
  volumeSnapshotClassName: csi-snapshot-class
  source:
    persistentVolumeClaimName: postgres-pvc
```

**CronJob للـ snapshots:**
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: database-snapshot
  namespace: production
spec:
  schedule: "0 1 * * *"  # يومياً الساعة 1 صباحاً
  jobTemplate:
    spec:
      template:
        spec:
          serviceAccountName: snapshot-creator
          containers:
          - name: create-snapshot
            image: bitnami/kubectl:latest
            command:
            - /bin/sh
            - -c
            - |
              kubectl create -f - <<EOF
              apiVersion: snapshot.storage.k8s.io/v1
              kind: VolumeSnapshot
              metadata:
                name: db-snapshot-$(date +%Y%m%d-%H%M%S)
                namespace: production
              spec:
                volumeSnapshotClassName: csi-snapshot-class
                source:
                  persistentVolumeClaimName: postgres-pvc
              EOF
          restartPolicy: OnFailure
```

**4. Disaster Recovery Plan:**

```yaml
# DR Namespace في cluster آخر
apiVersion: v1
kind: Namespace
metadata:
  name: production-dr
  labels:
    environment: disaster-recovery

---
# Velero restore schedule
apiVersion: velero.io/v1
kind: Schedule
metadata:
  name: production-sync
  namespace: velero
spec:
  schedule: "0 */4 * * *"
  template:
    includedNamespaces:
    - production
    storageLocation: aws-dr-location
```

**Testing DR:**
```bash
# 1. Simulate disaster
kubectl delete namespace production

# 2. Restore من backup
velero restore create dr-test --from-backup full-backup

# 3. التحقق
kubectl get all -n production

# 4. اختبار التطبيق
curl https://app.example.com/health
```

**Checklist للـ DR:**
```bash
# إنشاء DR runbook
cat > dr-runbook.md <<EOF
# Disaster Recovery Runbook

## 1. Assessment (5 دقائق)
- تحديد نطاق المشكلة
- التواصل مع الفريق

## 2. Restore من Backup (15-30 دقيقة)
\`\`\`bash
velero restore create --from-backup latest-backup
\`\`\`

## 3. Database Recovery (10-20 دقيقة)
\`\`\`bash
kubectl apply -f postgres-restore.yaml
\`\`\`

## 4. Verification (10 دقائق)
- اختبار endpoints
- التحقق من البيانات
- مراقبة logs

## 5. DNS Switchover (5 دقائق)
- تحديث DNS للـ DR cluster

## 6. Post-Recovery
- تحليل السبب الجذري
- تحديث DR plan
EOF
```

**Monitoring للـ Backups:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: backup-monitoring
data:
  check-backups.sh: |
    #!/bin/bash
    # التحقق من أن آخر backup كان خلال 24 ساعة
    LAST_BACKUP=$(velero backup get --output json | jq -r '.[0].status.completionTimestamp')
    CURRENT_TIME=$(date -u +%s)
    BACKUP_TIME=$(date -d "$LAST_BACKUP" +%s)
    DIFF=$((CURRENT_TIME - BACKUP_TIME))

    if [ $DIFF -gt 86400 ]; then
      echo "WARNING: Last backup is older than 24 hours!"
      # إرسال تنبيه
      curl -X POST https://alerts.example.com/webhook \
        -d '{"text":"Backup is outdated!"}'
      exit 1
    fi
```

---

## ملخص نصائح المقابلة

### نصائح عامة للإجابة على أسئلة Kubernetes:

1. **ابدأ بالأساسيات** ثم انتقل للتفاصيل المتقدمة
2. **استخدم أمثلة YAML** عملية في إجاباتك
3. **اذكر kubectl commands** ذات الصلة
4. **تحدث عن Best Practices** والأمان
5. **اربط الإجابة بسيناريوهات حقيقية** من خبرتك
6. **اذكر المشاكل الشائعة** وكيفية حلها
7. **كن مستعداً للأسئلة المتابعة** المتعمقة
8. **أظهر فهمك للـ trade-offs** بين الخيارات المختلفة

### الموضوعات الأكثر أهمية للمراجعة:

- Pods lifecycle و troubleshooting
- Deployments و rolling updates
- Services و networking
- Storage (PV, PVC, StorageClass)
- RBAC و security
- Helm charts
- Monitoring و logging
- Backup و disaster recovery

---

**حظاً موفقاً في مقابلتك!**
