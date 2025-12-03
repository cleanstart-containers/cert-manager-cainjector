# CleanStart Container for Cert-Manager CA Injector

A security-hardened container image for cert-manager's CA Injector controller. Automatically injects CA bundles into Kubernetes webhook configurations, API services, and CRDs to ensure secure TLS validation across the cluster. The cert-manager-cainjector container is a critical component of the cert-manager ecosystem, responsible for injecting CA bundles into webhook configurations. It automatically updates CA bundles in webhook configurations when certificates are renewed, ensuring continuous secure communication between Kubernetes components. This container is essential for maintaining proper TLS certificate management in enterprise Kubernetes environments.

**📌 Base Foundation:** Security-hardened, minimal base OS designed for enterprise containerized environments from CleanStart Registry.

**Image Path:** `cleanstart/cert-manager-cainjector`

**Registry:** CleanStart Registry

---

## Key Features

Core capabilities and strengths of this container:

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
- Kubernetes-native implementation

---

## Common Use Cases

Typical scenarios where this container excels:

- Automated TLS verification for Kubernetes webhooks
- Secure API server ↔ webhook communication
- Automated CA rotation after certificate renewal
- Cluster-wide CA bundle propagation
- Webhook security automation in microservices
- CI/CD environments requiring automated certificate trust
- CRD webhook certificate management
- Kubernetes cluster certificate management
- Enterprise PKI infrastructure integration

---

## Getting Started

### Pull Commands

Download the runtime container images:
```bash
docker pull cleanstart/cert-manager-cainjector:latest
```
```bash
docker pull cleanstart/cert-manager-cainjector:latest-dev
```

### Basic Test Run

Run the container with a basic test command:
```bash
kubectl run cainjector-test \
  --image=cleanstart/cert-manager-cainjector:latest-dev \
  --restart=Never \
  -- /usr/bin/cainjector --help
```

### Production Deployment

Recommended production deployment with hardened security:
```bash
docker run -d --name cert-manager-cainjector-prod \
  --read-only \
  --security-opt=no-new-privileges \
  --user 1000:1000 \
  cleanstart/cert-manager-cainjector:latest
```

---

## Configuration

### Environment Variables

Configuration options available through environment variables:

| Variable | Default | Description |
|----------|---------|-------------|
| `LOG_LEVEL` | `info` | Logging verbosity level |
| `NAMESPACE` | `cert-manager` | Target namespace for operations |
| `LEADER_ELECTION_NAMESPACE` | `kube-system` | Namespace for leader election |
| `KUBECONFIG` | `/etc/kubernetes/kubeconfig` | Path to kubeconfig file |
| `SSL_CERT_FILE` | `/etc/ssl/certs/ca-certificates.crt` | SSL certificate file path |
| `POD_NAME` | - | Pod name (auto-populated) |
| `POD_NAMESPACE` | - | Pod namespace (auto-populated) |

### Kubernetes Ports

- **9402** – Metrics HTTP port (Prometheus compatible)

### Required RBAC Permissions

The CA Injector needs cluster-scoped permissions to:

- Read Secrets (extract CA bundles)
- Update webhook configurations (mutating + validating)
- Update APIService resources
- Update CustomResourceDefinitions
- Watch cert-manager certificate resources
- Manage leader election leases

---

## Security

### Security Best Practices

Recommended security configurations and practices:

- Use RBAC policies to limit access permissions
- Enable Pod Security Policies
- Implement network policies for pod communication
- Regular security scanning of container images
- Monitor certificate renewal events
- Implement proper secret management
- Enable leader election for HA deployments
- Set CPU/Memory resource limits
- Monitor metrics on port 9402
- Apply Kubernetes network policies if required
- Keep container images updated regularly
- Validate cert-manager Certificate and CA bundle configuration
- Use read-only filesystem and drop all Linux capabilities

### Kubernetes Security Context

Recommended security context for Kubernetes deployments:
```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1001
  fsGroup: 1001
  readOnlyRootFilesystem: true
  allowPrivilegeEscalation: false
  capabilities:
    drop: ["ALL"]
  seccompProfile:
    type: RuntimeDefault
```

---

## Architecture Support

### Multi-Platform Images
```bash
docker pull --platform linux/amd64 cleanstart/cert-manager-cainjector:latest
```
```bash
docker pull --platform linux/arm64 cleanstart/cert-manager-cainjector:latest
```

---

## Resources

- **Official Documentation:** https://cert-manager.io/docs/
- **Provenance / SBOM / Signature:** https://images.cleanstart.com/images/cert-manager-cainjector
- **Docker Hub:** https://hub.docker.com/r/cleanstart/cert-manager-cainjector
- **CleanStart All Images:** https://images.cleanstart.com
- **CleanStart Community Images:** https://hub.docker.com/u/cleanstart

---

## Vulnerability Disclaimer

CleanStart provides Docker images that use third-party open-source components maintained by independent contributors. While CleanStart applies industry-standard security practices, it cannot guarantee the integrity of upstream components.

Users acknowledge that open-source software may contain undisclosed vulnerabilities or introduce new risks. CleanStart shall not be held liable for security issues originating from third-party libraries, including but not limited to zero-day exploits, supply-chain attacks, or contributor-introduced risks.

**Security remains a shared responsibility** — CleanStart provides secure updated images, while users are responsible for evaluating deployments and implementing appropriate controls.
