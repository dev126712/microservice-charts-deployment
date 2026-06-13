# Microservice Charts Deployment

Helm charts and ArgoCD Application manifests for the [GitOps microservices platform on GKE](https://github.com/dev126712/microservice-end-to-end). This is the **GitOps repo** — ArgoCD watches it and reconciles the cluster state automatically.

[![My Skills](https://skillicons.dev/icons?i=kubernetes,helm,docker)](https://skillicons.dev)

![Architecture](https://github.com/dev126712/microservice-charts-deployment/blob/24837da6dcd313547c8ccd9de65bc7c0b78ae00a/Untitled%20Diagram.drawio%20(8).png)

---

## Repository Role in the Three-Repo GitOps Pattern

```
microservices-app          ← Application code + CI (builds images, pushes to registry)
     │ (image tag update)
     ▼
microservice-charts-deployment  ← THIS REPO: Helm values + ArgoCD manifests
     │ (ArgoCD watches)
     ▼
GKE Cluster                ← micro-service-infra-management provisions the cluster
```

---

## Services

| Service | Kind | Key Kubernetes Resources |
|---|---|---|
| **Frontend** | Deployment | Deployment · Service · HPA · ConfigMap · HTTPRoute · NetworkPolicy |
| **Product** | Deployment | Deployment · Service · HPA · ConfigMap · HTTPRoute · NetworkPolicy |
| **Order** | Deployment | Deployment · Service · HPA · ConfigMap · HTTPRoute · NetworkPolicy |
| **Notification** | Deployment | Deployment · Service · HPA · ConfigMap · HTTPRoute · NetworkPolicy |
| **PostgreSQL** | StatefulSet | StatefulSet · Service · ConfigMap · NetworkPolicy |
| **Redis** | StatefulSet | StatefulSet · Service · ConfigMap · NetworkPolicy |

### Network Security
Every service has a `NetworkPolicy` — traffic is restricted to explicitly allowed service-to-service paths only. Default ingress/egress is denied.

### Traffic Routing
`HTTPRoute` resources (GKE Gateway API) define path-based routing rules per service. This replaces traditional Ingress for fine-grained traffic control.

### Autoscaling
HPA is configured for all stateless services. Scale triggers are defined per service based on CPU/memory thresholds.

---

## Deploying with ArgoCD

Apply the ArgoCD Application manifests to the cluster:

```bash
# The manifests in application-manifest/ point ArgoCD at this repo
kubectl apply -f application-manifest/

# ArgoCD will detect the Helm values and sync the cluster state
```

ArgoCD continuously watches this repo. Any push to `main` triggers an automatic reconciliation cycle — no manual `helm upgrade` needed.

---

## Related Repos

| Repo | Role |
|---|---|
| [microservices-app](https://github.com/dev126712/microservices-app) | Application code + per-service CI/CD pipelines |
| [micro-service-infra-management](https://github.com/dev126712/micro-service-infra-management) | Terraform — GKE cluster, networking, Vault, Grafana |
| [microservice-end-to-end](https://github.com/dev126712/microservice-end-to-end) | Overview and architecture documentation |
