# Kubernetes Services: Complete Guide

## Overview of Kubernetes Services

A Kubernetes Service is an abstraction layer that defines a logical set of Pods and a policy by which to access them. Services enable network connectivity between different parts of an application, providing a stable network endpoint for a set of Pods.

## Service Types 

### 1. ClusterIP Service
- **Purpose**: Default service type for internal cluster communication
- **Characteristics**:
  - Assigns a virtual IP address within the cluster
  - Only accessible from within the cluster
  - Used for internal microservice communication
  - Provides a stable endpoint for Pod communication

**Manifest Example**:
```yaml
---
# Deployment Manifest
apiVersion: apps/v1
kind: Deployment
metadata:
  name: clusterip-deployment
  labels:
    app: clusterip-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: clusterip-app
  template:
    metadata:
      labels:
        app: clusterip-app
    spec:
      containers:
      - name: nginx-container
        image: nginx:latest
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 250m
            memory: 256Mi

---
# ClusterIP Service Manifest
apiVersion: v1
kind: Service
metadata:
  name: clusterip-service
spec:
  type: ClusterIP
  selector:
    app: clusterip-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

### 2. NodePort Service
- **Purpose**: Exposes the service on a static port on each node's IP
- **Characteristics**:
  - Creates a ClusterIP automatically
  - Exposes service on a port (default range 30000-32767)
  - Accessible from outside the cluster via `<NodeIP>:<NodePort>`
  - Useful for development and testing environments

**Manifest Example**:
```yaml
---
# Deployment Manifest
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nodeport-deployment
  labels:
    app: nodeport-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nodeport-app
  template:
    metadata:
      labels:
        app: nodeport-app
    spec:
      containers:
      - name: web-app
        image: nginx:latest
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 250m
            memory: 256Mi

---
# NodePort Service Manifest
apiVersion: v1
kind: Service
metadata:
  name: nodeport-service
spec:
  type: NodePort
  selector:
    app: nodeport-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30080
```

### 3. LoadBalancer Service
- **Purpose**: Integrates with cloud provider's load balancer
- **Characteristics**:
  - Creates NodePort and ClusterIP automatically
  - Provisions an external load balancer
  - Ideal for exposing services to external traffic
  - Primarily used in cloud environments

**Manifest Example**:
```yaml
---
# Deployment Manifest
apiVersion: apps/v1
kind: Deployment
metadata:
  name: loadbalancer-deployment
  labels:
    app: loadbalancer-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: loadbalancer-app
  template:
    metadata:
      labels:
        app: loadbalancer-app
    spec:
      containers:
      - name: web-app
        image: nginx:latest
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 250m
            memory: 256Mi

---
# LoadBalancer Service Manifest
apiVersion: v1
kind: Service
metadata:
  name: loadbalancer-service
spec:
  type: LoadBalancer
  selector:
    app: loadbalancer-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

### 4. ExternalIP Service
- **Purpose**: Routes external traffic to cluster services
- **Characteristics**:
  - Uses a specific external IP to route traffic
  - Requires manual IP configuration
  - Limited control compared to LoadBalancer
  - Provides direct access to service from outside the cluster

**Manifest Example**:
```yaml
---
# Deployment Manifest
apiVersion: apps/v1
kind: Deployment
metadata:
  name: externalip-deployment
  labels:
    app: externalip-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: externalip-app
  template:
    metadata:
      labels:
        app: externalip-app
    spec:
      containers:
      - name: web-app
        image: nginx:latest
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 250m
            memory: 256Mi

---
# ExternalIP Service Manifest
apiVersion: v1
kind: Service
metadata:
  name: externalip-service
spec:
  type: ClusterIP
  selector:
    app: externalip-app
  externalIPs:
    - 192.168.1.100  # Replace with your actual external IP
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

### 5. ExternalName Service
- **Purpose**: Maps service to an external DNS name
- **Characteristics**:
  - Returns a CNAME record
  - Useful for referencing external services
  - No proxying of traffic
  - Provides service abstraction for external dependencies

**Manifest Example**:
```yaml
---
# Deployment Manifest
apiVersion: apps/v1
kind: Deployment
metadata:
  name: external-service-deployment
  labels:
    app: external-service-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: external-service-app
  template:
    metadata:
      labels:
        app: external-service-app
    spec:
      containers:
      - name: external-app
        image: nginx:latest
        ports:
        - containerPort: 80
        env:
        - name: EXTERNAL_SERVICE_HOST
          value: external.example.com

---
# ExternalName Service Manifest
apiVersion: v1
kind: Service
metadata:
  name: external-service
spec:
  type: ExternalName
  externalName: myservice.example.com
  ports:
    - port: 80
```

### 6. Headless Service
- **Purpose**: No load balancing or cluster IP
- **Characteristics**:
  - Returns individual Pod IPs
  - Used for stateful applications
  - Enables direct Pod-to-Pod communication
  - Typically used with StatefulSets

**Manifest Example**:
```yaml
---
# StatefulSet Manifest
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: headless-statefulset
spec:
  serviceName: "headless-service"
  replicas: 3
  selector:
    matchLabels:
      app: headless-app
  template:
    metadata:
      labels:
        app: headless-app
    spec:
      containers:
      - name: web-app
        image: nginx:latest
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 250m
            memory: 256Mi

---
# Headless Service Manifest
apiVersion: v1
kind: Service
metadata:
  name: headless-service
spec:
  clusterIP: None
  selector:
    app: headless-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

## DNS Naming Convention

Kubernetes Services are assigned DNS names in the following format:
- Within the same namespace: `<service-name>`
- Across namespaces: `<service-name>.<namespace-name>.svc.cluster.local`

## Key Considerations
- Choose service type based on your specific network requirements
- Consider security, accessibility, and performance
- Use appropriate selectors to match Pods
- Configure ports carefully
- Implement resource requests and limits

## Best Practices
1. Use ClusterIP for internal communication
2. Leverage NodePort for development and testing
3. Use LoadBalancer for production external access
4. Implement network policies for security
5. Monitor service performance and connectivity
6. Use consistent labeling across deployments and services
7. Implement health checks and readiness probes
8. Use namespace for better organization

## Advanced Configurations

### Network Policy Example
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: service-network-policy
spec:
  podSelector:
    matchLabels:
      app: my-app
  ingress:
    - from:
      - podSelector:
          matchLabels:
            allow-access: "true"
      ports:
        - protocol: TCP
          port: 80
```

## Troubleshooting Tips
- Verify service endpoint connectivity
- Check pod status and readiness
- Validate network policies
- Ensure correct service selectors
- Monitor cluster network events
