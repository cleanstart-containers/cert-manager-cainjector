# CleanStart Container for Cert-Manager CA Injector

A security-hardened container image for cert-manager's CA Injector controller.  
Automatically injects CA bundles into Kubernetes webhook configurations, API services, and CRDs to ensure secure TLS validation across the cluster.  
Optimized for enterprise Kubernetes environments with non-root execution and minimal attack surface.

---

## Key Features

- Automatic CA bundle injection into:
  - MutatingWebhookConfigurations
  - ValidatingWebhookConfigurations
  - APIService resources
  - CustomResourceDefinitions
- Real-time monitoring of cert-manager certificates
- Seamless integration with cert-manager certificate lifecycle
- Leader election support for high availability
- Lightweight and resource-efficient
- Secure CA bundle extraction and handling
- Built-in metrics endpoint (Prometheus compatible)

---

## Common Use Cases

- Automated TLS verification for Kubernetes webhooks  
- Secure API server ↔ webhook communication  
- Automated CA rotation after certificate renewal  
- Cluster-wide CA bundle propagation  
- Webhook security automation in microservices  
- CI/CD environments requiring automated certificate trust  
- CRD webhook certificate management  

---

## Pull Commands  
Download the runtime container images:

docker pull cleanstart/cert-manager-cainjector:latest
docker pull cleanstart/cert-manager-cainjector:latest-dev

yaml
Copy code

---

## Basic Test Run  
Run the container with a basic test command:

kubectl run cainjector-test
--image=cleanstart/cert-manager-cainjector:latest-dev
--restart=Never
-- /usr/bin/cainjector --help

yaml
Copy code

---

## Production Deployment  
Recommended production deployment with hardened security:

docker run -d --name cert-manager-cainjector-prod
--read-only
--security-opt=no-new-privileges
--user 1000:1000
cleanstart/cert-manager-cainjector:latest

yaml
Copy code

---

## Architecture Support  

### Multi-Platform Images

docker pull --platform linux/amd64 cleanstart/cert-manager-cainjector:latest
docker pull --platform linux/arm64 cleanstart/cert-manager-cainjector:latest

yaml
Copy code

---

## Kubernetes Configuration

### Ports
9402 – Metrics HTTP port

yaml
Copy code

### Environment Variables
- `SSL_CERT_FILE=/etc/ssl/certs/ca-certificates.crt`
- `POD_NAME` (auto-populated)
- `POD_NAMESPACE` (auto-populated)

### Required RBAC Permissions
The CA Injector needs cluster-scoped permissions to:
- Read Secrets (extract CA bundles)
- Update webhook configurations (mutating + validating)
- Update APIService resources
- Update CustomResourceDefinitions
- Watch cert-manager certificate resources
- Manage leader election leases

---

## Best Practices

- Enable leader election for HA deployments  
- Set CPU/Memory resource limits  
- Monitor metrics on port 9402  
- Apply Kubernetes network policies if required  
- Keep container images updated regularly  
- Validate cert-manager Certificate and CA bundle configuration  
- Use read-only filesystem and drop all Linux capabilities  

---

## Resources

- Official Documentation: https://cert-manager.io/docs/
- View Provenance, SBOM, Signature: https://images.cleanstart.com/images/cert-manager-cainjector
- CleanStart All Images: https://images.cleanstart.com
- CleanStart Community Images: https://hub.docker.com/u/cleanstart
- Docker Hub Repository: https://hub.docker.com/r/cleanstart/cert-manager-cainjector

---

## Vulnerability Disclaimer

CleanStart provides Docker images that use third-party open-source components maintained by independent contributors. While CleanStart applies industry-standard security practices, it cannot guarantee the integrity of upstream components.

Users acknowledge that open-source software may contain undisclosed vulnerabilities or introduce new risks. CleanStart shall not be held liable for security issues originating from third-party libraries, including but not limited to zero-day exploits, supply-chain attacks, or contributor-introduced risks.

Security remains a shared responsibility — CleanStart provides secure updated images, while users 
