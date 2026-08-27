# QuickNook Kubernetes Deployment

This repository contains the Kubernetes manifests and deployment commands used to deploy the **QuickNook application** on Kubernetes.

## Application Architecture

QuickNook consists of:

- Frontend: React + Nginx
- Backend: Spring Boot
- Database: MySQL
- Container Runtime: Podman
- Container Registry: Docker Hub
- Load Balancer: MetalLB
- Kubernetes Networking: Service-based communication
- MySQL Storage: PersistentVolume / PersistentVolumeClaim

### Application Flow

```text
                    Client
                      |
                      v
              MetalLB LoadBalancer
                      |
                      v
              QuickNook Frontend
                React + Nginx
                      |
                      v
             QuickNook Backend
                Spring Boot
                      |
                      v
              QuickNook MySQL
```

---

# 1. Podman Commands

## Build Backend Image

```bash
podman build -t quicknook-backend:1.1 .
```

## Tag Image for Docker Hub

```bash
podman tag localhost/quicknook-backend:1.1 \
docker.io/ramakrishnapanda2001/quicknook-backend:1.1
```

## Verify Image

```bash
podman images | grep quicknook-backend
```

## Push Image to Docker Hub

```bash
podman push docker.io/ramakrishnapanda2001/quicknook-backend:1.1
```

---

# 2. MetalLB Load Balancer

MetalLB is used to provide LoadBalancer IP addresses to Kubernetes services in the bare-metal/on-premises Kubernetes environment.

## Check MetalLB

```bash
kubectl get pods -n metallb-system
```

Check Kubernetes version:

```bash
kubectl version
```

## Install MetalLB

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.15.3/config/manifests/metallb-native.yaml
```

Check MetalLB pods:

```bash
kubectl get pods -n metallb-system
```

Watch the pods:

```bash
kubectl get pods -n metallb-system -w
```

## Check Node Network

```bash
ip -br addr
```

```bash
ip route
```

---

# 3. MetalLB IP Address Pool

Create a file named:

```text
metallb-config.yaml
```

Configuration:

```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: quicknook-pool
  namespace: metallb-system
spec:
  addresses:
    - 172.26.7.240-172.26.7.250

---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: quicknook-l2
  namespace: metallb-system
spec:
  ipAddressPools:
    - quicknook-pool
