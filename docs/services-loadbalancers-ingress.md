# Kubernetes Networking: Services, Load Balancers, and Ingress

## Services in Kubernetes

Services are the primary way to expose applications running in Kubernetes. They provide:

- A stable IP address and DNS name for pods
- Load balancing between multiple pods 
- Service discovery so pods can be created, destroyed and scaled without affecting connectivity

## Service Types

Kubernetes offers several types of services to meet different networking requirements:

### ClusterIP (Default)

- Exposes the service on an internal IP within the cluster
- Only accessible from within the cluster
- Perfect for internal communication between services

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-internal-service
spec:
  type: ClusterIP      # This is default and can be omitted
  ports:
  - port: 80           # Port exposed to the cluster
    targetPort: 8080   # Port your app listens on in the pod
  selector:
    app: my-app        # Selects pods with label app=my-app
```

### NodePort

- Exposes the service on each node's IP at a static port
- Accessible from outside the cluster using `<NodeIP>:<NodePort>`
- Port range is typically limited to 30000-32767

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nodeport-service
spec:
  type: NodePort
  ports:
  - port: 80           # Port exposed within the cluster
    targetPort: 8080   # Port your app listens on in the pod
    nodePort: 30007    # Port exposed on the node (optional, auto-assigned if omitted)
  selector:
    app: my-app
```

### LoadBalancer

- Extends NodePort by creating an external load balancer
- Supported by cloud providers (AWS ELB, GCP Load Balancer, etc.)
- Assigns a fixed, external IP address that routes to your service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-loadbalancer
spec:
  type: LoadBalancer
  ports:
  - port: 80           # Port exposed externally
    targetPort: 8080   # Port your app listens on in the pod
  selector:
    app: my-app        # Selects pods with label app=my-app
```

When this is deployed:
1. Kubernetes asks the cloud provider for a load balancer
2. Cloud provider provisions a load balancer and assigns an external IP
3. Traffic to that IP:80 is routed to port 8080 on your pods

### ExternalName

- Maps a service to a DNS name without selectors
- Useful for integrating with services outside your cluster
- Works by returning a CNAME record

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-database
spec:
  type: ExternalName
  externalName: database.example.com  # External service DNS
```

## Ingress

Ingress is not a service type, but a separate resource that sits in front of multiple services. It provides:

- HTTP/HTTPS routing
- TLS termination
- Name-based virtual hosting
- Path-based routing
- Load balancing

### Ingress Components

1. **Ingress Resource**: The Kubernetes API object defining routing rules
2. **Ingress Controller**: The actual implementation (NGINX, Traefik, HAProxy, etc.)

### Ingress Example

First, you need an Ingress Controller (typically deployed separately), then create an Ingress resource:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: myapp.example.com       # Host-based routing
    http:
      paths:
      - path: /api                # Path-based routing
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
      - path: /admin
        pathType: Prefix
        backend:
          service:
            name: admin-service
            port:
              number: 80
  # Optional TLS configuration
  tls:
  - hosts:
    - myapp.example.com
    secretName: myapp-tls-cert    # Secret containing TLS certificate
```

## Real-World Networking Examples

### Microservices Architecture

```yaml
# Frontend service (LoadBalancer for external access)
apiVersion: v1
kind: Service
metadata:
  name: storefront
spec:
  type: LoadBalancer
  ports:
  - port: 443
    targetPort: 8080
  selector:
    app: frontend
---
# API Gateway Ingress (multiple backend services)
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: api-gateway
spec:
  rules:
  - host: api.mystore.com
    http:
      paths:
      - path: /products
        pathType: Prefix
        backend:
          service:
            name: product-service
            port: 
              number: 80
      - path: /cart
        pathType: Prefix
        backend:
          service:
            name: cart-service
            port:
              number: 80
      - path: /orders
        pathType: Prefix
        backend:
          service:
            name: order-service
            port:
              number: 80
```

### Internal Service Mesh

```yaml
# Internal services using ClusterIP
apiVersion: v1
kind: Service
metadata:
  name: auth-service
spec:
  type: ClusterIP
  ports:
  - port: 9000
    targetPort: 9000
  selector:
    app: auth
---
apiVersion: v1
kind: Service
metadata:
  name: payment-processor
spec:
  type: ClusterIP
  ports:
  - port: 8500
    targetPort: 8500
  selector:
    app: payments
```

## Network Policies

Network Policies are like firewalls for pod communication. They specify how pods are allowed to communicate with each other and other network endpoints.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-allow
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          tier: frontend
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - podSelector:
        matchLabels:
          tier: database
    ports:
    - protocol: TCP
      port: 5432
```

This policy:
- Applies to pods labeled `app: api`
- Allows ingress from pods labeled `tier: frontend` on port 8080
- Allows egress to pods labeled `tier: database` on port 5432
- Blocks all other traffic

## Best Practices

1. **Use Meaningful Labels**: Services select pods using labels, so use a consistent labeling strategy.

2. **Choose the Right Service Type**: Use ClusterIP for internal services, LoadBalancer for external services, and Ingress for HTTP/HTTPS routing.

3. **Implement Network Policies**: Don't leave your pod network wide open - use network policies to restrict traffic.

4. **Use HTTPS**: Configure TLS in your Ingress resources for secure communication.

5. **Consider Service Mesh**: For complex applications, a service mesh like Istio or Linkerd provides advanced traffic management, security, and observability features.

6. **Monitor Service Health**: Implement readiness and liveness probes to ensure your services are healthy.

7. **Use External DNS**: For production environments, integrate with External DNS to automatically manage DNS records for your services.
