# Configuring Ingress Controller for Application Access in Kubernetes

[Previous sections remain the same until Step 1]

### Step 1: Deploy the Ingress Controller
Deploy the NGINX ingress controller:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```

Output:
```
namespace/ingress-nginx created
serviceaccount/ingress-nginx created
serviceaccount/ingress-nginx-admission created
role.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
configmap/ingress-nginx-controller created
service/ingress-nginx-controller created
service/ingress-nginx-controller-admission created
deployment.apps/ingress-nginx-controller created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created
ingressclass.networking.k8s.io/nginx created
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created
```

### Step 2: Check Pods in Ingress-NGINX Namespace
```bash
kubectl get pods -n ingress-nginx
```

Output:
```
NAME                                       READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-dhdll       0/1     Completed   0          22s
ingress-nginx-admission-patch-j52j2        0/1     Completed   0          22s
ingress-nginx-controller-974f4cbd8-4wh92   0/1     Running     0          22s
```

### Step 3: Create and Scale the httpd Deployment
```bash
# Create deployment
kubectl create deployment httpd --image=httpd:2.4
```
Output:
```
deployment.apps/httpd created
```

```bash
# Scale the deployment to 2 replicas
kubectl scale deployment httpd --replicas=2
```
Output:
```
deployment.apps/httpd scaled
```

Verify pods:
```bash
kubectl get pods
```
Output:
```
NAME                     READY   STATUS    RESTARTS   AGE
httpd-7fc696b987-fc4rx   1/1     Running   0          14s
httpd-7fc696b987-jpbtv   1/1     Running   0          14s
```

### Step 4: Exec into httpd Pods to Edit index.html
For first pod:
```bash
kubectl exec -it httpd-7fc696b987-fc4rx -- bash
echo "Welcome to Version 1" > htdocs/index.html
exit
```

For second pod:
```bash
kubectl exec -it httpd-7fc696b987-jpbtv -- bash
echo "Welcome to Version 2" > htdocs/index.html
exit
```

### Step 5: Expose the httpd Deployment
```bash
kubectl expose deployment httpd --port=80
```
Output:
```
service/httpd exposed
```

Verify the service:
```bash
kubectl get svc
```
Output:
```
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
httpd        ClusterIP   10.110.187.115   <none>        80/TCP    28s
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP   20h
```

### Step 6: Check Services in Ingress-NGINX Namespace
```bash
kubectl get svc -n ingress-nginx
```
Output:
```
NAME                                 TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.98.34.127   <pending>     80:30224/TCP,443:30209/TCP   5m49s
ingress-nginx-controller-admission   ClusterIP      10.96.70.193   <none>        443/TCP                      5m49s
```

### Step 7: Test the Setup with Curl (Round Robin)
```bash
curl localhost:30224
```
Multiple test outputs showing round-robin load balancing:
```
Welcome to Version 1
```
```
Welcome to Version 2
```
```
Welcome to Version 1
```
```
Welcome to Version 2
```

### Converting LoadBalancer to NodePort

After editing the service:
```bash
kubectl get svc -n ingress-nginx
```
Output:
```
NAME                                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort    10.98.34.127   <none>        80:30224/TCP,443:30209/TCP   8m19s
ingress-nginx-controller-admission   ClusterIP   10.96.70.193   <none>        443/TCP                      8m19s
```

### Additional Information
Check pod details with wide output:
```bash
kubectl get pod -o wide
```
Output:
```
NAME                     READY   STATUS    RESTARTS   AGE   IP            NODE           NOMINATED NODE   READINESS GATES
httpd-7fc696b987-fc4rx   1/1     Running   0          12m   192.168.1.7   node01         <none>           <none>
httpd-7fc696b987-jpbtv   1/1     Running   0          12m   192.168.0.4   controlplane   <none>           <none>
```

Check ingress-nginx pods with wide output:
```bash
kubectl get pod -n ingress-nginx -o wide
```
Output:
```
NAME                                       READY   STATUS      RESTARTS   AGE   IP            NODE     NOMINATED NODE   READINESS GATES
ingress-nginx-admission-create-dhdll       0/1     Completed   0          15m   192.168.1.4   node01   <none>           <none>
ingress-nginx-admission-patch-j52j2        0/1     Completed   0          15m   192.168.1.5   node01   <none>           <none>
ingress-nginx-controller-974f4cbd8-4wh92   1/1     Running     0          15m   192.168.1.6   node01   <none>           <none>
```
![image](https://github.com/user-attachments/assets/46f89465-b7b9-481d-acd1-d3f28ae9f95b)
![image](https://github.com/user-attachments/assets/ca195c14-23ad-45fa-84e8-755968598e75)


