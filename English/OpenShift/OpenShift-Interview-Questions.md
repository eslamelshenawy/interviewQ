# OpenShift Interview Questions and Answers

## Table of Contents
1. [OpenShift Fundamentals](#openshift-fundamentals)
2. [OpenShift vs Kubernetes](#openshift-vs-kubernetes)
3. [Projects vs Namespaces](#projects-vs-namespaces)
4. [Routes vs Ingress](#routes-vs-ingress)
5. [DeploymentConfig vs Deployment](#deploymentconfig-vs-deployment)
6. [Source-to-Image (S2I)](#source-to-image-s2i)
7. [BuildConfig](#buildconfig)
8. [ImageStreams](#imagestreams)
9. [Security Context Constraints (SCC)](#security-context-constraints-scc)
10. [oc CLI Commands](#oc-cli-commands)
11. [Real-World Scenarios](#real-world-scenarios)

---

## OpenShift Fundamentals

### Q1: What is OpenShift?
**Answer**: OpenShift is Red Hat's enterprise Kubernetes platform that provides a complete container application platform. It extends Kubernetes with developer and operational tools, integrated CI/CD, built-in security, and automated operations.

Key components:
- Kubernetes core for orchestration
- Integrated container registry
- CI/CD pipelines (Tekton)
- Developer and administrator web consoles
- Source-to-Image (S2I) build process
- Enhanced security with SCC

### Q2: What are the main components of OpenShift architecture?
**Answer**:
- **Master Nodes**: Control plane components (API server, scheduler, controller manager)
- **Worker Nodes**: Run application containers
- **etcd**: Distributed key-value store for cluster state
- **Container Registry**: Internal image registry
- **Router**: HAProxy-based ingress controller
- **Web Console**: UI for cluster management

---

## OpenShift vs Kubernetes

### Q3: What are the key differences between OpenShift and Kubernetes?

**Answer**:

| Aspect | Kubernetes | OpenShift |
|--------|-----------|-----------|
| **Origin** | Google, CNCF | Red Hat (built on K8s) |
| **Installation** | Manual, DIY | Automated installer (IPI/UPI) |
| **Security** | Basic RBAC, PSP | Enhanced RBAC, SCC, built-in OAuth |
| **Routing** | Ingress controller | Routes + Ingress |
| **CLI** | kubectl | oc (includes kubectl features) |
| **Registry** | External (optional) | Integrated registry |
| **Build System** | None (external CI/CD) | S2I, BuildConfig, Pipelines |
| **Web UI** | Basic Dashboard | Rich Developer & Admin consoles |
| **Updates** | Manual | Automated operator-based |
| **Support** | Community | Enterprise support from Red Hat |
| **Monitoring** | Prometheus (addon) | Integrated monitoring stack |

### Q4: Why choose OpenShift over vanilla Kubernetes?

**Answer**:
- **Enterprise Support**: Red Hat provides 24/7 support
- **Security**: Built-in security features and compliance certifications
- **Developer Productivity**: S2I, integrated CI/CD, developer console
- **Operational Simplicity**: Automated updates, integrated monitoring
- **Multi-tenancy**: Better isolation with Projects and SCC
- **Consistency**: Runs identically across cloud providers

---

## Projects vs Namespaces

### Q5: What is the difference between Projects and Namespaces?

**Answer**:

**Namespaces** (Kubernetes):
- Logical isolation of resources
- Basic RBAC support
- Resource quota support

**Projects** (OpenShift):
- OpenShift abstraction over namespaces
- Additional annotations and metadata
- Self-service provisioning
- Built-in network policies
- Project-level RBAC with automatic role bindings

### Q6: How do you create a Project?

**Answer**:

Using `oc` CLI:
```bash
# Create a new project
oc new-project my-application

# Create with description and display name
oc new-project my-app \
  --description="My Application" \
  --display-name="My Application"

# View current project
oc project

# Switch to another project
oc project another-project

# List all projects
oc projects
```

Using YAML:
```yaml
apiVersion: project.openshift.io/v1
kind: Project
metadata:
  name: my-application
  annotations:
    openshift.io/description: "My Application Project"
    openshift.io/display-name: "My Application"
spec:
  finalizers:
  - kubernetes
```

Apply:
```bash
oc apply -f project.yaml
```

### Q7: How do you set resource quotas on a Project?

**Answer**:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: project-quota
  namespace: my-application
spec:
  hard:
    requests.cpu: "10"
    requests.memory: 20Gi
    limits.cpu: "20"
    limits.memory: 40Gi
    persistentvolumeclaims: "5"
    pods: "50"
```

Apply:
```bash
oc apply -f resource-quota.yaml -n my-application

# View quota
oc get quota -n my-application
oc describe quota project-quota -n my-application
```

---

## Routes vs Ingress

### Q8: What is the difference between Routes and Ingress?

**Answer**:

**Routes** (OpenShift):
- Native OpenShift resource
- Simpler configuration
- Built-in load balancing via HAProxy
- Supports multiple TLS termination types
- Hostname-based routing

**Ingress** (Kubernetes):
- Standard Kubernetes resource
- Requires ingress controller
- More complex configuration
- Platform-agnostic

OpenShift supports both, but Routes are recommended for OpenShift-specific deployments.

### Q9: How do you create a Route?

**Answer**:

**Using CLI:**
```bash
# Simple route exposure
oc expose service my-service

# With custom hostname
oc expose service my-service --hostname=myapp.example.com

# With path-based routing
oc expose service my-service --path=/api
```

**Using YAML - HTTP Route:**
```yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: my-route
  namespace: my-application
spec:
  host: myapp.example.com
  to:
    kind: Service
    name: my-service
    weight: 100
  port:
    targetPort: 8080
```

**YAML - HTTPS Route with Edge Termination:**
```yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: secure-route
spec:
  host: secure.myapp.example.com
  to:
    kind: Service
    name: my-service
  port:
    targetPort: 8080
  tls:
    termination: edge
    insecureEdgeTerminationPolicy: Redirect
```

**YAML - Passthrough Termination:**
```yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: passthrough-route
spec:
  host: secure.myapp.example.com
  to:
    kind: Service
    name: my-service
  port:
    targetPort: 8443
  tls:
    termination: passthrough
```

**YAML - Re-encryption:**
```yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: reencrypt-route
spec:
  host: secure.myapp.example.com
  to:
    kind: Service
    name: my-service
  port:
    targetPort: 8443
  tls:
    termination: reencrypt
    destinationCACertificate: |
      -----BEGIN CERTIFICATE-----
      ... backend CA certificate ...
      -----END CERTIFICATE-----
```

### Q10: What are the TLS termination types in Routes?

**Answer**:

1. **Edge**: TLS terminates at the router, traffic to backend is HTTP
2. **Passthrough**: Encrypted traffic passes through router to backend
3. **Re-encrypt**: TLS terminates at router, re-encrypted to backend

```bash
# Edge termination
oc create route edge my-route --service=my-service

# Passthrough termination
oc create route passthrough my-route --service=my-service

# Re-encrypt termination
oc create route reencrypt my-route --service=my-service \
  --dest-ca-cert=backend-ca.crt
```

---

## DeploymentConfig vs Deployment

### Q11: What is the difference between DeploymentConfig and Deployment?

**Answer**:

**DeploymentConfig** (OpenShift):
- OpenShift-specific resource
- Automatic rollout triggers (ImageChange, ConfigChange)
- Custom deployment strategies (Rolling, Recreate, Custom)
- Lifecycle hooks (pre, mid, post)
- Automatic rollback support

**Deployment** (Kubernetes):
- Standard Kubernetes resource
- Manual or GitOps-triggered updates
- Rolling update strategy
- More stable and widely supported
- Better performance at scale

**Recommendation**: Use Deployments for new applications unless you need DeploymentConfig-specific features.

### Q12: Show examples of DeploymentConfig and Deployment

**Answer**:

**DeploymentConfig:**
```yaml
apiVersion: apps.openshift.io/v1
kind: DeploymentConfig
metadata:
  name: myapp
  namespace: my-application
spec:
  replicas: 3
  selector:
    app: myapp
  strategy:
    type: Rolling
    rollingParams:
      updatePeriodSeconds: 1
      intervalSeconds: 1
      timeoutSeconds: 600
      maxUnavailable: 25%
      maxSurge: 25%
      pre:
        failurePolicy: Abort
        execNewPod:
          containerName: myapp
          command: ["/bin/sh", "-c", "echo Pre-deployment hook"]
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
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:latest
        ports:
        - containerPort: 8080
        env:
        - name: APP_ENV
          value: "production"
```

**Deployment:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: my-application
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: image-registry.openshift-image-registry.svc:5000/my-application/myapp:latest
        ports:
        - containerPort: 8080
        env:
        - name: APP_ENV
          value: "production"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
```

### Q13: How do you trigger a rollout?

**Answer**:

**For DeploymentConfig:**
```bash
# Manual rollout
oc rollout latest dc/myapp

# View rollout status
oc rollout status dc/myapp

# View rollout history
oc rollout history dc/myapp

# Rollback to previous version
oc rollout undo dc/myapp

# Rollback to specific revision
oc rollout undo dc/myapp --to-revision=2
```

**For Deployment:**
```bash
# Trigger rollout by updating image
oc set image deployment/myapp myapp=myapp:v2

# View rollout status
oc rollout status deployment/myapp

# View rollout history
oc rollout history deployment/myapp

# Rollback
oc rollout undo deployment/myapp

# Pause/Resume rollout
oc rollout pause deployment/myapp
oc rollout resume deployment/myapp
```

---

## Source-to-Image (S2I)

### Q14: What is Source-to-Image (S2I)?

**Answer**:
S2I is a framework that automates the process of building reproducible container images from application source code. It:

- Takes source code as input
- Uses a builder image (language-specific)
- Produces a ready-to-run container image
- No need to write Dockerfile
- Supports incremental builds
- Enables patching and security updates

**Workflow:**
1. Developer pushes source code
2. S2I build triggered
3. Builder image + source code → Application image
4. Image pushed to registry
5. Deployment triggered with new image

### Q15: How do you create an application using S2I?

**Answer**:

**Using CLI:**
```bash
# Basic S2I build from Git repository
oc new-app python:3.9~https://github.com/username/python-app

# With specific branch
oc new-app python:3.9~https://github.com/username/python-app#develop

# With context directory
oc new-app python:3.9~https://github.com/username/monorepo \
  --context-dir=python-app

# With environment variables
oc new-app python:3.9~https://github.com/username/python-app \
  -e DATABASE_URL=postgresql://db:5432/mydb

# With custom name
oc new-app python:3.9~https://github.com/username/python-app \
  --name=my-python-app
```

**Complete example:**
```bash
# Create new project
oc new-project python-app

# Create S2I application
oc new-app python:3.9~https://github.com/sclorg/django-ex

# Expose service
oc expose svc/django-ex

# Get route
oc get route django-ex

# Check build status
oc logs -f bc/django-ex

# Check deployment
oc get pods
```

### Q16: How do you create a custom S2I builder image?

**Answer**:

S2I builder images require specific scripts:
- `assemble`: Builds the application
- `run`: Runs the application
- `save-artifacts`: (Optional) Saves artifacts for incremental builds

**Directory structure:**
```
s2i-builder/
├── Dockerfile
└── s2i/
    └── bin/
        ├── assemble
        ├── run
        └── save-artifacts
```

**Dockerfile:**
```dockerfile
FROM registry.redhat.io/ubi8/ubi:latest

LABEL io.k8s.description="Custom S2I Builder" \
      io.k8s.display-name="Custom Builder" \
      io.openshift.s2i.scripts-url="image:///usr/libexec/s2i"

RUN yum install -y python39 python39-pip && \
    yum clean all

COPY s2i/bin/ /usr/libexec/s2i/

USER 1001

EXPOSE 8080
```

**assemble script:**
```bash
#!/bin/bash
echo "Installing application dependencies..."
pip install --no-cache-dir -r requirements.txt

echo "Building application..."
python setup.py install
```

**run script:**
```bash
#!/bin/bash
exec python app.py
```

---

## BuildConfig

### Q17: What is a BuildConfig?

**Answer**:
BuildConfig defines the build process in OpenShift. It specifies:
- Build strategy (Source, Docker, Custom, Pipeline)
- Source location (Git, binary, Dockerfile)
- Output image destination
- Build triggers (webhook, image change, config change)
- Environment variables and secrets

### Q18: What are the different build strategies?

**Answer**:

1. **Source Build (S2I)**: Builds from source code using builder image
2. **Docker Build**: Uses Dockerfile from source
3. **Custom Build**: Custom builder image with custom logic
4. **Pipeline Build**: Jenkins/Tekton pipeline (deprecated in 4.x)

### Q19: Show examples of different BuildConfig types

**Answer**:

**Source (S2I) Build:**
```yaml
apiVersion: build.openshift.io/v1
kind: BuildConfig
metadata:
  name: python-app
  namespace: my-application
spec:
  source:
    git:
      uri: https://github.com/username/python-app
      ref: main
    contextDir: /
  strategy:
    sourceStrategy:
      from:
        kind: ImageStreamTag
        name: python:3.9
      env:
      - name: PIP_INDEX_URL
        value: https://pypi.org/simple
  output:
    to:
      kind: ImageStreamTag
      name: python-app:latest
  triggers:
  - type: ConfigChange
  - type: ImageChange
  - type: GitHub
    github:
      secretReference:
        name: github-webhook-secret
```

**Docker Build:**
```yaml
apiVersion: build.openshift.io/v1
kind: BuildConfig
metadata:
  name: docker-app
spec:
  source:
    git:
      uri: https://github.com/username/docker-app
      ref: main
    contextDir: /
  strategy:
    dockerStrategy:
      dockerfilePath: Dockerfile
      env:
      - name: BUILD_ENV
        value: production
  output:
    to:
      kind: ImageStreamTag
      name: docker-app:latest
  triggers:
  - type: ConfigChange
  - type: GitHub
    github:
      secretReference:
        name: github-webhook-secret
```

**Binary Build (local source):**
```yaml
apiVersion: build.openshift.io/v1
kind: BuildConfig
metadata:
  name: binary-app
spec:
  source:
    binary: {}
    type: Binary
  strategy:
    sourceStrategy:
      from:
        kind: ImageStreamTag
        name: nodejs:16
  output:
    to:
      kind: ImageStreamTag
      name: binary-app:latest
```

Use with:
```bash
# Start binary build from local directory
oc start-build binary-app --from-dir=. --follow

# Start build from archive
oc start-build binary-app --from-file=app.tar.gz --follow
```

**Custom Build:**
```yaml
apiVersion: build.openshift.io/v1
kind: BuildConfig
metadata:
  name: custom-app
spec:
  source:
    git:
      uri: https://github.com/username/app
  strategy:
    customStrategy:
      from:
        kind: ImageStreamTag
        name: custom-builder:latest
      env:
      - name: CUSTOM_VAR
        value: "custom-value"
  output:
    to:
      kind: ImageStreamTag
      name: custom-app:latest
```

### Q20: How do you manage BuildConfig?

**Answer**:

```bash
# Create BuildConfig from file
oc create -f buildconfig.yaml

# Start a build
oc start-build python-app

# Start build with logs
oc start-build python-app --follow

# Start build from local directory
oc start-build python-app --from-dir=. --follow

# View builds
oc get builds

# View build logs
oc logs build/python-app-1

# Cancel a build
oc cancel-build python-app-1

# Delete BuildConfig and all builds
oc delete bc/python-app

# View build configuration
oc describe bc/python-app

# Edit BuildConfig
oc edit bc/python-app

# Set environment variable
oc set env bc/python-app DATABASE_URL=postgresql://db:5432/mydb

# Add webhook trigger
oc set triggers bc/python-app --from-github
oc describe bc/python-app | grep -i webhook
```

---

## ImageStreams

### Q21: What are ImageStreams?

**Answer**:
ImageStreams are OpenShift abstractions for referencing container images. They:

- Track image updates and changes
- Enable image promotion across environments
- Trigger automatic deployments on image updates
- Provide image lifecycle management
- Support image tagging and versioning
- Reference external or internal images

**Benefits:**
- Decouple image references from deployment
- Enable image security scanning integration
- Simplify image management across clusters
- Support image mirroring and caching

### Q22: How do you create and manage ImageStreams?

**Answer**:

**Create ImageStream:**
```yaml
apiVersion: image.openshift.io/v1
kind: ImageStream
metadata:
  name: myapp
  namespace: my-application
spec:
  lookupPolicy:
    local: true
  tags:
  - name: latest
    from:
      kind: DockerImage
      name: quay.io/username/myapp:latest
    importPolicy:
      scheduled: true
    referencePolicy:
      type: Source
```

**CLI Commands:**
```bash
# Create ImageStream
oc create imagestream myapp

# Import external image
oc import-image myapp:latest \
  --from=quay.io/username/myapp:latest \
  --confirm

# Import with scheduled updates (every 15 minutes)
oc import-image myapp:latest \
  --from=quay.io/username/myapp:latest \
  --scheduled=true \
  --confirm

# View ImageStreams
oc get imagestreams
oc get is

# View ImageStream details
oc describe is/myapp

# View ImageStream tags
oc get istag

# Tag image within ImageStream
oc tag myapp:latest myapp:v1.0

# Tag image from another project
oc tag other-project/myapp:latest myapp:latest

# Tag external image
oc tag docker.io/nginx:latest nginx:latest --scheduled

# Delete ImageStream
oc delete is/myapp
```

### Q23: How do you promote images across environments?

**Answer**:

**Scenario**: Promote image from dev → test → prod

```bash
# In development project
oc new-project dev
oc new-app python~https://github.com/username/app --name=myapp
oc expose svc/myapp

# Once development is stable, tag for testing
oc tag dev/myapp:latest test/myapp:test

# In test project (automatic deployment if ImageChange trigger exists)
oc new-project test
oc new-app --image-stream=myapp:test
oc expose svc/myapp

# After testing, promote to production
oc tag test/myapp:test prod/myapp:prod

# In production project
oc new-project prod
oc new-app --image-stream=myapp:prod
oc expose svc/myapp
```

**Using YAML for cross-project ImageStream:**
```yaml
apiVersion: image.openshift.io/v1
kind: ImageStream
metadata:
  name: myapp
  namespace: prod
spec:
  tags:
  - name: prod
    from:
      kind: ImageStreamTag
      namespace: test
      name: myapp:test
```

### Q24: How do you use ImageStreams with Deployments?

**Answer**:

**With DeploymentConfig (automatic trigger):**
```yaml
apiVersion: apps.openshift.io/v1
kind: DeploymentConfig
metadata:
  name: myapp
spec:
  replicas: 3
  triggers:
  - type: ImageChange
    imageChangeParams:
      automatic: true
      containerNames:
      - myapp
      from:
        kind: ImageStreamTag
        name: myapp:latest
  template:
    spec:
      containers:
      - name: myapp
        image: myapp:latest
```

**With Deployment (manual trigger):**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: myapp
        image: image-registry.openshift-image-registry.svc:5000/my-project/myapp:latest
```

Update Deployment when image changes:
```bash
oc set image deployment/myapp \
  myapp=image-registry.openshift-image-registry.svc:5000/my-project/myapp:latest
```

---

## Security Context Constraints (SCC)

### Q25: What are Security Context Constraints (SCC)?

**Answer**:
SCC is OpenShift's security mechanism that controls what actions pods can perform and what resources they can access. It's more fine-grained than Kubernetes PodSecurityPolicies.

**SCC controls:**
- User and group IDs that containers run as
- Capabilities that can be added/dropped
- Volume types that can be used
- Host network/port access
- SELinux context
- Privilege escalation
- Root filesystem access

### Q26: What are the default SCCs in OpenShift?

**Answer**:

| SCC | Description | Use Case |
|-----|-------------|----------|
| **restricted** | Most restrictive, default for all users | Standard applications |
| **restricted-v2** | Enhanced restricted (OpenShift 4.11+) | Modern applications |
| **nonroot** | Allows non-root user with specific UID | Apps needing specific UID |
| **anyuid** | Allows any UID | Legacy apps, databases |
| **hostaccess** | Allows host access | System monitoring |
| **hostmount-anyuid** | Host mounts + any UID | Storage plugins |
| **hostnetwork** | Access to host network | Network tools |
| **node-exporter** | For node exporters | Monitoring |
| **privileged** | Full privileges, no restrictions | System-level containers |

```bash
# List all SCCs
oc get scc

# View SCC details
oc describe scc restricted

# View which SCC is used by a pod
oc get pod <pod-name> -o yaml | grep openshift.io/scc
```

### Q27: How do you assign SCC to a Service Account?

**Answer**:

**Method 1: Using oc adm policy**
```bash
# Create service account
oc create sa myapp-sa

# Add SCC to service account
oc adm policy add-scc-to-user anyuid -z myapp-sa

# Add SCC to service account in specific namespace
oc adm policy add-scc-to-user anyuid -z myapp-sa -n my-project

# Remove SCC from service account
oc adm policy remove-scc-from-user anyuid -z myapp-sa

# View SCC for service account
oc get scc anyuid -o yaml
```

**Method 2: Using Role and RoleBinding**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: scc-anyuid
  namespace: my-application
rules:
- apiGroups:
  - security.openshift.io
  resources:
  - securitycontextconstraints
  resourceNames:
  - anyuid
  verbs:
  - use
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: sa-scc-anyuid
  namespace: my-application
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: scc-anyuid
subjects:
- kind: ServiceAccount
  name: myapp-sa
  namespace: my-application
```

**Use in Pod:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  serviceAccountName: myapp-sa
  containers:
  - name: myapp
    image: myapp:latest
    securityContext:
      runAsUser: 1000
```

### Q28: How do you create a custom SCC?

**Answer**:

```yaml
apiVersion: security.openshift.io/v1
kind: SecurityContextConstraints
metadata:
  name: custom-scc
allowHostDirVolumePlugin: false
allowHostIPC: false
allowHostNetwork: false
allowHostPID: false
allowHostPorts: false
allowPrivilegedContainer: false
allowPrivilegeEscalation: true
allowedCapabilities:
- NET_BIND_SERVICE
defaultAddCapabilities: null
fsGroup:
  type: MustRunAs
  ranges:
  - min: 1000
    max: 65535
priority: 10
readOnlyRootFilesystem: false
requiredDropCapabilities:
- KILL
- MKNOD
- SETUID
- SETGID
runAsUser:
  type: MustRunAsRange
  uidRangeMin: 1000
  uidRangeMax: 65535
seLinuxContext:
  type: MustRunAs
supplementalGroups:
  type: RunAsAny
volumes:
- configMap
- downwardAPI
- emptyDir
- persistentVolumeClaim
- projected
- secret
```

Apply and assign:
```bash
# Create custom SCC
oc apply -f custom-scc.yaml

# Assign to service account
oc adm policy add-scc-to-user custom-scc -z myapp-sa

# Verify
oc describe scc custom-scc
```

---

## oc CLI Commands

### Q29: What are the essential oc commands?

**Answer**:

**Authentication & Projects:**
```bash
# Login
oc login https://api.cluster.example.com:6443 --token=<token>
oc login https://api.cluster.example.com:6443 -u username -p password

# Logout
oc logout

# Get current context
oc whoami
oc whoami --show-server
oc whoami --show-token
oc whoami --show-console

# Projects
oc projects
oc project my-app
oc new-project my-app --display-name="My Application"
oc delete project my-app
```

**Resource Management:**
```bash
# Get resources
oc get all
oc get pods
oc get pods -o wide
oc get pods --all-namespaces
oc get pods -w  # watch mode
oc get pods --show-labels
oc get pods -l app=myapp

# Describe resources
oc describe pod <pod-name>
oc describe svc <service-name>

# Create resources
oc create -f resource.yaml
oc apply -f resource.yaml
oc create -f https://raw.githubusercontent.com/user/repo/main/resource.yaml

# Delete resources
oc delete pod <pod-name>
oc delete all -l app=myapp
oc delete -f resource.yaml

# Edit resources
oc edit pod <pod-name>
oc edit deployment <deployment-name>
```

**Application Deployment:**
```bash
# Create application from source
oc new-app python~https://github.com/user/python-app
oc new-app --docker-image=nginx
oc new-app --image-stream=myapp:latest

# Expose service
oc expose svc/myapp
oc expose svc/myapp --hostname=myapp.example.com

# Scale
oc scale deployment/myapp --replicas=5
oc scale dc/myapp --replicas=3

# Autoscale
oc autoscale deployment/myapp --min=2 --max=10 --cpu-percent=80
```

**Build & Deploy:**
```bash
# Start build
oc start-build myapp
oc start-build myapp --follow
oc start-build myapp --from-dir=.

# Build status
oc get builds
oc logs build/myapp-1

# Deployment
oc rollout latest dc/myapp
oc rollout status deployment/myapp
oc rollout undo deployment/myapp
oc rollout history deployment/myapp
oc rollout pause deployment/myapp
oc rollout resume deployment/myapp
```

**Debugging:**
```bash
# Logs
oc logs <pod-name>
oc logs <pod-name> -c <container-name>
oc logs -f <pod-name>  # follow
oc logs <pod-name> --previous  # previous container

# Execute commands
oc exec <pod-name> -- ls /app
oc exec -it <pod-name> -- /bin/bash

# Remote shell
oc rsh <pod-name>

# Port forwarding
oc port-forward <pod-name> 8080:8080

# Copy files
oc cp <pod-name>:/path/to/file ./local-file
oc cp ./local-file <pod-name>:/path/to/file

# Debug
oc debug deployment/myapp
oc debug node/<node-name>

# Events
oc get events
oc get events --sort-by='.lastTimestamp'
```

**Configuration:**
```bash
# ConfigMaps
oc create configmap myconfig --from-file=config.properties
oc create configmap myconfig --from-literal=key1=value1
oc get configmap myconfig -o yaml

# Secrets
oc create secret generic mysecret --from-literal=password=secret123
oc create secret docker-registry regcred \
  --docker-server=quay.io \
  --docker-username=user \
  --docker-password=pass

# Environment variables
oc set env deployment/myapp DATABASE_URL=postgresql://db:5432/mydb
oc set env deployment/myapp --list
oc set env deployment/myapp --from=configmap/myconfig
oc set env deployment/myapp --from=secret/mysecret
```

**Image & Registry:**
```bash
# Image streams
oc get is
oc import-image myapp:latest --from=quay.io/user/myapp:latest --confirm
oc tag myapp:latest myapp:v1.0

# Registry
oc registry info
oc registry login

# Image policy
oc set image-lookup myapp
oc set image deployment/myapp myapp=myapp:v2
```

**Monitoring:**
```bash
# Resource usage
oc adm top nodes
oc adm top pods
oc adm top pod <pod-name> --containers

# Cluster info
oc cluster-info
oc version
oc api-resources
oc api-versions

# Node management
oc get nodes
oc describe node <node-name>
oc adm drain <node-name>
oc adm uncordon <node-name>
```

**Advanced:**
```bash
# Process templates
oc process -f template.yaml -p PARAM1=value1 | oc apply -f -

# Export resources
oc get deployment/myapp -o yaml --export > deployment.yaml

# Patch resources
oc patch deployment/myapp -p '{"spec":{"replicas":3}}'

# Label resources
oc label pod <pod-name> environment=production
oc label pod <pod-name> environment-  # remove label

# Annotate resources
oc annotate pod <pod-name> description="My application"

# Policy & RBAC
oc adm policy add-role-to-user admin user1 -n my-project
oc adm policy add-scc-to-user anyuid -z myapp-sa
oc adm policy who-can delete pods

# Extract secrets
oc extract secret/mysecret --to=./secrets/

# Service accounts
oc create sa myapp-sa
oc get sa
oc describe sa myapp-sa
```

### Q30: What are the differences between kubectl and oc?

**Answer**:

`oc` is a superset of `kubectl` with additional OpenShift-specific features:

**oc-specific commands:**
```bash
# Login (not in kubectl)
oc login

# Projects (namespaces with extras)
oc new-project
oc project

# Application creation
oc new-app
oc new-build

# Routes (OpenShift-specific)
oc expose
oc create route

# Builds
oc start-build

# Rollouts with DeploymentConfig
oc rollout latest dc/myapp

# Registry
oc registry

# Cluster administration
oc adm
```

**kubectl equivalent:**
```bash
# Use kubeconfig
kubectl config use-context <context>
kubectl config set-context --current --namespace=<namespace>

# kubectl can be used in OpenShift
kubectl get pods
kubectl apply -f deployment.yaml
```

---

## Real-World Scenarios

### Q31: Scenario - Deploy a microservices application with database

**Answer**:

**Architecture**: Frontend (React) → Backend (Node.js) → Database (PostgreSQL)

```bash
# Create project
oc new-project microservices-app

# Deploy PostgreSQL
oc new-app postgresql-persistent \
  -p DATABASE_SERVICE_NAME=postgresql \
  -p POSTGRESQL_USER=admin \
  -p POSTGRESQL_PASSWORD=secret \
  -p POSTGRESQL_DATABASE=appdb

# Deploy Backend (Node.js)
oc new-app nodejs~https://github.com/username/backend-api \
  --name=backend \
  -e DATABASE_URL=postgresql://admin:secret@postgresql:5432/appdb

# Expose backend internally only (no route)
oc expose svc/backend

# Deploy Frontend (React)
oc new-app nodejs~https://github.com/username/frontend-app \
  --name=frontend \
  -e REACT_APP_API_URL=http://backend:8080

# Expose frontend externally
oc expose svc/frontend --hostname=myapp.example.com

# Check status
oc get all
oc get routes
```

**With YAML:**

**backend-deployment.yaml:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: image-registry.openshift-image-registry.svc:5000/microservices-app/backend:latest
        ports:
        - containerPort: 8080
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: url
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  selector:
    app: backend
  ports:
  - port: 8080
    targetPort: 8080
```

### Q32: Scenario - Implement Blue-Green Deployment

**Answer**:

Blue-Green deployment for zero-downtime releases:

```bash
# Deploy Blue version (current production)
oc new-app nodejs~https://github.com/user/app#v1.0 --name=app-blue
oc expose svc/app-blue

# Create route pointing to blue
oc create route edge production --service=app-blue

# Deploy Green version (new version)
oc new-app nodejs~https://github.com/user/app#v2.0 --name=app-green

# Test green version internally
oc expose svc/app-green --name=app-green-test

# Switch traffic to green
oc patch route/production -p '{"spec":{"to":{"name":"app-green"}}}'

# Rollback if needed
oc patch route/production -p '{"spec":{"to":{"name":"app-blue"}}}'

# Delete old blue version after successful deployment
oc delete all -l app=app-blue
```

**YAML approach:**
```yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: production
spec:
  host: myapp.example.com
  to:
    kind: Service
    name: app-blue
    weight: 100
  alternateBackends:
  - kind: Service
    name: app-green
    weight: 0
  tls:
    termination: edge
```

**Switch traffic:**
```bash
# Gradual shift: 90% blue, 10% green
oc patch route/production -p '
{
  "spec": {
    "to": {"name": "app-blue", "weight": 90},
    "alternateBackends": [{"name": "app-green", "weight": 10}]
  }
}'

# Full switch to green
oc patch route/production -p '
{
  "spec": {
    "to": {"name": "app-green", "weight": 100},
    "alternateBackends": [{"name": "app-blue", "weight": 0}]
  }
}'
```

### Q33: Scenario - Setup CI/CD Pipeline with GitHub Webhooks

**Answer**:

**Step 1: Create BuildConfig with webhook**
```bash
# Create application
oc new-app python~https://github.com/user/app --name=myapp

# Get webhook URL
oc describe bc/myapp | grep -A 1 "Webhook GitHub"
# Example: https://api.cluster.com:6443/apis/build.openshift.io/v1/namespaces/myproject/buildconfigs/myapp/webhooks/<secret>/github
```

**Step 2: Configure GitHub Webhook**
1. Go to GitHub repository → Settings → Webhooks
2. Add webhook URL from above
3. Content type: application/json
4. Events: Push events
5. Save

**Step 3: Test webhook**
```bash
# Make code change and push
git add .
git commit -m "Update application"
git push origin main

# Watch build automatically trigger
oc get builds -w
oc logs -f bc/myapp
```

**Complete CI/CD with OpenShift Pipelines (Tekton):**

**pipeline.yaml:**
```yaml
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: build-and-deploy
spec:
  params:
  - name: GIT_REPO
    type: string
  - name: GIT_REVISION
    type: string
    default: main
  - name: IMAGE_NAME
    type: string
  workspaces:
  - name: shared-workspace
  tasks:
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

  - name: build-image
    taskRef:
      name: buildah
    runAfter:
    - fetch-repository
    workspaces:
    - name: source
      workspace: shared-workspace
    params:
    - name: IMAGE
      value: $(params.IMAGE_NAME)

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

### Q34: Scenario - Secure application with SSL and Secrets

**Answer**:

**Step 1: Create TLS Secret**
```bash
# Create TLS secret from certificate files
oc create secret tls myapp-tls \
  --cert=path/to/tls.crt \
  --key=path/to/tls.key

# Or create from literal
oc create secret generic app-secrets \
  --from-literal=api-key=abc123 \
  --from-literal=db-password=secret
```

**Step 2: Create Secure Route**
```yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: secure-route
spec:
  host: secure.myapp.example.com
  to:
    kind: Service
    name: myapp
  port:
    targetPort: 8443
  tls:
    termination: edge
    certificate: |
      -----BEGIN CERTIFICATE-----
      ...
      -----END CERTIFICATE-----
    key: |
      -----BEGIN PRIVATE KEY-----
      ...
      -----END PRIVATE KEY-----
    insecureEdgeTerminationPolicy: Redirect
```

**Step 3: Use Secrets in Deployment**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    spec:
      containers:
      - name: myapp
        image: myapp:latest
        env:
        - name: API_KEY
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: api-key
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: db-password
        volumeMounts:
        - name: tls-certs
          mountPath: /etc/tls
          readOnly: true
      volumes:
      - name: tls-certs
        secret:
          secretName: myapp-tls
```

**Sealed Secrets (for GitOps):**
```bash
# Install Sealed Secrets controller
oc apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.18.0/controller.yaml

# Install kubeseal CLI
# Create sealed secret
kubeseal --format yaml < secret.yaml > sealed-secret.yaml

# Commit sealed-secret.yaml to Git (safe)
git add sealed-secret.yaml
git commit -m "Add sealed secret"
```

### Q35: Scenario - Troubleshoot failing deployment

**Answer**:

**Issue**: Pods are in CrashLoopBackOff

**Step 1: Check pod status**
```bash
oc get pods
# NAME                     READY   STATUS             RESTARTS   AGE
# myapp-5d4c8f6b7-xyz12    0/1     CrashLoopBackOff   5          5m
```

**Step 2: Check pod events**
```bash
oc describe pod myapp-5d4c8f6b7-xyz12
# Look for events at bottom
```

**Step 3: Check logs**
```bash
# Current container logs
oc logs myapp-5d4c8f6b7-xyz12

# Previous container logs (if crashed)
oc logs myapp-5d4c8f6b7-xyz12 --previous

# All containers in pod
oc logs myapp-5d4c8f6b7-xyz12 --all-containers
```

**Step 4: Check deployment configuration**
```bash
oc describe deployment myapp
oc get deployment myapp -o yaml
```

**Common Issues & Solutions:**

**Issue 1: Image Pull Error**
```bash
# Check image pull secret
oc get pods -o yaml | grep -i imagepull

# Create image pull secret
oc create secret docker-registry regcred \
  --docker-server=quay.io \
  --docker-username=user \
  --docker-password=pass

# Link to service account
oc secrets link default regcred --for=pull
```

**Issue 2: SCC Permission Denied**
```bash
# Check which SCC is being used
oc get pod myapp-xyz12 -o yaml | grep scc

# Check pod security context
oc get pod myapp-xyz12 -o yaml | grep -A 10 securityContext

# Grant required SCC
oc create sa myapp-sa
oc adm policy add-scc-to-user anyuid -z myapp-sa
oc set serviceaccount deployment/myapp myapp-sa
```

**Issue 3: Resource Limits**
```bash
# Check resource usage
oc adm top pod myapp-xyz12

# Check node resources
oc adm top nodes

# Increase resources
oc set resources deployment/myapp \
  --limits=cpu=500m,memory=512Mi \
  --requests=cpu=250m,memory=256Mi
```

**Issue 4: Configuration Error**
```bash
# Check ConfigMap
oc get configmap myconfig -o yaml

# Check Secret
oc get secret mysecret -o yaml

# Verify environment variables
oc set env deployment/myapp --list

# Debug with temporary pod
oc debug deployment/myapp
```

**Step 5: Check events**
```bash
# Get all events sorted by time
oc get events --sort-by='.lastTimestamp'

# Filter events for specific pod
oc get events --field-selector involvedObject.name=myapp-xyz12
```

### Q36: Scenario - Implement autoscaling based on metrics

**Answer**:

**Horizontal Pod Autoscaler (HPA):**

```bash
# Create HPA based on CPU
oc autoscale deployment/myapp --min=2 --max=10 --cpu-percent=70

# Create HPA based on memory
oc autoscale deployment/myapp --min=2 --max=10 --memory-percent=80

# Check HPA status
oc get hpa
oc describe hpa myapp
```

**HPA YAML:**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 2
  maxReplicas: 10
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
        periodSeconds: 15
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15
      - type: Pods
        value: 4
        periodSeconds: 15
      selectPolicy: Max
```

**Custom Metrics (using Prometheus):**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-custom-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
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

**Vertical Pod Autoscaler (VPA):**
```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: myapp-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
    - containerName: myapp
      minAllowed:
        cpu: 100m
        memory: 128Mi
      maxAllowed:
        cpu: 1
        memory: 1Gi
```

**Test autoscaling:**
```bash
# Generate load
oc run load-generator --image=busybox --restart=Never -- /bin/sh -c "while true; do wget -q -O- http://myapp; done"

# Watch HPA
oc get hpa myapp --watch

# Watch pods scaling
oc get pods -w
```

### Q37: Scenario - Multi-environment deployment strategy

**Answer**:

**Setup: Dev → Test → Staging → Production**

**Step 1: Create base configuration**

**kustomization/base/deployment.yaml:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 1
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
        image: image-registry.openshift-image-registry.svc:5000/dev/myapp:latest
        env:
        - name: ENVIRONMENT
          value: "dev"
```

**Step 2: Environment overlays**

**kustomization/overlays/dev/kustomization.yaml:**
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: dev
bases:
- ../../base
patchesStrategicMerge:
- deployment-patch.yaml
images:
- name: myapp
  newTag: latest
```

**kustomization/overlays/prod/deployment-patch.yaml:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 5
  template:
    spec:
      containers:
      - name: myapp
        env:
        - name: ENVIRONMENT
          value: "production"
        resources:
          limits:
            cpu: 1
            memory: 1Gi
          requests:
            cpu: 500m
            memory: 512Mi
```

**Step 3: Deploy to environments**
```bash
# Development
oc apply -k kustomization/overlays/dev

# Production
oc apply -k kustomization/overlays/prod
```

**GitOps with ArgoCD:**

**argocd-app.yaml:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-dev
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/user/app-config
    targetRevision: main
    path: kustomization/overlays/dev
  destination:
    server: https://kubernetes.default.svc
    namespace: dev
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
---
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-prod
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/user/app-config
    targetRevision: main
    path: kustomization/overlays/prod
  destination:
    server: https://kubernetes.default.svc
    namespace: prod
  syncPolicy:
    automated:
      prune: false
      selfHeal: false
```

---

## Additional Interview Questions

### Q38: How do you backup and restore OpenShift applications?

**Answer**:

**Using Velero (formerly Heptio Ark):**

```bash
# Install Velero
velero install \
  --provider aws \
  --bucket openshift-backups \
  --secret-file ./credentials-velero

# Backup specific namespace
velero backup create myapp-backup --include-namespaces=my-application

# Backup with label selector
velero backup create myapp-backup --selector app=myapp

# Schedule automatic backups
velero schedule create daily-backup --schedule="0 2 * * *" --include-namespaces=my-application

# List backups
velero backup get

# Restore from backup
velero restore create --from-backup myapp-backup

# Restore to different namespace
velero restore create --from-backup myapp-backup \
  --namespace-mappings my-application:my-application-restored
```

### Q39: How do you implement network policies in OpenShift?

**Answer**:

**Deny all traffic by default:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: my-application
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

**Allow specific ingress:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
  namespace: my-application
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

**Allow egress to database:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-db-egress
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: postgresql
    ports:
    - protocol: TCP
      port: 5432
```

### Q40: How do you monitor OpenShift applications?

**Answer**:

OpenShift includes built-in monitoring with Prometheus and Grafana.

**Enable user workload monitoring:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cluster-monitoring-config
  namespace: openshift-monitoring
data:
  config.yaml: |
    enableUserWorkload: true
```

**Create ServiceMonitor:**
```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: myapp-monitor
  namespace: my-application
spec:
  selector:
    matchLabels:
      app: myapp
  endpoints:
  - port: metrics
    interval: 30s
```

**Query metrics:**
```bash
# Access Prometheus
oc get route -n openshift-monitoring

# Query via CLI
oc exec -n openshift-monitoring prometheus-k8s-0 -- \
  promtool query instant http://localhost:9090 \
  'up{namespace="my-application"}'
```

**Create PrometheusRule for alerts:**
```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: myapp-alerts
  namespace: my-application
spec:
  groups:
  - name: myapp
    interval: 30s
    rules:
    - alert: HighErrorRate
      expr: rate(http_requests_total{status="500"}[5m]) > 0.05
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "High error rate detected"
        description: "Error rate is {{ $value }} requests/sec"
```

---

This comprehensive guide covers the most important OpenShift concepts and real-world scenarios you'll encounter in interviews and production environments. Practice these commands and scenarios in an OpenShift cluster for hands-on experience.
