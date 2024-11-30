# Kubernetes Resources Deep Dive

## 1. Pods

### Theoretical Overview
- Smallest deployable unit in Kubernetes
- Represents a single instance of a running process
- Can contain one or multiple containers
- Shares network namespace and storage volumes

### Lab: Creating a Simple Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-pod
spec:
  containers:
  - name: nginx-container
    image: nginx:latest
    ports:
    - containerPort: 80
```

### Commands for Pod Management:
```bash
# Create a pod
kubectl apply -f pod.yaml

# List pods
kubectl get pods

# Describe pod details
kubectl describe pod simple-pod

# Delete a pod
kubectl delete pod simple-pod
```

## 2. Multi-Container Pods

### Types of Multi-Container Pod Patterns
1. **Sidecar Pattern**
   - Auxiliary container supports main application
   - Example: Logging, monitoring sidecars

2. **Ambassador Pattern**
   - Proxy container handles external communications
   - Abstracts network complexity

3. **Adapter Pattern**
   - Transforms main container's output
   - Standardizes monitoring or logging

### Lab: Multi-Container Pod Example
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
  - name: main-app
    image: myapp:v1
    ports:
    - containerPort: 8080
  
  - name: logging-sidecar
    image: fluent/fluent-bit
    volumeMounts:
    - name: log-volume
      mountPath: /var/log
```

## 3. ReplicaSet

### Key Characteristics
- Ensures specified number of pod replicas are running
- Automatically replaces failed pods
- Provides high availability and scalability

### Lab: ReplicaSet Configuration
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
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

## 4. Deployments

### Features
- Declarative updates for Pods and ReplicaSets
- Rolling update strategies
- Rollback capabilities
- Self-healing mechanism

### Lab: Deployment with Rolling Update
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
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

## 5. DaemonSet

### Use Cases
- Runs a pod on every node in cluster
- Ideal for node-level logging
- Monitoring agents
- Storage daemons

### Lab: DaemonSet Example
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
spec:
  selector:
    matchLabels:
      app: node-monitoring
  template:
    metadata:
      labels:
        app: node-monitoring
    spec:
      containers:
      - name: node-exporter
        image: prom/node-exporter
```

## 6. Jobs & CronJobs

### Jobs
- Runs tasks to completion
- Ensures specified number of successful completions

### CronJobs
- Schedule recurring jobs
- Similar to crontab in Unix/Linux

### Lab: Job and CronJob Examples
```yaml
# Job Example
apiVersion: batch/v1
kind: Job
metadata:
  name: backup-job
spec:
  template:
    spec:
      containers:
      - name: backup
        image: backup-tool:latest
      restartPolicy: OnFailure

# CronJob Example
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: daily-backup
spec:
  schedule: "0 2 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: backup-tool:latest
          restartPolicy: OnFailure
```

## 7. StatefulSets

### Characteristics
- Stable, unique network identifiers
- Stable, persistent storage
- Ordered, graceful deployment and scaling
- Ordered, automated rolling updates

### Lab: StatefulSet Configuration
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql-statefulset
spec:
  serviceName: "mysql"
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
```

## InitContainers

### Purpose
- Run before main application containers
- Perform setup tasks
- Block and delay main container startup until initialization completes

### Lab: InitContainer Example
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-demo
spec:
  initContainers:
  - name: init-service
    image: busybox
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for service; sleep 2; done;']
  
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo Application is running!']
```

## Best Practices
- Use labels and selectors effectively
- Implement health checks
- Use resource limits
- Consider network policies
- Implement proper logging and monitoring
