***** Step-by-Step Guide to Manage SSL Termination on Kubernetes Using Ingress *****

This is to manage SSL termination an advanced feature of Ingress on K8s cluster.

**Prerequisites:**
1. A running Kubernetes cluster
2. `kubectl` installed and configured to interact with your cluster
3. Basic understanding of Kubernetes objects like Pods, Services, and Deployments
4. An Ingress controller installed (e.g., Nginx Ingress Controller)

Step 1: Install Cert-Manager
Cert-Manager is a popular Kubernetes add-on that automates the management and issuance of TLS certificates from various issuing sources.

1. **Add the Jetstack Helm repository:**

```sh
helm repo add jetstack https://charts.jetstack.io
helm repo update
```

2. **Install Cert-Manager:**

```sh
kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.8.0/cert-manager.yaml
```

Verify the installation:

```sh
kubectl get pods --namespace cert-manager
```

Step 2: Create a ClusterIssuer
A ClusterIssuer is a resource that defines how cert-manager should request certificates. Here, we'll use Let's Encrypt as the Certificate Authority (CA).

1. **Create a ClusterIssuer for Let's Encrypt:**

```yaml
# cluster-issuer.yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: your-email@example.com # Replace with your email
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
```

Apply the ClusterIssuer:

```sh
kubectl apply -f cluster-issuer.yaml
```

Step 3: Create a Certificate Resource
A Certificate resource specifies the desired certificate and the details of how to obtain it.

1. **Create a Certificate resource:**

```yaml
# certificate.yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: nginx-cert
  namespace: default
spec:
  secretName: nginx-tls
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  commonName: your-domain.com
  dnsNames:
  - your-domain.com
```

Apply the Certificate resource:

```sh
kubectl apply -f certificate.yaml
```

Cert-Manager will now request a certificate from Let's Encrypt and store it in a secret named `nginx-tls`.

Step 4: Update the Ingress Resource
Modify the Ingress resource to use the TLS certificate.

1. **Update the Ingress resource:**

```yaml
# nginx-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  tls:
  - hosts:
    - your-domain.com
    secretName: nginx-tls
  rules:
  - host: your-domain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80
```

Apply the updated Ingress resource:

```sh
kubectl apply -f nginx-ingress.yaml
```

Step 5: Verify the Setup
1. **Check the status of the certificate:**

```sh
kubectl describe certificate nginx-cert
```

2. **Test your setup:**
Open a web browser and navigate to `https://your-domain.com`. Ensure that the connection is secure and that the certificate is valid.

### Additional Tips
- **Renewals:** Cert-Manager automatically handles the renewal of certificates before they expire.
- **Customizations:** Depending on your needs, you can customize Cert-Manager solvers, use DNS challenges for wildcard certificates, and more.
- **Monitoring:** Keep an eye on the Cert-Manager logs and resources to ensure everything is functioning correctly.

This guide provides a basic setup for SSL termination using Let's Encrypt and Cert-Manager on a Kubernetes cluster. Adjust configurations and practices based on your specific requirements and environment.
