# OpenShift Documentation

## Overview

Red Hat OpenShift is an enterprise-ready Kubernetes container platform with full-stack automated operations to manage hybrid cloud, multicloud, and edge deployments. OpenShift includes Kubernetes along with additional developer and operations-centric tools.

## Table of Contents

1. [What is OpenShift?](#what-is-openshift)
2. [Key Features](#key-features)
3. [Architecture](#architecture)
4. [Core Concepts](#core-concepts)
5. [Getting Started](#getting-started)
6. [Additional Resources](#additional-resources)

## What is OpenShift?

OpenShift is Red Hat's Platform-as-a-Service (PaaS) offering built on top of Kubernetes. It provides:

- **Enhanced Developer Experience**: Simplified application deployment and management
- **Enterprise Security**: Built-in security features and compliance tools
- **Automated Operations**: GitOps, CI/CD integration, and automated updates
- **Multi-cloud Support**: Runs consistently across cloud providers and on-premises

## Key Features

### 1. **Developer Tools**
- Source-to-Image (S2I) for building container images from source code
- Integrated CI/CD with OpenShift Pipelines (Tekton)
- Developer Console with topology view
- Built-in image registry

### 2. **Security**
- Security Context Constraints (SCC)
- Role-Based Access Control (RBAC)
- Image scanning and vulnerability detection
- Network policies and encryption

### 3. **Routing and Networking**
- Routes for external access
- Service mesh integration (Istio)
- Software Defined Networking (SDN)
- Load balancing

### 4. **Build and Deploy**
- BuildConfig for automated builds
- DeploymentConfig and Deployment support
- ImageStreams for image management
- Automated rollouts and rollbacks

## Architecture

```
┌─────────────────────────────────────────────────┐
│             OpenShift Platform                   │
├─────────────────────────────────────────────────┤
│  Developer Tools  │  CI/CD  │  Monitoring       │
├─────────────────────────────────────────────────┤
│              OpenShift Extensions                │
│  Routes │ S2I │ BuildConfig │ ImageStreams │SCC │
├─────────────────────────────────────────────────┤
│              Kubernetes Core                     │
│  Pods │ Services │ Deployments │ ConfigMaps     │
├─────────────────────────────────────────────────┤
│              Container Runtime                   │
│              (CRI-O / Docker)                    │
├─────────────────────────────────────────────────┤
│         Operating System (RHEL CoreOS)          │
└─────────────────────────────────────────────────┘
```

## Core Concepts

### Projects vs Namespaces
- **Projects**: OpenShift's abstraction over Kubernetes namespaces with additional features
- Provides multi-tenancy and access control
- Includes default network policies and resource quotas

### Routes vs Ingress
- **Routes**: OpenShift-native way to expose services externally
- Simpler configuration than Kubernetes Ingress
- Supports TLS termination, edge, passthrough, and re-encryption

### DeploymentConfig vs Deployment
- **DeploymentConfig**: OpenShift-specific deployment resource with additional triggers
- Supports automatic redeployment on image or config changes
- **Deployment**: Standard Kubernetes deployment resource

### Source-to-Image (S2I)
- Automated process to build reproducible container images
- Takes source code and base builder image as input
- Produces ready-to-run container image

### BuildConfig
- Defines the build process for applications
- Supports multiple build strategies (Source, Docker, Custom, Pipeline)
- Can trigger builds automatically on code changes

### ImageStreams
- Abstraction for referencing container images
- Tracks image updates and triggers deployments
- Enables image promotion across environments

### Security Context Constraints (SCC)
- OpenShift's mechanism to control pod permissions
- More fine-grained than Kubernetes PodSecurityPolicies
- Defines what pods can do and what resources they can access

## Getting Started

### Prerequisites
- Access to an OpenShift cluster
- `oc` CLI tool installed
- Basic Kubernetes knowledge

### Basic Commands

```bash
# Login to OpenShift
oc login https://api.cluster-url:6443 --token=<your-token>

# Create a new project
oc new-project my-app

# Deploy an application using S2I
oc new-app python~https://github.com/openshift/django-ex

# Expose the service
oc expose svc/django-ex

# Get route URL
oc get route
```

### Common Operations

```bash
# List all projects
oc projects

# Switch project
oc project my-app

# View pods
oc get pods

# View logs
oc logs <pod-name>

# Execute command in pod
oc rsh <pod-name>

# Scale application
oc scale dc/<name> --replicas=3

# Delete application
oc delete all -l app=<app-name>
```

## OpenShift vs Kubernetes

| Feature | Kubernetes | OpenShift |
|---------|-----------|-----------|
| **Installation** | Manual, complex | Automated installer |
| **Security** | PodSecurityPolicy | Security Context Constraints |
| **Routing** | Ingress | Routes (+ Ingress) |
| **Image Builds** | External tools | Built-in (S2I, BuildConfig) |
| **Registry** | External | Integrated registry |
| **CLI** | kubectl | oc (superset of kubectl) |
| **Web Console** | Dashboard (addon) | Integrated, feature-rich |
| **Updates** | Manual | Automated over-the-air |

## Additional Resources

- [Interview Questions](./OpenShift-Interview-Questions.md)
- [Official Documentation](https://docs.openshift.com/)
- [OpenShift Interactive Learning Portal](https://learn.openshift.com/)
- [OpenShift Blog](https://www.openshift.com/blog)

## Version Information

This documentation covers OpenShift 4.x features. Some features may vary by version.

---

**Note**: Always refer to the official Red Hat OpenShift documentation for the most up-to-date and version-specific information.
