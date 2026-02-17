# Kubernetes Architecture Documentation

## 1. Executive Summary

PropIntel uses Kubernetes (K8s) for production-grade orchestration, providing capabilities like self-healing (auto-restart of crashed pods), load balancing (via Services), and horizontal scaling. The deployment is defined in two key manifests: `k8s-deployment.yaml` for managing the application pods and `k8s-service.yaml` for exposing the network interface.

---

## 2. Resource Objects

### 2.1 Deployment (`k8s-deployment.yaml`)

The `Deployment` object ensures that a specified number of pod replicas are running at any given time.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: propintel-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: propintel-app
```

#### Pod Specification
Defines the container runtime details.
- **Image**: `docker.io/library/propintel:latest` (PullPolicy: `Never` for local Minikube contexts).
- **Resources**:
  - **Requests**: `memory: "512Mi"`, `cpu: "250m"` (Minimum needed to start).
  - **Limits**: `memory: "1Gi"`, `cpu: "500m"` (Cap to prevent node starvation).

#### Health Checks (Probes)
Critical for self-healing.
- **Liveness Probe**: "Are you alive?"
  - Checks `/_stcore/health` endpoint.
  - Failure = Restart Container.
- **Readiness Probe**: "Can you accept traffic?"
  - Checks if app is fully loaded.
  - Failure = Remove from Service Endpoints (stop sending traffic).

### 2.2 Service (`k8s-service.yaml`)

The `Service` object provides a stable network endpoint (IP/Port) for the dynamic set of pods managed by the Deployment.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: propintel-service
spec:
  type: NodePort
  ports:
    - port: 8501
      targetPort: 8501
      nodePort: 30001
```

- **Type: NodePort**: Exposes the service on each Node's IP at a static port (`30001`). Suitable for development or on-prem clusters. In cloud (AWS/GCP), `LoadBalancer` would be used.
- **Traffic Flow**: Request -> Node:30001 -> Kube-Proxy -> Pod:8501.

---

## 3. Storage Strategy

### 3.1 Persistent Volumes (PVCs)
K8s pods are ephemeral; if they restart, file system changes are lost.

**Current Implementation**:
- The provided manifests utilize `hostPath` (implied in local testing) or `emptyDir`.
- **Recommendation for Production**:
  ```yaml
  volumes:
  - name: data-storage
    persistentVolumeClaim:
      claimName: propintel-pvc
  ```

---

## 4. Network Topology in Cluster

```mermaid
graph TD
    User[User @ Browser] -->|http://minikube-ip:30001| NodePort
    
    subgraph K8s Cluster
        NodePort --> Service[PropIntel Service]
        Service -->|Load Balance| Pod1[Pod: PropIntel Apps]
        Service -->|Load Balance| Pod2[Pod: PropIntel Apps (Replica)]
        
        Pod1 -->|Env: OLLAMA_HOST| External[External Ollama Service]
    end
```

---

## 5. Scaling Strategy

### 5.1 Manual Scaling
Administrators can react to load by manually increasing replicas.
```bash
kubectl scale deployment propintel-app --replicas=3
```

### 5.2 Horizontal Pod Autoscaler (HPA)
Future implementation to automate scaling based on CPU usage.
```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
spec:
  scaleTargetRef:
    kind: Deployment
    name: propintel-app
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 50
```

---

## 6. Implementation Guide

### 6.1 Prerequisites
- **Minikube** or valid Kubeconfig.
- **Kubectl** CLI tool.

### 6.2 Deployment Steps
1. **Load Image**: `minikube image load propintel:latest`
2. **Apply Manifests**:
   ```bash
   kubectl apply -f k8s-deployment.yaml
   kubectl apply -f k8s-service.yaml
   ```
3. **Access**:
   ```bash
   minikube service propintel-service
   ```

---

## 7. Troubleshooting

| Symptom | Probable Cause | Fix |
|---------|----------------|-----|
| **ImagePullBackOff** | Image not found in registry | Use `imagePullPolicy: Never` for local images or push to Docker Hub. |
| **CrashLoopBackOff** | Application error / OOM | Check logs: `kubectl logs -f <pod-name>`. Increase memory limits. |
| **Connection Refused** | Service not exposed | Verify `kubectl get svc` shows port mapping. |

---

## 8. Development vs Production

| Feature | Development (Minikube) | Production (Cloud) |
|---------|------------------------|-------------------|
| **Service Type** | NodePort | LoadBalancer / Ingress |
| **Storage** | HostPath | PVC (EBS/GPD) |
| **Replicas** | 1 | >2 (Multi-AZ) |
| **Database** | CSV in Pod | Checkpoint to S3 / SQL DB |
