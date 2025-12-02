# Cert-Manager CA Injector - CleanStart Container

A security-hardened container image for cert-manager's CA Injector controller, enabling automatic CA bundle injection into webhook configurations and API services for seamless TLS certificate validation.

## Overview

The cert-manager CA Injector is a Kubernetes controller that automatically injects CA bundle data from cert-manager-issued certificates into webhook configurations and API services. This ensures that Kubernetes webhooks and API services can properly validate TLS certificates issued by cert-manager, enabling secure communication between components without manual CA bundle management.

**Key Features:**
* Automatic CA bundle injection into MutatingWebhookConfigurations
* Automatic CA bundle injection into ValidatingWebhookConfigurations
* Automatic CA bundle injection into APIServices
* Automatic CA bundle injection into CustomResourceDefinitions
* Integration with cert-manager for automated certificate lifecycle management
* Leader election support for high availability deployments
* Metrics endpoint for monitoring and observability
* Lightweight and efficient operation with minimal resource requirements
* Secure CA bundle handling and validation
* Continuous monitoring and automatic updates of webhook configurations

**Common Use Cases:**
* Automated TLS certificate provisioning for Kubernetes webhooks
* Secure communication between Kubernetes API server and webhook services
* Automated CA bundle management for admission webhooks
* API service certificate validation automation
* Multi-webhook certificate management in Kubernetes clusters
* Development and staging environment webhook certificate automation
* CI/CD pipeline webhook certificate provisioning
* Microservices architecture with automated webhook certificate management
* Custom resource definition webhook certificate automation

## What Cert-Manager CA Injector Does

The CA Injector operates as a controller that continuously monitors and updates webhook configurations:

1. **Monitors Certificates**: Watches cert-manager Certificate resources across all namespaces to detect new certificates and CA bundles
2. **Extracts CA Bundles**: Reads CA bundle data from Certificate secrets when certificates are issued or renewed
3. **Injects into Webhooks**: Automatically updates MutatingWebhookConfigurations and ValidatingWebhookConfigurations with the correct CA bundles
4. **Injects into API Services**: Updates APIServices with CA bundles for proper TLS validation
5. **Injects into CRDs**: Updates CustomResourceDefinitions with CA bundles for webhook validation
6. **Maintains Sync**: Continuously watches for changes and keeps webhook configurations up-to-date automatically
7. **Leader Election**: Uses leader election to ensure only one instance is active in high availability deployments

## Image Details

**Image:** `cleanstart/cert-manager-cainjector:latest-dev`

**Key Specifications:**
* **Binary Location:** `/usr/bin/cainjector`
* **User:** Non-root (UID 1000)
* **Architecture:** `amd64`
* **OS:** `linux`
* **Metrics Port:** `9402` (default metrics HTTP port)
* **SSL Certificates:** Pre-configured at `/etc/ssl/certs/ca-certificates.crt`

## How It Works

The CleanStart cert-manager CA Injector image provides a security-hardened implementation of the cert-manager cainjector component:

1. **Security Hardening**: Runs as non-root user (UID 1000) with all Linux capabilities dropped, following the principle of least privilege
2. **Privilege Escalation Prevention**: Configured with `allowPrivilegeEscalation: false` to prevent privilege escalation attacks
3. **Pre-Configured SSL/TLS**: Includes complete SSL certificate bundle for secure connections to Kubernetes API server
4. **Kubernetes Integration**: Automatically receives pod metadata (POD_NAME, POD_NAMESPACE) through downward API for proper logging
5. **Resource Management**: Deployed with conservative resource requests and limits for predictable resource usage
6. **Health Monitoring**: Built-in metrics endpoint enables Kubernetes monitoring and observability
7. **Leader Election**: Supports leader election for high availability deployments, ensuring only one active instance

When cert-manager is installed, the CA Injector automatically ensures that all webhook configurations reference the correct CA bundles from cert-manager-issued certificates, enabling seamless TLS validation without manual intervention.

## Kubernetes Deployment

The `kubernetes/` directory contains a complete, production-ready Kubernetes deployment:

* `deployment.yaml` - Complete deployment manifest (ServiceAccount, ClusterRole, ClusterRoleBinding, Deployment)
* `README.md` - Comprehensive deployment guide with step-by-step instructions, testing procedures, and troubleshooting

## Configuration

### Ports

* **9402** - Metrics HTTP port (TCP) - Exposes Prometheus metrics for monitoring

### Environment Variables

* `SSL_CERT_FILE=/etc/ssl/certs/ca-certificates.crt` - SSL certificate path for secure communication with Kubernetes API server
* `PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin` - Standard PATH environment variable
* `POD_NAME` - Automatically populated from Kubernetes pod metadata via downward API
* `POD_NAMESPACE` - Automatically populated from Kubernetes pod metadata via downward API

### RBAC Permissions

The CA Injector requires cluster-scoped permissions to:
* Read secrets and configmaps (to extract CA bundles from certificate secrets)
* Read and update MutatingWebhookConfigurations and ValidatingWebhookConfigurations
* Read and update APIServices
* Read and update CustomResourceDefinitions
* Watch cert-manager resources (certificates, certificaterequests, issuers, clusterissuers)
* Read CustomResourceDefinitions (to check if cert-manager CRDs are installed)
* Manage leader election leases (for high availability)

## Best Practices

* Deploy the CA Injector with proper resource limits to ensure predictable resource usage
* Monitor CA Injector logs for injection success rates and troubleshooting
* Use the metrics endpoint for Prometheus monitoring and alerting
* Keep the container image updated with the latest security patches
* Implement proper network policies if required by your security policies
* Review cert-manager Certificate resources to ensure proper CA bundle generation
* Enable leader election for high availability deployments
* Monitor webhook configurations to verify CA bundles are being injected correctly

## Security Notes

* The container runs as non-root user (UID 1000)
* All Linux capabilities are dropped for security
* Privilege escalation is prevented (`allowPrivilegeEscalation: false`)
* SSL certificates are pre-configured for secure communication with Kubernetes API server
* Security context prevents privilege escalation and enforces non-root execution
* CA bundles are read from secrets and injected securely into webhook configurations
* ClusterRole permissions follow the principle of least privilege
* Leader election ensures only one active instance, reducing attack surface

## Observability

The CA Injector provides structured logging that includes:
* Controller startup and initialization events
* Certificate monitoring and CA bundle extraction events
* Webhook configuration update events (mutating, validating, API services, CRDs)
* Leader election status and lease acquisition
* Error conditions and failure reasons
* Metrics server startup and binding status

The CA Injector exposes a Prometheus metrics endpoint on port 9402, providing observability into:
* Controller reconciliation rates
* Webhook configuration update counts
* Error rates and failure metrics
* Leader election status
* Certificate monitoring statistics

Health check functionality allows Kubernetes to monitor CA Injector readiness. The metrics endpoint can be used for Kubernetes liveness and readiness probes, enabling proper integration with Kubernetes monitoring systems.

For deployment instructions, configuration details, and troubleshooting guides, see the `kubernetes/README.md` file in the `kubernetes/` directory.

