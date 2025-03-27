# Kubernetes Deployments

Deployments are one of the most common Kubernetes resources, providing declarative updates for Pods and ReplicaSets. They're the recommended way to manage stateless applications in Kubernetes.

## Key Features of Deployments

- **Declarative Updates**: Define your desired state, and Kubernetes works to achieve and maintain it
- **Rolling Updates**: Update applications with zero downtime
- **Rollback**: Easy rollback to previous versions if something goes wrong
- **Scaling**: Scale applications up or down easily
- **Pause/Resume**: Pause deployments to fix issues, then resume them

## Basic Deployment Structure

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  replicas: 3                 # Number of pods to run
  selector:
    matchLabels:
      app: my-app             # Label selector for pods
  template:                   # Pod template
    metadata:
      labels:
        app: my-app           # Label applied to pods
    spec:
      containers:
      - name: my-app
        image: nginx:1.21     # Container image
        ports:
        - containerPort: 80   # Port the container exposes
        resources:
          limits:
            cpu: "500m"       # 500 millicpu (0.5 CPU core)
            memory: "256Mi"   # 256 MB memory
          requests:
            cpu: "100m"       # Request 100 millicpu (0.1 CPU core)
            memory: "128Mi"   # Request 128 MB memory
```

## Creating and Managing Deployments

### Create a Deployment

```bash
# From a YAML file
kubectl apply -f deployment.yaml

# From the command line
kubectl create deployment my-app --image=nginx:1.21 --replicas=3
```

### View Deployments

```bash
# List all deployments
kubectl get deployments

# Get details of a specific deployment
kubectl describe deployment my-app
```

### Update a Deployment

```bash
# Update the image
kubectl set image deployment/my-app my-app=nginx:1.22

# Scale the deployment
kubectl scale deployment/my-app --replicas=5

# Edit the deployment directly
kubectl edit deployment/my-app
```

### Rolling Updates

By default, Deployments use a rolling update strategy:

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%     # Maximum number of pods that can be unavailable during update
      maxSurge: 25%           # Maximum number of pods that can be created over the desired number
```

This ensures zero downtime during updates by gradually replacing old pods with new ones.

### Rollback a Deployment

```bash
# View rollout history
kubectl rollout history deployment/my-app

# Rollback to previous version
kubectl rollout undo deployment/my-app

# Rollback to a specific revision
kubectl rollout undo deployment/my-app --to-revision=2
```

### Pause and Resume Deployments

```bash
# Pause a deployment
kubectl rollout pause deployment/my-app

# Make changes during pause
kubectl set resources deployment/my-app -c=my-app --limits=cpu=200m,memory=512Mi

# Resume the deployment to apply changes
kubectl rollout resume deployment/my-app
```

## Deployment Strategies

### RollingUpdate (Default)

- Gradually replaces old pods with new ones
- Ensures application availability during updates
- Controls update rate with `maxUnavailable` and `maxSurge`

### Recreate

```yaml
spec:
  strategy:
    type: Recreate
```

- Terminates all existing pods before creating new ones
- Results in downtime, but ensures no two versions run simultaneously
- Useful when versions are incompatible or for stateful applications that can't run multiple versions

### Blue/Green Deployment (using labels and services)

1. Deploy new version with different labels
2. Test the new version
3. Switch service selector to point to new version
4. Remove old version when ready

### Canary Deployment (using multiple deployments)

1. Deploy a small number of new version pods alongside existing ones
2. Route a percentage of traffic to the new version
3. Monitor for issues
4. Gradually increase new version pods if stable

## Advanced Deployment Features

### Health Checks

```yaml
spec:
  template:
    spec:
      containers:
      - name: my-app
        livenessProbe:          # Restarts container if it fails
          httpGet:
            path: /healthz
            port: 8080
          initialDelaySeconds: 3
          periodSeconds: 3
        readinessProbe:         # Removes pod from service if it fails
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
```

### Init Containers

```yaml
spec:
  template:
    spec:
      initContainers:
      - name: init-myservice
        image: busybox:1.34
        command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
      containers:
      - name: my-app
        image: nginx:1.21
```

### Environment Variables

```yaml
spec:
  template:
    spec:
      containers:
      - name: my-app
        image: my-app:1.0
        env:
        - name: DATABASE_URL
          value: "postgresql://user:password@postgres:5432/db"
        - name: NODE_ENV
          value: "production"
```

### Using ConfigMaps and Secrets

```yaml
spec:
  template:
    spec:
      containers:
      - name: my-app
        image: my-app:1.0
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: database-url
        - name: APP_CONFIG
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: config.json
        volumeMounts:
        - name: config-volume
          mountPath: /etc/config
      volumes:
      - name: config-volume
        configMap:
          name: app-config
```

## Best Practices

1. **Use Resource Limits**: Always set resource requests and limits to ensure fair resource allocation.

2. **Implement Health Checks**: Use readiness and liveness probes to maintain application health.

3. **Set Update Strategy**: Configure `maxUnavailable` and `maxSurge` based on your application's needs.

4. **Label Carefully**: Use meaningful labels that make it easy to identify application components.

5. **Use Namespaces**: Organize deployments into namespaces for better resource management.

6. **Keep Deployments Stateless**: Deployments are designed for stateless applications. Use StatefulSets for stateful applications.

7. **Version Control**: Store deployment YAML files in version control.

8. **Use Helm Charts**: For complex applications, consider using Helm charts to manage deployments.

9. **Monitor Deployments**: Implement monitoring and alerting for deployment status.

10. **Document Deployment Procedures**: Document deployment, scaling, and rollback procedures for your team.
