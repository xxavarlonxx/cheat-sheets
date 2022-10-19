
Argo CD is a declarative, GitOps continuous delivery tool for Kubernetes.
Application definitions, configurations, and environments should be declarative and version controlled. Application deployment and lifecycle management should be automated, auditable, and easy to understand.

**Project:** [ArgoCD](https://argo-cd.readthedocs.io/en/stable/)
**Documentation:** [Link](https://argo-cd.readthedocs.io/en/stable/)

## Steps

### Install ArgoCD in Cluster

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Create new SSL Certificate

[Link](./Use a self-signed SSL Certificate.md)

### Add Ingress Route
```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: argocd-server
  namespace: argocd
spec:
  entryPoints:
    - websecure
  routes:
    - kind: Rule
      match: Host(`argocd.example.com`)
      priority: 10
      services:
        - name: argocd-server
          port: 80
    - kind: Rule
      match: Host(`argocd.example.com`) && Headers(`Content-Type`, `application/grpc`)
      priority: 11
      services:
        - name: argocd-server
          port: 80
          scheme: h2c
  tls:
    secretName: argocd-tls-secret
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: argocd-cert
  namespace: argocd
spec:
  secretName: argocd-tls-secret
  issuerRef:
    name: ssl-issuer
    kind: ClusterIssuer
  dnsNames:
    - argocd.example.com
```

### Fix Redirect Loop
Open the `argocd-server`- Deployment Manifest
```bash
kubectl -n argocd edit deployments.apps argocd-server
```

and change this:
```yaml
containers:
      - command:
        - argocd-server
        - --insecure
```

to that:
```yaml
containers:
  - command:
      - argocd-server
      - '--insecure'

```

### First Login

Username: admin

To get the initial Password:
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