```

Apply the configuration:

```bash
kubectl apply -f metallb-config.yaml
```

Verify the IP address pool:

```bash
kubectl get ipaddresspool -n metallb-system
```

Verify the L2 advertisement:

```bash
kubectl get l2advertisement -n metallb-system
```

---

# 4. QuickNook Kubernetes Namespace

Check the QuickNook namespace:

```bash
kubectl get namespace
```

Check all resources inside the namespace:

```bash
kubectl get all -n quicknook
```

---

# 5. Deploy QuickNook Application

Apply the Kubernetes YAML files:

```bash
kubectl apply -f .
```

> Make sure sensitive files such as Kubernetes Secrets, passwords, private keys, and kubeconfig files are not committed to the public repository.

If the manifests are separated into directories, they can also be applied individually:

```bash
kubectl apply -f mysql/
kubectl apply -f backend/
kubectl apply -f frontend/
```

---

# 6. Check Kubernetes Resources

## Pods

```bash
kubectl get pods -n quicknook
```

For detailed pod information:

```bash
kubectl get pods -n quicknook -o wide
```

## Deployments

```bash
kubectl get deployment -n quicknook
```

## Services

```bash
kubectl get svc -n quicknook
```

## StatefulSets

```bash
kubectl get statefulset -n quicknook
```

## PersistentVolumeClaims

```bash
kubectl get pvc -n quicknook
```

## PersistentVolumes

```bash
kubectl get pv
```

## Kubernetes Secrets

```bash
kubectl get secret -n quicknook
```

---

# 7. Check Application Services

```bash
kubectl get svc -n quicknook
```

Example:

```text
NAME                 TYPE           CLUSTER-IP      EXTERNAL-IP
quicknook-backend    ClusterIP      10.x.x.x        <none>
quicknook-frontend   LoadBalancer   10.x.x.x        x.x.x.x
quicknook-mysql      ClusterIP      None            <none>
```

The frontend is exposed externally using the MetalLB LoadBalancer IP.

Access the application using:

```text
http://x.x.x.x:80
```

> The actual IP may be different depending on the IP assigned by MetalLB.

---

# 8. Check Backend Logs

```bash
kubectl logs -n quicknook deployment/quicknook-backend
```

Follow backend logs:

```bash
kubectl logs -f -n quicknook deployment/quicknook-backend
```

---

# 9. Check Frontend Logs

```bash
kubectl logs -n quicknook deployment/quicknook-frontend
```

---

# 10. Check MySQL Logs

```bash
kubectl logs -n quicknook quicknook-mysql-0
```

---

# 11. Access MySQL

Connect directly to the MySQL pod:

```bash
kubectl exec -it -n quicknook quicknook-mysql-0 -- mysql -u root -p
```

Select the QuickNook database:

```sql
USE quicknook;
```

Check tables:

```sql
SHOW TABLES;
```

Check users:

```sql
SELECT * FROM users;
```

Check technicians:

```sql
SELECT * FROM technician;
```

---

# 12. Useful Kubernetes Commands

## Describe a Pod

```bash
kubectl describe pod <pod-name> -n quicknook
```

## Execute Shell Inside a Pod

```bash
kubectl exec -it <pod-name> -n quicknook -- /bin/bash
```

## Check Deployment Status

```bash
kubectl rollout status deployment/quicknook-backend -n quicknook
```

```bash
kubectl rollout status deployment/quicknook-frontend -n quicknook
```

## Restart Backend

```bash
kubectl rollout restart deployment/quicknook-backend -n quicknook
```

## Restart Frontend

```bash
kubectl rollout restart deployment/quicknook-frontend -n quicknook
```

## View All QuickNook Resources

```bash
kubectl get all -n quicknook
```

---

# 13. Troubleshooting

### Check Pod Status

```bash
kubectl get pods -n quicknook -o wide
```

### Check Pod Events

```bash
kubectl describe pod <pod-name> -n quicknook
```

### Check Backend Logs

```bash
kubectl logs -f -n quicknook deployment/quicknook-backend
```

### Check Services

```bash
kubectl get svc -n quicknook
```

### Check Persistent Storage

```bash
kubectl get pv
kubectl get pvc -n quicknook
```

### Check MetalLB

```bash
kubectl get pods -n metallb-system
kubectl get ipaddresspool -n metallb-system
kubectl get l2advertisement -n metallb-system
```

---

# 14. Technology Stack

| Component | Technology |
|---|---|
| Frontend | React |
| Web Server | Nginx |
| Backend | Spring Boot |
| Database | MySQL 8 |
| Container Runtime | Podman |
| Container Registry | Docker Hub |
| Orchestration | Kubernetes |
| Load Balancer | MetalLB |
| Load Balancing Mode | Layer 2 |
| Storage | Kubernetes PV/PVC |

---

# 15. Repository Structure

```text
quicknook-k8s/
│
├── README.md
├── metallb-config.yaml
│
├── namespace.yaml
│
├── mysql/
│   ├── mysql-service.yaml
│   └── mysql-statefulset.yaml
│
├── backend/
│   ├── backend-deployment.yaml
│   └── backend-service.yaml
│
└── frontend/
    ├── frontend-deployment.yaml
    └── frontend-service.yaml
```

---

# 16. Project

**QuickNook – Full Stack Application on Kubernetes**

This project demonstrates containerized application deployment using Kubernetes with:

- React frontend
- Nginx
- Spring Boot backend
- MySQL StatefulSet
- Kubernetes Services
- Persistent Storage
- MetalLB LoadBalancer
- Layer 2 networking
- Podman containerization
- Docker Hub image registry
