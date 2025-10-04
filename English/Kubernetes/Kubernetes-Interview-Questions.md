# Kubernetes Interview Questions & Answers

A comprehensive guide covering Kubernetes concepts, architecture, components, and real-world scenarios with practical examples.

## Table of Contents

1. [Architecture Questions](#architecture-questions)
2. [Pods](#pods)
3. [Deployments](#deployments)
4. [Services](#services)
5. [ConfigMaps and Secrets](#configmaps-and-secrets)
6. [Volumes and PersistentVolumes](#volumes-and-persistentvolumes)
7. [Namespaces](#namespaces)
8. [Ingress](#ingress)
9. [Auto-scaling (HPA)](#auto-scaling-hpa)
10. [RBAC (Role-Based Access Control)](#rbac-role-based-access-control)
11. [kubectl Commands](#kubectl-commands)
12. [Troubleshooting](#troubleshooting)
13. [Helm](#helm)
14. [Real-World Scenarios](#real-world-scenarios)

---

## Architecture Questions

### Q1: Explain Kubernetes architecture and its main components

**Answer:**

Kubernetes follows a master-worker (control plane-worker node) architecture:

**Control Plane Components:**

1. **API Server (kube-apiserver)**
   - Central management entity
   - Exposes Kubernetes REST API
   - Entry point for all administrative tasks
   - Validates and processes API requests

2. **etcd**
   - Distributed key-value store
   - Stores all cluster data and state
   - Provides backup and disaster recovery
   - Consistent and highly-available

3. **Scheduler (kube-scheduler)**
   - Watches for newly created pods
   - Assigns pods to nodes based on:
     - Resource requirements
     - Hardware/software constraints
     - Affinity/anti-affinity rules
     - Data locality

4. **Controller Manager (kube-controller-manager)**
   - Runs controller processes:
     - Node Controller: Monitors node health
     - Replication Controller: Maintains correct pod count
     - Endpoints Controller: Populates endpoints
     - Service Account Controller: Creates default accounts

5. **Cloud Controller Manager** (optional)
   - Integrates with cloud provider APIs
   - Manages cloud-specific resources

**Worker Node Components:**

1. **Kubelet**
   - Agent running on each node
   - Ensures containers are running in pods
   - Reports node and pod status to API server
   - Executes pod lifecycle operations

2. **Kube-proxy**
   - Network proxy on each node
   - Maintains network rules
   - Handles service load balancing
   - Enables pod-to-pod communication

3. **Container Runtime**
   - Software for running containers
   - Examples: Docker, containerd, CRI-O
   - Pulls images and runs containers

### Q2: What is the difference between Control Plane and Worker Nodes?

**Answer:**

| Control Plane | Worker Nodes |
|---------------|--------------|
| Manages cluster state | Runs application workloads |
| Makes scheduling decisions | Executes scheduled pods |
| Detects and responds to events | Reports status to control plane |
| Runs: API Server, Scheduler, Controller Manager, etcd | Runs: Kubelet, Kube-proxy, Container Runtime |
| Typically 3+ nodes for HA | Can have unlimited nodes |
| Houses cluster brain | Houses application containers |

### Q3: How does communication work in Kubernetes?

**Answer:**

**Pod-to-Pod Communication:**
- Each pod gets unique IP address
- Pods can communicate directly without NAT
- Flat network model across nodes

**Pod-to-Service Communication:**
- Services provide stable IP/DNS
- kube-proxy maintains iptables/IPVS rules
- Load balances traffic to pod endpoints

**External-to-Service Communication:**
- NodePort: Exposes service on node IP:port
- LoadBalancer: Cloud provider load balancer
- Ingress: HTTP/HTTPS routing

**Control Plane Communication:**
- All components communicate via API Server
- API Server is the only component that talks to etcd
- Kubelet initiates connection to API Server

---

## Pods

### Q4: What is a Pod and why is it the smallest unit in Kubernetes?

**Answer:**

A **Pod** is the smallest deployable unit in Kubernetes representing one or more containers that should run together.

**Key Characteristics:**
- Atomic unit of deployment
- Containers in a pod share:
  - Network namespace (same IP address)
  - IPC namespace
  - UTS namespace (hostname)
  - Optionally, PID namespace
- Containers share storage volumes
- Scheduled together on same node
- Scale together as a unit

**Why Pods?**
- Encapsulates application containers
- Enables tight coupling of helper containers (sidecars)
- Simplifies networking and storage sharing
- Represents single instance of application

**Example:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
    tier: frontend
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

### Q5: What are the different phases of a Pod lifecycle?

**Answer:**

1. **Pending**: Pod accepted but containers not yet created
   - Waiting for scheduling
   - Pulling images

2. **Running**: Pod bound to node, containers created
   - At least one container is running
   - Or starting/restarting

3. **Succeeded**: All containers terminated successfully
   - Won't be restarted
   - Typical for Jobs

4. **Failed**: All containers terminated, at least one failed
   - Exited with non-zero status
   - Or terminated by system

5. **Unknown**: Pod state cannot be obtained
   - Communication error with node

**Check pod status:**
```bash
kubectl get pods
kubectl describe pod <pod-name>
```

### Q6: Explain Pod design patterns

**Answer:**

**1. Sidecar Pattern**
- Main container + helper container
- Helper enhances main container functionality

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-app-with-logging
spec:
  containers:
  - name: web-app
    image: nginx
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/nginx
  - name: log-shipper
    image: fluent/fluentd
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/nginx
  volumes:
  - name: shared-logs
    emptyDir: {}
```

**2. Ambassador Pattern**
- Proxy container for main container
- Simplifies network communication

**3. Adapter Pattern**
- Transforms output of main container
- Standardizes monitoring/logging

### Q7: What are Init Containers?

**Answer:**

Init containers run before app containers and always run to completion.

**Use Cases:**
- Wait for service to be available
- Clone git repository
- Generate configuration files
- Security/permission setup

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done']
  - name: init-mydb
    image: busybox
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done']
  containers:
  - name: myapp
    image: myapp:1.0
```

### Q8: How do you implement health checks in Pods?

**Answer:**

**Three types of probes:**

**1. Liveness Probe**
- Determines if container is running
- Restarts container if probe fails

**2. Readiness Probe**
- Determines if container is ready to serve traffic
- Removes pod from service endpoints if fails

**3. Startup Probe**
- For slow-starting containers
- Disables liveness/readiness until succeeds

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: health-check-pod
spec:
  containers:
  - name: webapp
    image: myapp:1.0
    ports:
    - containerPort: 8080
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 30
      periodSeconds: 10
      timeoutSeconds: 5
      failureThreshold: 3
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
      successThreshold: 1
    startupProbe:
      httpGet:
        path: /startup
        port: 8080
      failureThreshold: 30
      periodSeconds: 10
```

**Probe Handlers:**
- **exec**: Executes command in container
- **httpGet**: HTTP GET request
- **tcpSocket**: TCP connection test

---

## Deployments

### Q9: What is a Deployment and how does it differ from a ReplicaSet?

**Answer:**

**Deployment:**
- Higher-level abstraction managing ReplicaSets
- Provides declarative updates
- Supports rolling updates and rollbacks
- Manages multiple ReplicaSet versions

**ReplicaSet:**
- Ensures specified number of pod replicas running
- Lower-level primitive
- Rarely created directly by users

**Relationship:**
```
Deployment → ReplicaSet → Pods
```

**Deployment Example:**

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
        resources:
          requests:
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "128Mi"
            cpu: "200m"
```

### Q10: Explain deployment strategies in Kubernetes

**Answer:**

**1. Rolling Update (Default)**
- Gradually replaces old pods with new ones
- Zero downtime
- Can control pace with maxSurge and maxUnavailable

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 25%        # Max pods above desired count
    maxUnavailable: 25%  # Max pods unavailable during update
```

**2. Recreate**
- Terminates all old pods before creating new ones
- Downtime during update
- Useful when versions cannot coexist

```yaml
strategy:
  type: Recreate
```

**3. Blue/Green Deployment**
- Deploy new version alongside old
- Switch traffic when ready
- Implemented using Services and labels

**4. Canary Deployment**
- Route small traffic percentage to new version
- Gradually increase if successful
- Implemented using multiple Deployments

### Q11: How do you perform rollback in Kubernetes?

**Answer:**

**View rollout history:**
```bash
kubectl rollout history deployment/nginx-deployment
```

**Rollback to previous version:**
```bash
kubectl rollout undo deployment/nginx-deployment
```

**Rollback to specific revision:**
```bash
kubectl rollout undo deployment/nginx-deployment --to-revision=2
```

**Check rollout status:**
```bash
kubectl rollout status deployment/nginx-deployment
```

**Pause/Resume rollout:**
```bash
kubectl rollout pause deployment/nginx-deployment
kubectl rollout resume deployment/nginx-deployment
```

**Example with revision history:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  revisionHistoryLimit: 10  # Keep 10 old ReplicaSets
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
      annotations:
        kubernetes.io/change-cause: "Update to nginx 1.21"
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
```

### Q12: What are DaemonSets and when would you use them?

**Answer:**

**DaemonSet** ensures all (or specific) nodes run a copy of a pod.

**Use Cases:**
- Log collectors (Fluentd, Logstash)
- Monitoring agents (Prometheus Node Exporter)
- Storage daemons (Ceph, Gluster)
- Network plugins (Calico, Weave)

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
```

---

## Services

### Q13: What is a Service and why is it needed?

**Answer:**

**Service** provides stable networking for a set of pods.

**Why needed:**
- Pods are ephemeral (can be destroyed/recreated)
- Pod IPs change when recreated
- Need stable endpoint for discovery
- Load balancing across pod replicas

**Service acts as:**
- Stable IP address
- DNS name
- Load balancer
- Service discovery mechanism

### Q14: Explain different types of Services

**Answer:**

**1. ClusterIP (Default)**
- Exposes service on cluster-internal IP
- Only accessible within cluster
- Used for internal communication

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
    port: 80        # Service port
    targetPort: 8080  # Container port
```

**2. NodePort**
- Exposes service on each node's IP at static port
- Accessible from outside cluster: `<NodeIP>:<NodePort>`
- Port range: 30000-32767

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: NodePort
  selector:
    app: web
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
    nodePort: 30080  # Optional: K8s assigns if omitted
```

**3. LoadBalancer**
- Creates external load balancer (cloud provider)
- Assigns external IP
- Automatically creates NodePort and ClusterIP

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  type: LoadBalancer
  selector:
    app: frontend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  loadBalancerIP: 78.11.24.19  # Optional: specific IP
```

**4. ExternalName**
- Maps service to DNS name
- No proxy, just DNS CNAME
- Used for external services

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-database
spec:
  type: ExternalName
  externalName: database.example.com
```

**5. Headless Service**
- No cluster IP assigned (clusterIP: None)
- Returns pod IPs directly
- Used for stateful applications

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-headless
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
  - port: 3306
```

### Q15: How does Service discovery work in Kubernetes?

**Answer:**

**Two methods:**

**1. Environment Variables**
- Kubernetes injects env vars for each service
- Format: `<SERVICE_NAME>_SERVICE_HOST` and `<SERVICE_NAME>_SERVICE_PORT`
- Only available for services created before pod

```bash
BACKEND_SERVICE_HOST=10.96.0.10
BACKEND_SERVICE_PORT=80
```

**2. DNS (Recommended)**
- CoreDNS creates DNS records for services
- Format: `<service-name>.<namespace>.svc.cluster.local`
- Works for services created after pod

```bash
# Same namespace
curl http://backend-service

# Different namespace
curl http://backend-service.production.svc.cluster.local

# Short form
curl http://backend-service.production
```

**DNS Records:**
- **A Record**: `service-name.namespace.svc.cluster.local` → ClusterIP
- **SRV Record**: Port information
- **Pod DNS**: `pod-ip-address.namespace.pod.cluster.local`

### Q16: What are Endpoints in Kubernetes?

**Answer:**

**Endpoints** are objects that track IP addresses of pods matching a service selector.

```bash
# View endpoints
kubectl get endpoints
kubectl describe endpoints <service-name>
```

**Automatic Endpoints:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: myapp  # Automatically creates endpoints for matching pods
  ports:
  - port: 80
```

**Manual Endpoints:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-service
spec:
  ports:
  - port: 80
---
apiVersion: v1
kind: Endpoints
metadata:
  name: external-service
subsets:
- addresses:
  - ip: 192.168.1.100
  - ip: 192.168.1.101
  ports:
  - port: 80
```

---

## ConfigMaps and Secrets

### Q17: What is the difference between ConfigMaps and Secrets?

**Answer:**

| ConfigMap | Secret |
|-----------|--------|
| Stores non-sensitive configuration | Stores sensitive data |
| Data stored in plain text | Data base64 encoded |
| Used for config files, env vars, command-line args | Used for passwords, tokens, keys |
| No encryption by default | Can be encrypted at rest |
| Max size: 1MB | Max size: 1MB |

**Best Practice:** Always use Secrets for sensitive data even though base64 is not encryption.

### Q18: How do you create and use ConfigMaps?

**Answer:**

**Creating ConfigMaps:**

**1. From literal values:**
```bash
kubectl create configmap app-config \
  --from-literal=DATABASE_URL=postgres://db:5432 \
  --from-literal=LOG_LEVEL=debug
```

**2. From file:**
```bash
kubectl create configmap nginx-config --from-file=nginx.conf
```

**3. From directory:**
```bash
kubectl create configmap app-configs --from-file=config-dir/
```

**4. From YAML:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  DATABASE_URL: "postgres://db:5432"
  LOG_LEVEL: "debug"
  app.properties: |
    server.port=8080
    server.name=myapp
    cache.enabled=true
```

**Using ConfigMaps:**

**1. As environment variables:**
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
    # Or individual keys
    env:
    - name: DATABASE_URL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: DATABASE_URL
```

**2. As volume:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: config-volume
      mountPath: /etc/nginx/nginx.conf
      subPath: nginx.conf
  volumes:
  - name: config-volume
    configMap:
      name: nginx-config
```

### Q19: How do you create and use Secrets?

**Answer:**

**Creating Secrets:**

**1. From literal:**
```bash
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=s3cr3tp@ss
```

**2. From file:**
```bash
kubectl create secret generic ssh-key --from-file=ssh-privatekey=/path/to/key
```

**3. From YAML (base64 encoded):**
```bash
# Encode data
echo -n 'admin' | base64  # YWRtaW4=
echo -n 's3cr3tp@ss' | base64  # czNjcjN0cEBzcw==
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: YWRtaW4=
  password: czNjcjN0cEBzcw==
# Or use stringData for plain text (K8s will encode)
stringData:
  username: admin
  password: s3cr3tp@ss
```

**Secret Types:**
- `Opaque`: Generic secret
- `kubernetes.io/service-account-token`: Service account token
- `kubernetes.io/dockerconfigjson`: Docker registry credentials
- `kubernetes.io/tls`: TLS certificate and key

**Docker Registry Secret:**
```bash
kubectl create secret docker-registry regcred \
  --docker-server=myregistry.com \
  --docker-username=user \
  --docker-password=pass \
  --docker-email=user@example.com
```

**Using Secrets:**

**1. As environment variables:**
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
```

**2. As volume:**
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
    - name: secret-volume
      mountPath: /etc/secrets
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: db-secret
```

**3. For image pull:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: private-app
spec:
  imagePullSecrets:
  - name: regcred
  containers:
  - name: app
    image: myregistry.com/myapp:1.0
```

### Q20: How can you update ConfigMaps and Secrets without restarting pods?

**Answer:**

**Challenge:** Pods don't automatically reload when ConfigMaps/Secrets change.

**Solutions:**

**1. Volume Mounts (Automatic Update)**
- ConfigMaps/Secrets mounted as volumes are updated automatically
- Update delay: kubelet sync period (default: 1 minute)
- Application must watch files for changes

**2. Environment Variables (No Auto-Update)**
- Require pod restart to pick up changes
- Update deployment to trigger rolling restart

**3. Force Pod Restart:**
```bash
# Update ConfigMap/Secret
kubectl edit configmap app-config

# Rolling restart deployment
kubectl rollout restart deployment/myapp

# Or use annotation to trigger update
kubectl patch deployment myapp -p \
  "{\"spec\":{\"template\":{\"metadata\":{\"annotations\":{\"date\":\"`date +'%s'`\"}}}}}"
```

**4. Use Reloader (Third-party)**
- Watches ConfigMaps/Secrets
- Automatically restarts pods on changes
- GitHub: stakater/Reloader

**Best Practice:** Mount as volumes when possible for automatic updates.

---

## Volumes and PersistentVolumes

### Q21: What are Volumes in Kubernetes?

**Answer:**

**Volumes** provide persistent storage for pods. Containers are ephemeral; volumes preserve data beyond container lifecycle.

**Volume Types:**

**1. emptyDir**
- Temporary storage for pod lifetime
- Deleted when pod is removed
- Shared between containers in pod

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
    - name: cache-volume
      mountPath: /cache
  volumes:
  - name: cache-volume
    emptyDir: {}
```

**2. hostPath**
- Mounts file/directory from host node
- Persists beyond pod lifecycle
- Data tied to specific node

```yaml
volumes:
- name: host-volume
  hostPath:
    path: /data
    type: Directory
```

**3. configMap / secret**
- Mount ConfigMaps/Secrets as files

**4. persistentVolumeClaim**
- Claims persistent storage
- Decouples storage from pod

### Q22: Explain PersistentVolume (PV) and PersistentVolumeClaim (PVC)

**Answer:**

**PersistentVolume (PV):**
- Cluster resource for storage
- Provisioned by admin or dynamically
- Independent lifecycle from pods
- Can be network storage (NFS, EBS, Azure Disk, etc.)

**PersistentVolumeClaim (PVC):**
- Request for storage by user
- Binds to available PV
- Pods use PVCs to access storage

**Relationship:**
```
Pod → PVC → PV → Actual Storage
```

**PersistentVolume Example:**

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-mysql
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

**PersistentVolumeClaim Example:**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-mysql
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: standard
```

**Using PVC in Pod:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysql-pod
spec:
  containers:
  - name: mysql
    image: mysql:8.0
    env:
    - name: MYSQL_ROOT_PASSWORD
      value: password
    volumeMounts:
    - name: mysql-storage
      mountPath: /var/lib/mysql
  volumes:
  - name: mysql-storage
    persistentVolumeClaim:
      claimName: pvc-mysql
```

### Q23: What are Access Modes in Kubernetes volumes?

**Answer:**

**Access Modes** define how volumes can be mounted:

1. **ReadWriteOnce (RWO)**
   - Volume mounted as read-write by single node
   - Multiple pods on same node can access
   - Most common for block storage (EBS, Azure Disk)

2. **ReadOnlyMany (ROX)**
   - Volume mounted as read-only by many nodes
   - Used for shared configuration/static content

3. **ReadWriteMany (RWX)**
   - Volume mounted as read-write by many nodes
   - Requires shared filesystem (NFS, CephFS, GlusterFS)
   - Not supported by block storage

4. **ReadWriteOncePod (RWOP)**
   - Volume mounted as read-write by single pod (K8s 1.22+)
   - Even stricter than RWO

**Example:**
```yaml
accessModes:
- ReadWriteOnce  # AWS EBS
- ReadOnlyMany   # NFS (read-only)
- ReadWriteMany  # NFS (read-write)
```

### Q24: What is StorageClass and dynamic provisioning?

**Answer:**

**StorageClass** defines different classes of storage with different properties.

**Benefits:**
- Dynamic PV creation
- No manual PV provisioning
- Automatic storage allocation

**StorageClass Example:**

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  iops: "3000"
  throughput: "125"
allowVolumeExpansion: true
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```

**Using StorageClass:**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-dynamic
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: fast-ssd  # References StorageClass
  resources:
    requests:
      storage: 50Gi
```

**Common Provisioners:**
- `kubernetes.io/aws-ebs` - AWS Elastic Block Store
- `kubernetes.io/azure-disk` - Azure Disk
- `kubernetes.io/gce-pd` - Google Persistent Disk
- `kubernetes.io/cinder` - OpenStack Cinder
- `kubernetes.io/nfs` - NFS

**Volume Binding Modes:**
- **Immediate**: PV created immediately when PVC created
- **WaitForFirstConsumer**: PV created when pod using PVC is scheduled (better for topology)

### Q25: What is Reclaim Policy?

**Answer:**

**Reclaim Policy** determines what happens to PV when PVC is deleted:

**1. Retain**
- PV retained after PVC deletion
- Manual cleanup required
- Data preserved for recovery

**2. Delete**
- PV and underlying storage deleted
- Default for dynamic provisioning
- Data permanently lost

**3. Recycle (Deprecated)**
- Basic scrub (`rm -rf /volume/*`)
- PV available for new claim
- Deprecated in favor of dynamic provisioning

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-important-data
spec:
  persistentVolumeReclaimPolicy: Retain
  capacity:
    storage: 100Gi
  accessModes:
  - ReadWriteOnce
```

**Lifecycle:**
```
Available → Bound → Released → (Retain/Delete)
```

---

## Namespaces

### Q26: What are Namespaces and why use them?

**Answer:**

**Namespaces** provide virtual clusters within physical Kubernetes cluster.

**Benefits:**
1. **Resource Isolation**: Separate resources by team/project/environment
2. **Resource Quotas**: Limit resources per namespace
3. **Access Control**: RBAC policies per namespace
4. **Organization**: Logical grouping of resources
5. **Multi-tenancy**: Multiple teams sharing cluster

**Default Namespaces:**
- **default**: Default namespace for objects without namespace
- **kube-system**: Kubernetes system components
- **kube-public**: Publicly accessible, even without auth
- **kube-node-lease**: Node heartbeat information

**Create Namespace:**

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: development
```

```bash
kubectl create namespace development
```

### Q27: How do you use Namespaces?

**Answer:**

**Create resources in namespace:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: development  # Specify namespace
spec:
  containers:
  - name: nginx
    image: nginx
```

```bash
# Using kubectl
kubectl apply -f pod.yaml -n development
kubectl create deployment nginx --image=nginx -n development
```

**View resources:**
```bash
# Specific namespace
kubectl get pods -n development

# All namespaces
kubectl get pods --all-namespaces
kubectl get pods -A

# Set default namespace
kubectl config set-context --current --namespace=development
```

**Cross-namespace communication:**
```bash
# Service DNS format
<service-name>.<namespace>.svc.cluster.local

# Example
curl http://database.production.svc.cluster.local:5432
```

### Q28: How do you implement Resource Quotas in Namespaces?

**Answer:**

**ResourceQuota** limits resource consumption per namespace.

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: development
spec:
  hard:
    requests.cpu: "10"        # Total CPU requests
    requests.memory: 20Gi     # Total memory requests
    limits.cpu: "20"          # Total CPU limits
    limits.memory: 40Gi       # Total memory limits
    persistentvolumeclaims: "5"  # Max PVCs
    pods: "20"                # Max pods
    services: "10"            # Max services
    secrets: "15"             # Max secrets
    configmaps: "10"          # Max ConfigMaps
```

**Object Count Quota:**
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: object-quota
  namespace: development
spec:
  hard:
    count/deployments.apps: "10"
    count/services: "5"
    count/secrets: "10"
    count/configmaps: "10"
```

**Check quota:**
```bash
kubectl get resourcequota -n development
kubectl describe resourcequota compute-quota -n development
```

### Q29: What are LimitRanges?

**Answer:**

**LimitRange** sets default limits for containers in namespace.

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-memory-limits
  namespace: development
spec:
  limits:
  - max:
      cpu: "2"
      memory: 4Gi
    min:
      cpu: "100m"
      memory: 128Mi
    default:  # Default limits if not specified
      cpu: "500m"
      memory: 512Mi
    defaultRequest:  # Default requests if not specified
      cpu: "250m"
      memory: 256Mi
    type: Container
  - max:
      storage: 10Gi
    min:
      storage: 1Gi
    type: PersistentVolumeClaim
```

**Effects:**
- Enforces min/max resource constraints
- Sets defaults for pods without limits
- Prevents resource hogging

---

## Ingress

### Q30: What is Ingress and how does it differ from Service?

**Answer:**

**Service:**
- Layer 4 (TCP/UDP) load balancing
- One external IP per service (LoadBalancer)
- Limited routing capabilities

**Ingress:**
- Layer 7 (HTTP/HTTPS) routing
- Single entry point for multiple services
- Advanced routing (host-based, path-based)
- SSL/TLS termination
- Name-based virtual hosting

**Relationship:**
```
Internet → Ingress → Services → Pods
```

**Requires:** Ingress Controller (NGINX, Traefik, HAProxy, Istio, etc.)

### Q31: How do you create and configure Ingress?

**Answer:**

**1. Install Ingress Controller:**

```bash
# NGINX Ingress Controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml
```

**2. Basic Ingress (Single Service):**

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
            name: web-service
            port:
              number: 80
```

**3. Multiple Paths:**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: path-based-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
      - path: /web
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

**4. Multiple Hosts:**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: host-based-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
  - host: web.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

**5. TLS/SSL:**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret
type: kubernetes.io/tls
data:
  tls.crt: <base64-encoded-cert>
  tls.key: <base64-encoded-key>
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-ingress
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - secure.example.com
    secretName: tls-secret
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
              number: 443
```

### Q32: What are Ingress annotations and common use cases?

**Answer:**

**Annotations** provide additional configuration for Ingress controllers.

**Common NGINX Ingress Annotations:**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: annotated-ingress
  annotations:
    # SSL/TLS
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"

    # Rewrites
    nginx.ingress.kubernetes.io/rewrite-target: /$2
    nginx.ingress.kubernetes.io/use-regex: "true"

    # CORS
    nginx.ingress.kubernetes.io/enable-cors: "true"
    nginx.ingress.kubernetes.io/cors-allow-origin: "*"

    # Rate limiting
    nginx.ingress.kubernetes.io/limit-rps: "10"
    nginx.ingress.kubernetes.io/limit-connections: "5"

    # Timeouts
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "30"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "30"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "30"

    # Authentication
    nginx.ingress.kubernetes.io/auth-type: basic
    nginx.ingress.kubernetes.io/auth-secret: basic-auth
    nginx.ingress.kubernetes.io/auth-realm: "Authentication Required"

    # Whitelist IPs
    nginx.ingress.kubernetes.io/whitelist-source-range: "10.0.0.0/24,192.168.1.0/24"

    # Client body size
    nginx.ingress.kubernetes.io/proxy-body-size: "50m"

    # Custom headers
    nginx.ingress.kubernetes.io/configuration-snippet: |
      more_set_headers "X-Custom-Header: value";
spec:
  ingressClassName: nginx
  rules:
  - host: example.com
    http:
      paths:
      - path: /api(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
```

---

## Auto-scaling (HPA)

### Q33: What is Horizontal Pod Autoscaler (HPA)?

**Answer:**

**HPA** automatically scales number of pods based on metrics.

**How it works:**
1. Metrics Server collects resource metrics
2. HPA controller queries metrics periodically
3. Compares current vs target metrics
4. Scales deployment up/down

**Formula:**
```
desiredReplicas = ceil[currentReplicas * (currentMetricValue / targetMetricValue)]
```

**Requirements:**
- Metrics Server installed
- Resource requests defined in pods

### Q34: How do you configure HPA?

**Answer:**

**1. Install Metrics Server:**

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

**2. CPU-based HPA:**

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70  # Target 70% CPU
```

**3. Memory-based HPA:**

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: memory-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

**4. Multiple Metrics:**

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: multi-metric-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-deployment
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
  behavior:  # Scaling behavior
    scaleDown:
      stabilizationWindowSeconds: 300  # Wait 5 min before scale down
      policies:
      - type: Percent
        value: 50  # Scale down max 50% of pods
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100  # Double pods
        periodSeconds: 60
      - type: Pods
        value: 4  # Or add 4 pods
        periodSeconds: 60
      selectPolicy: Max  # Use policy giving max pods
```

**5. Custom Metrics (Prometheus):**

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: custom-metric-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Pods
    pods:
      metric:
        name: http_requests_per_second
      target:
        type: AverageValue
        averageValue: "1000"
```

**Create HPA via kubectl:**

```bash
kubectl autoscale deployment web-deployment \
  --cpu-percent=70 \
  --min=2 \
  --max=10
```

**Monitor HPA:**

```bash
kubectl get hpa
kubectl describe hpa web-hpa
kubectl top pods  # View resource usage
```

### Q35: What is Vertical Pod Autoscaler (VPA)?

**Answer:**

**VPA** automatically adjusts CPU and memory requests/limits.

**Difference from HPA:**
- HPA: Scales number of pods (horizontal)
- VPA: Scales pod resources (vertical)

**Use Cases:**
- Optimize resource allocation
- Right-sizing containers
- Cost optimization

**VPA Example:**

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: web-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-deployment
  updatePolicy:
    updateMode: "Auto"  # Auto, Initial, Off
  resourcePolicy:
    containerPolicies:
    - containerName: web
      minAllowed:
        cpu: 100m
        memory: 128Mi
      maxAllowed:
        cpu: 2
        memory: 2Gi
      controlledResources: ["cpu", "memory"]
```

**Update Modes:**
- **Auto**: VPA updates pods automatically
- **Initial**: VPA only sets resources on pod creation
- **Off**: VPA only provides recommendations

### Q36: What is Cluster Autoscaler?

**Answer:**

**Cluster Autoscaler** automatically adjusts number of nodes in cluster.

**Scales Up When:**
- Pods are pending due to insufficient resources
- New nodes needed to accommodate workload

**Scales Down When:**
- Nodes underutilized for extended period
- Pods can be rescheduled to other nodes

**Cloud Provider Integration:**
```yaml
# AWS example
apiVersion: v1
kind: ServiceAccount
metadata:
  name: cluster-autoscaler
  namespace: kube-system
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cluster-autoscaler
  namespace: kube-system
spec:
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
        - --cloud-provider=aws
        - --namespace=kube-system
        - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/my-cluster
        - --balance-similar-node-groups
        - --skip-nodes-with-system-pods=false
```

**Autoscaling Combination:**
- **HPA**: Scales pods based on metrics
- **VPA**: Optimizes pod resources
- **Cluster Autoscaler**: Scales nodes to accommodate pods

---

## RBAC (Role-Based Access Control)

### Q37: What is RBAC in Kubernetes?

**Answer:**

**RBAC** regulates access to Kubernetes resources based on roles assigned to users/groups.

**Core Concepts:**
1. **Subject**: User, Group, or ServiceAccount
2. **Role/ClusterRole**: Set of permissions
3. **RoleBinding/ClusterRoleBinding**: Binds role to subject

**Scope:**
- **Role/RoleBinding**: Namespace-scoped
- **ClusterRole/ClusterRoleBinding**: Cluster-wide

### Q38: How do you create Roles and RoleBindings?

**Answer:**

**1. Role (Namespace-scoped):**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: development
rules:
- apiGroups: [""]  # "" indicates core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
- apiGroups: [""]
  resources: ["pods/log"]
  verbs: ["get"]
```

**2. RoleBinding:**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: development
subjects:
- kind: User
  name: john
  apiGroup: rbac.authorization.k8s.io
- kind: ServiceAccount
  name: my-service-account
  namespace: development
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

**3. ClusterRole (Cluster-wide):**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-admin-custom
rules:
- apiGroups: [""]
  resources: ["*"]
  verbs: ["*"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets", "statefulsets"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: ["batch"]
  resources: ["jobs", "cronjobs"]
  verbs: ["*"]
```

**4. ClusterRoleBinding:**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-binding
subjects:
- kind: User
  name: admin
  apiGroup: rbac.authorization.k8s.io
- kind: Group
  name: system:masters
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-admin-custom
  apiGroup: rbac.authorization.k8s.io
```

### Q39: What are common RBAC verbs and resources?

**Answer:**

**Verbs (Actions):**
- `get`: Read single resource
- `list`: Read all resources of type
- `watch`: Watch for changes
- `create`: Create new resource
- `update`: Update existing resource
- `patch`: Partially update resource
- `delete`: Delete resource
- `deletecollection`: Delete collection of resources
- `*`: All verbs

**Resources:**
- Core API (`""`): pods, services, configmaps, secrets, persistentvolumeclaims, namespaces
- Apps API (`apps`): deployments, replicasets, statefulsets, daemonsets
- Batch API (`batch`): jobs, cronjobs
- Networking (`networking.k8s.io`): ingresses, networkpolicies
- RBAC (`rbac.authorization.k8s.io`): roles, rolebindings, clusterroles, clusterrolebindings

**Subresources:**
- `pods/log`: Pod logs
- `pods/exec`: Execute in pod
- `pods/portforward`: Port forwarding
- `deployments/scale`: Scale deployments

### Q40: How do you create and use ServiceAccounts?

**Answer:**

**ServiceAccount** provides identity for processes running in pods.

**Create ServiceAccount:**

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-service-account
  namespace: development
```

**Assign Role to ServiceAccount:**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: deployment-manager
  namespace: development
rules:
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch", "create", "update", "patch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: deployment-manager-binding
  namespace: development
subjects:
- kind: ServiceAccount
  name: my-service-account
  namespace: development
roleRef:
  kind: Role
  name: deployment-manager
  apiGroup: rbac.authorization.k8s.io
```

**Use ServiceAccount in Pod:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  namespace: development
spec:
  serviceAccountName: my-service-account
  containers:
  - name: kubectl
    image: bitnami/kubectl
    command: ['sleep', '3600']
```

**Access API from Pod:**
```bash
# Token automatically mounted at /var/run/secrets/kubernetes.io/serviceaccount/token
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
APISERVER=https://kubernetes.default.svc
NAMESPACE=$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace)

curl -H "Authorization: Bearer $TOKEN" \
     --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt \
     $APISERVER/api/v1/namespaces/$NAMESPACE/pods
```

### Q41: How do you check RBAC permissions?

**Answer:**

**Check if you can perform action:**
```bash
kubectl auth can-i create deployments
kubectl auth can-i delete pods --namespace development
kubectl auth can-i '*' '*'  # All actions on all resources

# Check for specific user
kubectl auth can-i get pods --as john
kubectl auth can-i create secrets --as system:serviceaccount:development:my-service-account
```

**View roles and bindings:**
```bash
kubectl get roles -n development
kubectl get rolebindings -n development
kubectl get clusterroles
kubectl get clusterrolebindings

kubectl describe role pod-reader -n development
kubectl describe rolebinding read-pods -n development
```

**List permissions for ServiceAccount:**
```bash
kubectl describe serviceaccount my-service-account -n development
```

---

## kubectl Commands

### Q42: What are essential kubectl commands?

**Answer:**

**Cluster Info:**
```bash
kubectl cluster-info
kubectl version
kubectl api-resources
kubectl api-versions
```

**Get Resources:**
```bash
kubectl get nodes
kubectl get pods
kubectl get pods -o wide  # More details
kubectl get pods --all-namespaces
kubectl get pods -n development
kubectl get all  # All resources in namespace
kubectl get pods --show-labels
kubectl get pods -l app=nginx  # Filter by label
kubectl get pods --field-selector status.phase=Running
```

**Describe Resources:**
```bash
kubectl describe node <node-name>
kubectl describe pod <pod-name>
kubectl describe deployment <deployment-name>
```

**Create Resources:**
```bash
kubectl create deployment nginx --image=nginx
kubectl create service clusterip my-service --tcp=80:80
kubectl create namespace development
kubectl create configmap app-config --from-literal=key=value
kubectl create secret generic db-secret --from-literal=password=secret
```

**Apply/Delete:**
```bash
kubectl apply -f deployment.yaml
kubectl apply -f directory/
kubectl delete -f deployment.yaml
kubectl delete pod <pod-name>
kubectl delete deployment <deployment-name>
kubectl delete pods --all
kubectl delete all --all  # Dangerous!
```

**Update Resources:**
```bash
kubectl edit deployment <deployment-name>
kubectl patch deployment <deployment-name> -p '{"spec":{"replicas":5}}'
kubectl set image deployment/nginx nginx=nginx:1.22
kubectl scale deployment nginx --replicas=5
kubectl rollout restart deployment nginx
```

**Logs and Exec:**
```bash
kubectl logs <pod-name>
kubectl logs <pod-name> -c <container-name>  # Multi-container pod
kubectl logs -f <pod-name>  # Follow logs
kubectl logs --tail=100 <pod-name>
kubectl logs --since=1h <pod-name>
kubectl logs <pod-name> --previous  # Previous container instance

kubectl exec -it <pod-name> -- /bin/bash
kubectl exec <pod-name> -- ls /app
kubectl exec <pod-name> -c <container-name> -- env
```

**Port Forward:**
```bash
kubectl port-forward pod/<pod-name> 8080:80
kubectl port-forward service/<service-name> 8080:80
kubectl port-forward deployment/<deployment-name> 8080:80
```

**Copy Files:**
```bash
kubectl cp <pod-name>:/path/to/file ./local-file
kubectl cp ./local-file <pod-name>:/path/to/file
```

**Labels and Annotations:**
```bash
kubectl label pods <pod-name> environment=production
kubectl label pods <pod-name> environment-  # Remove label
kubectl annotate pods <pod-name> description="My pod"
```

**Resource Usage:**
```bash
kubectl top nodes
kubectl top pods
kubectl top pods --containers
```

### Q43: What are useful kubectl flags and options?

**Answer:**

**Output Formats:**
```bash
kubectl get pods -o yaml
kubectl get pods -o json
kubectl get pods -o wide
kubectl get pods -o name
kubectl get pods -o jsonpath='{.items[*].metadata.name}'
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase
```

**Dry Run:**
```bash
kubectl apply -f deployment.yaml --dry-run=client
kubectl apply -f deployment.yaml --dry-run=server
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deployment.yaml
```

**Watch and Wait:**
```bash
kubectl get pods --watch
kubectl wait --for=condition=ready pod/<pod-name>
kubectl wait --for=condition=available deployment/<deployment-name>
```

**Context and Namespace:**
```bash
kubectl config get-contexts
kubectl config use-context <context-name>
kubectl config set-context --current --namespace=development
kubectl config view
```

**Explain:**
```bash
kubectl explain pod
kubectl explain pod.spec
kubectl explain pod.spec.containers
```

**Diff:**
```bash
kubectl diff -f deployment.yaml
```

---

## Troubleshooting

### Q44: How do you troubleshoot Pod issues?

**Answer:**

**1. Check Pod Status:**
```bash
kubectl get pods
kubectl describe pod <pod-name>
```

**Common States:**
- **Pending**: Not scheduled (resource constraints, node selector)
- **ImagePullBackOff**: Cannot pull image
- **CrashLoopBackOff**: Container keeps crashing
- **Error**: Container exited with error
- **Unknown**: Communication lost with node

**2. Check Events:**
```bash
kubectl get events --sort-by='.lastTimestamp'
kubectl get events --field-selector involvedObject.name=<pod-name>
```

**3. Check Logs:**
```bash
kubectl logs <pod-name>
kubectl logs <pod-name> --previous  # Previous crashed container
kubectl logs <pod-name> -c <container-name>
```

**4. Exec into Container:**
```bash
kubectl exec -it <pod-name> -- /bin/sh
# Check processes, files, network, etc.
```

**5. Check Resource Limits:**
```bash
kubectl describe node <node-name>
kubectl top nodes
kubectl top pods
```

**6. Debug with Ephemeral Container (K8s 1.23+):**
```bash
kubectl debug <pod-name> -it --image=busybox --target=<container-name>
```

### Q45: How do you troubleshoot Service connectivity issues?

**Answer:**

**1. Verify Service Exists:**
```bash
kubectl get svc
kubectl describe svc <service-name>
```

**2. Check Endpoints:**
```bash
kubectl get endpoints <service-name>
```

**If no endpoints:**
- Check pod labels match service selector
- Verify pods are running and ready

**3. Test DNS Resolution:**
```bash
kubectl run test --image=busybox -it --rm -- nslookup <service-name>
kubectl run test --image=busybox -it --rm -- nslookup <service-name>.<namespace>.svc.cluster.local
```

**4. Test Connectivity:**
```bash
kubectl run test --image=busybox -it --rm -- wget -O- http://<service-name>:<port>
```

**5. Check Network Policies:**
```bash
kubectl get networkpolicies
kubectl describe networkpolicy <policy-name>
```

**6. Verify kube-proxy:**
```bash
kubectl get pods -n kube-system | grep kube-proxy
kubectl logs -n kube-system <kube-proxy-pod>
```

### Q46: How do you troubleshoot Node issues?

**Answer:**

**1. Check Node Status:**
```bash
kubectl get nodes
kubectl describe node <node-name>
```

**Node Conditions:**
- **Ready**: Node is healthy
- **MemoryPressure**: Low memory
- **DiskPressure**: Low disk space
- **PIDPressure**: Too many processes
- **NetworkUnavailable**: Network not configured

**2. Check Node Resources:**
```bash
kubectl top node <node-name>
kubectl describe node <node-name> | grep -A 5 "Allocated resources"
```

**3. Check Kubelet:**
```bash
# SSH to node
systemctl status kubelet
journalctl -u kubelet -f
```

**4. Check Container Runtime:**
```bash
# SSH to node
systemctl status docker  # or containerd
docker ps
```

**5. Drain and Cordon:**
```bash
kubectl cordon <node-name>  # Mark unschedulable
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
kubectl uncordon <node-name>  # Mark schedulable again
```

### Q47: Common errors and solutions

**Answer:**

**ImagePullBackOff:**
```yaml
# Problem: Cannot pull image
# Solutions:
- Check image name/tag is correct
- Verify image exists in registry
- Check imagePullSecrets for private registry
- Verify network connectivity to registry
```

**CrashLoopBackOff:**
```yaml
# Problem: Container keeps crashing
# Solutions:
- Check logs: kubectl logs <pod-name> --previous
- Verify command/args are correct
- Check resource limits (OOMKilled)
- Verify dependencies (database, services)
- Check liveness probe configuration
```

**Pending Pods:**
```yaml
# Problem: Pod not scheduled
# Solutions:
- Check node resources: kubectl describe nodes
- Verify PVC is bound
- Check node selectors/affinity rules
- Verify tolerations for tainted nodes
- Check resource quotas
```

**OOMKilled:**
```yaml
# Problem: Container killed due to memory
# Solutions:
- Increase memory limits
- Fix memory leak in application
- Check actual memory usage: kubectl top pods
```

**CreateContainerConfigError:**
```yaml
# Problem: Container config error
# Solutions:
- Check ConfigMap/Secret exists
- Verify keys in ConfigMap/Secret
- Check volume mounts
```

---

## Helm

### Q48: What is Helm and why use it?

**Answer:**

**Helm** is the package manager for Kubernetes.

**Benefits:**
1. **Package Management**: Bundle related K8s resources
2. **Templating**: Parameterize manifests
3. **Versioning**: Track releases and rollback
4. **Reusability**: Share charts across projects
5. **Dependency Management**: Manage chart dependencies

**Concepts:**
- **Chart**: Package of K8s resources
- **Release**: Instance of chart running in cluster
- **Repository**: Collection of charts
- **Values**: Configuration parameters

### Q49: How do you create and use Helm charts?

**Answer:**

**Install Helm:**
```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

**Basic Commands:**

```bash
# Add repository
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Search charts
helm search repo nginx
helm search hub wordpress

# Install chart
helm install my-nginx bitnami/nginx
helm install my-release bitnami/mysql --namespace database --create-namespace

# List releases
helm list
helm list --all-namespaces

# Get release info
helm status my-nginx
helm get values my-nginx
helm get manifest my-nginx

# Upgrade release
helm upgrade my-nginx bitnami/nginx --set replicaCount=3
helm upgrade my-nginx bitnami/nginx -f custom-values.yaml

# Rollback
helm rollback my-nginx
helm rollback my-nginx 2  # To specific revision

# Uninstall
helm uninstall my-nginx
```

**Create Custom Chart:**

```bash
helm create myapp
```

**Chart Structure:**
```
myapp/
├── Chart.yaml          # Chart metadata
├── values.yaml         # Default values
├── charts/             # Dependencies
└── templates/          # K8s manifests
    ├── deployment.yaml
    ├── service.yaml
    ├── ingress.yaml
    ├── _helpers.tpl    # Template helpers
    └── NOTES.txt       # Usage notes
```

**Chart.yaml:**
```yaml
apiVersion: v2
name: myapp
description: A Helm chart for MyApp
type: application
version: 1.0.0
appVersion: "1.0"
dependencies:
- name: postgresql
  version: 12.x.x
  repository: https://charts.bitnami.com/bitnami
```

**values.yaml:**
```yaml
replicaCount: 2

image:
  repository: myapp
  pullPolicy: IfNotPresent
  tag: "1.0.0"

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: true
  className: nginx
  hosts:
  - host: myapp.example.com
    paths:
    - path: /
      pathType: Prefix

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
        - containerPort: 8080
          protocol: TCP
        resources:
          {{- toYaml .Values.resources | nindent 12 }}
```

**Install Custom Chart:**
```bash
helm install myapp ./myapp
helm install myapp ./myapp -f production-values.yaml
helm install myapp ./myapp --set replicaCount=5
```

**Package and Share:**
```bash
helm package myapp
helm repo index .
```

### Q50: What are Helm hooks and tests?

**Answer:**

**Helm Hooks** allow executing resources at specific points in release lifecycle.

**Hook Types:**
- `pre-install`: Before install
- `post-install`: After install
- `pre-upgrade`: Before upgrade
- `post-upgrade`: After upgrade
- `pre-delete`: Before delete
- `post-delete`: After delete
- `pre-rollback`: Before rollback
- `post-rollback`: After rollback

**Example Hook (Database Migration):**

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: {{ include "myapp.fullname" . }}-migration
  annotations:
    "helm.sh/hook": pre-upgrade,pre-install
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": before-hook-creation
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: migration
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        command: ['python', 'manage.py', 'migrate']
```

**Helm Tests:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: {{ include "myapp.fullname" . }}-test
  annotations:
    "helm.sh/hook": test
spec:
  restartPolicy: Never
  containers:
  - name: test
    image: busybox
    command: ['wget']
    args: ['{{ include "myapp.fullname" . }}:{{ .Values.service.port }}']
```

**Run Tests:**
```bash
helm test myapp
```

---

## Real-World Scenarios

### Q51: Design a production-ready microservices deployment

**Answer:**

**Architecture:**
```
Internet → Ingress → Frontend Service → Frontend Pods
                  ↘ API Service → API Pods → Database Service → Database StatefulSet
```

**1. Namespace Organization:**

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    environment: production
```

**2. Database (StatefulSet):**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: postgres-secret
  namespace: production
type: Opaque
stringData:
  POSTGRES_PASSWORD: "secure-password"
  POSTGRES_USER: "appuser"
  POSTGRES_DB: "appdb"
---
apiVersion: v1
kind: Service
metadata:
  name: postgres
  namespace: production
spec:
  clusterIP: None  # Headless
  selector:
    app: postgres
  ports:
  - port: 5432
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
  namespace: production
spec:
  serviceName: postgres
  replicas: 3
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
        envFrom:
        - secretRef:
            name: postgres-secret
        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
        resources:
          requests:
            cpu: 500m
            memory: 1Gi
          limits:
            cpu: 1000m
            memory: 2Gi
        livenessProbe:
          exec:
            command:
            - pg_isready
            - -U
            - appuser
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          exec:
            command:
            - pg_isready
            - -U
            - appuser
          initialDelaySeconds: 5
          periodSeconds: 5
  volumeClaimTemplates:
  - metadata:
      name: postgres-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: fast-ssd
      resources:
        requests:
          storage: 50Gi
```

**3. Backend API (Deployment):**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: api-config
  namespace: production
data:
  DATABASE_HOST: postgres
  DATABASE_PORT: "5432"
  LOG_LEVEL: "info"
  CACHE_ENABLED: "true"
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
  namespace: production
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: api
      tier: backend
  template:
    metadata:
      labels:
        app: api
        tier: backend
        version: v1
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - api
              topologyKey: kubernetes.io/hostname
      containers:
      - name: api
        image: myregistry.com/api:1.2.3
        ports:
        - containerPort: 8080
          name: http
        envFrom:
        - configMapRef:
            name: api-config
        env:
        - name: DATABASE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: POSTGRES_PASSWORD
        - name: DATABASE_USER
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: POSTGRES_USER
        resources:
          requests:
            cpu: 250m
            memory: 256Mi
          limits:
            cpu: 1000m
            memory: 512Mi
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
          successThreshold: 1
          failureThreshold: 3
        lifecycle:
          preStop:
            exec:
              command: ["/bin/sh", "-c", "sleep 15"]
---
apiVersion: v1
kind: Service
metadata:
  name: api
  namespace: production
spec:
  type: ClusterIP
  selector:
    app: api
    tier: backend
  ports:
  - port: 8080
    targetPort: 8080
    name: http
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api
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
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
```

**4. Frontend (Deployment):**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
      tier: frontend
  template:
    metadata:
      labels:
        app: frontend
        tier: frontend
    spec:
      containers:
      - name: frontend
        image: myregistry.com/frontend:1.2.3
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 256Mi
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
---
apiVersion: v1
kind: Service
metadata:
  name: frontend
  namespace: production
spec:
  type: ClusterIP
  selector:
    app: frontend
    tier: frontend
  ports:
  - port: 80
    targetPort: 80
```

**5. Ingress with TLS:**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret
  namespace: production
type: kubernetes.io/tls
data:
  tls.crt: <base64-cert>
  tls.key: <base64-key>
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  namespace: production
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/rate-limit: "100"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - myapp.example.com
    - api.myapp.example.com
    secretName: tls-secret
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend
            port:
              number: 80
  - host: api.myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api
            port:
              number: 8080
```

**6. Resource Quotas:**

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: production-quota
  namespace: production
spec:
  hard:
    requests.cpu: "50"
    requests.memory: 100Gi
    limits.cpu: "100"
    limits.memory: 200Gi
    persistentvolumeclaims: "20"
    services.loadbalancers: "2"
```

### Q52: Implement zero-downtime deployment strategy

**Answer:**

**Strategy: Blue-Green Deployment**

**1. Blue Deployment (Current):**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-blue
  labels:
    app: myapp
    version: blue
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
      - name: myapp
        image: myapp:v1.0
        ports:
        - containerPort: 8080
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
        lifecycle:
          preStop:
            exec:
              command: ["/bin/sh", "-c", "sleep 20"]
```

**2. Service (Points to Blue):**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  selector:
    app: myapp
    version: blue  # Points to blue
  ports:
  - port: 80
    targetPort: 8080
```

**3. Deploy Green:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-green
  labels:
    app: myapp
    version: green
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
      - name: myapp
        image: myapp:v2.0  # New version
        ports:
        - containerPort: 8080
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
```

**4. Switch Traffic:**

```bash
# After green is ready and tested
kubectl patch service myapp -p '{"spec":{"selector":{"version":"green"}}}'

# Rollback if issues
kubectl patch service myapp -p '{"spec":{"selector":{"version":"blue"}}}'
```

**Alternative: Canary Deployment**

```yaml
# Primary deployment (90% traffic)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-stable
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
      - name: myapp
        image: myapp:v1.0
---
# Canary deployment (10% traffic)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-canary
spec:
  replicas: 1
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
      - name: myapp
        image: myapp:v2.0
---
# Service selects both
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  selector:
    app: myapp  # Selects both stable and canary
  ports:
  - port: 80
    targetPort: 8080
```

### Q53: Implement backup and disaster recovery strategy

**Answer:**

**1. etcd Backup:**

```bash
# Backup etcd
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-snapshot-$(date +%Y%m%d-%H%M%S).db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Verify backup
ETCDCTL_API=3 etcdctl snapshot status /backup/etcd-snapshot.db

# Restore (if needed)
ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-snapshot.db \
  --data-dir /var/lib/etcd-restore
```

**Automated Backup CronJob:**

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: etcd-backup
  namespace: kube-system
spec:
  schedule: "0 */6 * * *"  # Every 6 hours
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: k8s.gcr.io/etcd:3.5.9
            command:
            - /bin/sh
            - -c
            - |
              ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-$(date +%Y%m%d-%H%M%S).db \
                --endpoints=https://etcd:2379 \
                --cacert=/etc/kubernetes/pki/etcd/ca.crt \
                --cert=/etc/kubernetes/pki/etcd/server.crt \
                --key=/etc/kubernetes/pki/etcd/server.key
              # Upload to S3/GCS
              aws s3 cp /backup/etcd-*.db s3://my-backup-bucket/etcd/
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
            emptyDir: {}
          restartPolicy: OnFailure
```

**2. PersistentVolume Backup (Velero):**

```bash
# Install Velero
velero install \
  --provider aws \
  --bucket velero-backup \
  --secret-file ./credentials-velero \
  --backup-location-config region=us-east-1 \
  --snapshot-location-config region=us-east-1

# Create backup
velero backup create production-backup --include-namespaces production

# Schedule automatic backups
velero schedule create daily-backup --schedule="0 2 * * *" --include-namespaces production

# Restore
velero restore create --from-backup production-backup
```

**3. Application State Backup:**

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: database-backup
  namespace: production
spec:
  schedule: "0 2 * * *"  # Daily at 2 AM
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: pg-dump
            image: postgres:15
            command:
            - /bin/bash
            - -c
            - |
              pg_dump -h postgres -U appuser appdb | gzip > /backup/db-$(date +%Y%m%d).sql.gz
              aws s3 cp /backup/db-*.sql.gz s3://my-backup-bucket/database/
              # Keep only last 30 days
              find /backup -name "db-*.sql.gz" -mtime +30 -delete
            env:
            - name: PGPASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgres-secret
                  key: POSTGRES_PASSWORD
            volumeMounts:
            - name: backup
              mountPath: /backup
          volumes:
          - name: backup
            persistentVolumeClaim:
              claimName: backup-pvc
          restartPolicy: OnFailure
```

### Q54: Monitor and debug performance issues

**Answer:**

**1. Install Monitoring Stack (Prometheus + Grafana):**

```bash
# Add helm repo
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Install kube-prometheus-stack
helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace
```

**2. ServiceMonitor for Custom App:**

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: api-metrics
  namespace: production
spec:
  selector:
    matchLabels:
      app: api
  endpoints:
  - port: metrics
    interval: 30s
    path: /metrics
```

**3. Debug High CPU Usage:**

```bash
# Check pod resource usage
kubectl top pods -n production

# Check specific pod
kubectl top pod api-xyz --containers

# Get detailed metrics
kubectl get --raw /apis/metrics.k8s.io/v1beta1/namespaces/production/pods/api-xyz

# Profile application
kubectl exec -it api-xyz -- curl http://localhost:8080/debug/pprof/profile?seconds=30 > cpu.prof
```

**4. Debug Memory Issues:**

```bash
# Check for OOMKilled
kubectl describe pod api-xyz | grep -A 5 "Last State"

# Get heap dump
kubectl exec -it api-xyz -- curl http://localhost:8080/debug/pprof/heap > heap.prof

# Check memory usage over time
kubectl top pod api-xyz --containers
```

**5. Network Debugging:**

```bash
# Test connectivity
kubectl run test --image=nicolaka/netshoot -it --rm -- bash

# Inside pod:
# DNS test
nslookup api.production.svc.cluster.local

# Connection test
curl -v http://api.production:8080

# Network trace
tcpdump -i any port 8080

# Check iptables rules
iptables-save | grep api
```

**6. PrometheusRule for Alerts:**

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: api-alerts
  namespace: production
spec:
  groups:
  - name: api
    interval: 30s
    rules:
    - alert: HighErrorRate
      expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "High error rate detected"
    - alert: HighMemoryUsage
      expr: container_memory_usage_bytes{pod=~"api-.*"} / container_spec_memory_limit_bytes > 0.9
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "Pod {{ $labels.pod }} using > 90% memory"
```

### Q55: Implement multi-tenancy with security

**Answer:**

**1. Namespace Isolation:**

```yaml
# Tenant 1 namespace
apiVersion: v1
kind: Namespace
metadata:
  name: tenant-a
  labels:
    tenant: tenant-a
---
# Tenant 2 namespace
apiVersion: v1
kind: Namespace
metadata:
  name: tenant-b
  labels:
    tenant: tenant-b
```

**2. Network Policies:**

```yaml
# Deny all traffic by default
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: tenant-a
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
---
# Allow within namespace only
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-same-namespace
  namespace: tenant-a
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector: {}
  egress:
  - to:
    - podSelector: {}
  - to:  # Allow DNS
    - namespaceSelector:
        matchLabels:
          name: kube-system
    ports:
    - protocol: UDP
      port: 53
```

**3. RBAC for Tenants:**

```yaml
# Tenant A admin role
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: tenant-admin
  namespace: tenant-a
rules:
- apiGroups: ["", "apps", "batch"]
  resources: ["*"]
  verbs: ["*"]
- apiGroups: ["networking.k8s.io"]
  resources: ["ingresses", "networkpolicies"]
  verbs: ["*"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: tenant-a-admin-binding
  namespace: tenant-a
subjects:
- kind: User
  name: tenant-a-admin
  apiGroup: rbac.authorization.k8s.io
- kind: Group
  name: tenant-a-admins
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: tenant-admin
  apiGroup: rbac.authorization.k8s.io
---
# Prevent cross-namespace access
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: tenant-limited
rules:
- apiGroups: [""]
  resources: ["namespaces"]
  verbs: ["get", "list"]
  resourceNames: ["tenant-a"]  # Only their namespace
```

**4. Resource Quotas per Tenant:**

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: tenant-quota
  namespace: tenant-a
spec:
  hard:
    requests.cpu: "20"
    requests.memory: 40Gi
    limits.cpu: "40"
    limits.memory: 80Gi
    persistentvolumeclaims: "10"
    services.loadbalancers: "2"
    count/deployments.apps: "20"
    count/jobs.batch: "10"
```

**5. Pod Security Standards:**

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: tenant-a
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

**6. Separate Ingress per Tenant:**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tenant-a-ingress
  namespace: tenant-a
  annotations:
    nginx.ingress.kubernetes.io/whitelist-source-range: "203.0.113.0/24"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - tenant-a.example.com
    secretName: tenant-a-tls
  rules:
  - host: tenant-a.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: tenant-a-app
            port:
              number: 80
```

---

## Additional Best Practices

### Security Best Practices

1. **Use RBAC**: Implement least-privilege access
2. **Network Policies**: Restrict pod-to-pod communication
3. **Pod Security Standards**: Enforce security constraints
4. **Secrets Management**: Use external secret managers (Vault, AWS Secrets Manager)
5. **Image Security**: Scan images for vulnerabilities
6. **Audit Logging**: Enable and monitor audit logs
7. **mTLS**: Use service mesh for encryption (Istio, Linkerd)

### Operational Best Practices

1. **Monitoring**: Prometheus, Grafana, ELK stack
2. **Logging**: Centralized logging (Fluentd, Loki)
3. **GitOps**: Use ArgoCD or Flux for deployments
4. **CI/CD Integration**: Automated pipelines
5. **Cost Optimization**: Resource right-sizing, spot instances
6. **Disaster Recovery**: Regular backups, multi-region
7. **Documentation**: Keep runbooks and architecture diagrams

---

This comprehensive guide covers essential Kubernetes concepts for interviews and real-world usage. Practice these examples in a test cluster to gain hands-on experience.
