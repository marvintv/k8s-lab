# Kubernetes Homelab - Learning Repository

![Kubernetes Logo](https://miro.medium.com/v2/resize:fit:730/1*WCsqMt85nMP0DvYv0JnkOA.png)

This repository contains my personal Kubernetes learning materials, configurations, and notes. As I learn Kubernetes, I'll be documenting concepts, configurations, and example deployments here.

## Status

- **LEARNING**: This repository is primarily for learning and documentation purposes
- **EXPERIMENTAL**: These configurations are for testing in a homelab environment
- Work in progress as I continue to learn and experiment

## Kubernetes Basics

### Core Concepts

1. **Pods**: The smallest deployable units in Kubernetes that can be created and managed. A pod encapsulates one or more containers, storage resources, a unique network IP, and options that govern how the container(s) should run.

2. **Nodes**: Worker machines in Kubernetes that run containerized applications. Each node is managed by the control plane and contains the services necessary to run pods.

3. **Control Plane**: The brain of a Kubernetes cluster that manages the worker nodes and the pods in the cluster. Components include:
   - API Server: Entry point for all REST commands
   - Scheduler: Assigns pods to nodes
   - Controller Manager: Runs controller processes
   - etcd: Consistent and highly-available key-value store

4. **Namespaces**: Virtual clusters within a physical cluster that provide a way to divide cluster resources between multiple users or teams.

### Workload Resources

1. **Deployments**: Provides declarative updates for Pods and ReplicaSets. Used for stateless applications.

2. **StatefulSets**: Used for applications that require persistent storage and stable network identifiers.

3. **DaemonSets**: Ensures all (or some) nodes run a copy of a pod. Useful for node monitoring or log collection.

4. **Jobs & CronJobs**: Run-to-completion tasks that execute once or on a schedule.

### Networking

1. **Services**: An abstract way to expose an application running on a set of Pods as a network service.
   - ClusterIP: Default service type, only accessible within the cluster
   - NodePort: Exposes the service on each Node's IP at a static port
   - LoadBalancer: Exposes the service externally using a cloud provider's load balancer
   - ExternalName: Maps the service to a DNS name

2. **Ingress**: API object that manages external access to services in a cluster, typically HTTP. Provides load balancing, SSL termination, and name-based virtual hosting.

3. **Network Policies**: Specifications of how groups of pods are allowed to communicate with each other and other network endpoints.

## Helm Charts

Helm is a package manager for Kubernetes that helps you define, install, and upgrade complex Kubernetes applications.

### Key Concepts

1. **Charts**: A Helm package that contains pre-configured Kubernetes resources. Think of it as a Kubernetes application package.

2. **Repositories**: Places where charts can be stored and shared. Like container registries for Docker images.

3. **Releases**: An instance of a chart running in a Kubernetes cluster. One chart can be installed multiple times into the same cluster.

### Basic Helm Commands

```bash
# Add a repository
helm repo add bitnami https://charts.bitnami.com/bitnami

# Search for charts
helm search repo nginx

# Install a chart
helm install my-release bitnami/nginx

# List releases
helm list

# Upgrade a release
helm upgrade my-release bitnami/nginx --set replicaCount=3

# Uninstall a release
helm uninstall my-release
```

### Creating Your Own Charts

```bash
# Create a new chart
helm create my-chart

# Package a chart
helm package my-chart

# Install a local chart
helm install my-release ./my-chart
```

A typical Helm chart structure:

```
my-chart/
  ├── Chart.yaml          # Metadata about the chart
  ├── values.yaml         # Default configuration values
  ├── templates/          # Kubernetes manifest templates
  │   ├── deployment.yaml
  │   ├── service.yaml
  │   └── _helpers.tpl    # Template helpers
  └── charts/             # Dependencies/subcharts
```

## Contents of This Repository

- Various deployment examples
- Service configurations
- Namespace definitions
- Documentation and learning notes

## Future Plans

- Document setup process for Kubernetes cluster
- Add more application deployments
- Implement proper monitoring with Prometheus and Grafana
- Explore GitOps with ArgoCD or Flux
- Add Helm chart examples and explanations
- Explore Kubernetes Operators

---

This repository is constantly evolving as I learn more about Kubernetes and related technologies.
