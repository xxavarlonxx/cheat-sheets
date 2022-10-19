# Create and import a self-signed Certificate for SSL with cert-manager

## Steps

1. Create a Certificate Authority
    a. Create a CA private key
    ```bash
    openssl genrsa -out ca.key 4096
    ```
    b. Create a CA certificate
    ```bash
    openssl req -new -x509 -sha256 -days 365 -key ca.key -out ca.crt
    ```
    c. Import the CA certificate in the `trusted Root Ca store` of your clients
2. Convert the content of the key and crt to base64 oneline
    ```bash
    cat ca.crt | base64 -w 0
    cat ca.key | base64 -w 0
    ```
3. Create a secret object `nginx1-ca-secret.yml` and put in the key and crt content
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: nginx1-ca-secret
  namespace: cert-manager
type: Opaque
data:
  tls.crt: #Base64 String of the CRT - cat ca.crt | base64
  tls.key: #Base64 String of the Private Key - cat ca.key | base64
```
5. Create a cluster issuer object `nginx1-clusterissuer.yml`
```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: nginx1-clusterissuer
spec:
  ca:
    secretName: nginx1-ca-secret
```
7. Create a new certificate `nginx1-cert.yml` for your projects
```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: nginx1-cert
  namespace: nginx1
spec:
  secretName: nginx1-tls-secret
  issuerRef:
    name: nginx1-clusterissuer
    kind: ClusterIssuer
  dnsNames:
    - #FQDN of your domain
```
9. Add a `tls` reference in your ingress `nginx1-ingress.yml`
```yaml
tls:
  - hosts:
    - nginx.kube.home  # Your hostname
    secretName: nginx1-tls-secret # Your TLS Secret
```
11. Apply all changes

