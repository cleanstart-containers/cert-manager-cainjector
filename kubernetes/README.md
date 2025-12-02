# Cert-Manager CA Injector - Kubernetes Deployment Guide

Complete Kubernetes deployment guide for the CleanStart Cert-Manager CA Injector container. The CA Injector is a controller that automatically injects CA bundle data into webhook configurations and API services, ensuring that webhooks can validate TLS certificates issued by cert-manager.

## How CA Injector Works

The CA Injector is responsible for:
1. **Watching Certificates**: Monitors cert-manager Certificate resources across the cluster
2. **Extracting CA Bundles**: Reads the CA bundle from Certificate secrets
3. **Updating Webhooks**: Injects the CA bundle into:
   - MutatingWebhookConfigurations
   - ValidatingWebhookConfigurations
   - APIServices
4. **Maintaining Sync**: Continuously watches for changes and keeps webhook configurations up-to-date

This ensures that when cert-manager issues certificates for webhooks, the webhook configurations automatically reference the correct CA bundle for validation.
## Files

- `deployment.yaml` - Complete deployment manifest (ServiceAccount, ClusterRole, ClusterRoleBinding, Deployment)
- `README.md` - This documentation

## Image Details

**Image:** `cleanstart/cert-manager-cainjector:latest-dev`

**Key Features:**
- **Binary Location:** `/usr/bin/cainjector`
- **Command:** The deployment uses `command: ["/usr/bin/cainjector"]` to run the controller
- **User:** Non-root (UID 1000)
- **Architecture:** `amd64`
- **OS:** `linux`
- **SSL Certificates:** Pre-configured at `/etc/ssl/certs/ca-certificates.crt`

## Complete Deployment Steps

### Prerequisites

1. **Kubernetes cluster** (Kind, minikube, k3s, GKE, EKS, AKS, or any other)
2. **kubectl** installed and configured to access your cluster
3. **cert-manager** installed (optional, for full integration testing)

### Step 1: Verify Kubernetes Cluster

```bash
# Check cluster connectivity
kubectl cluster-info

# Verify nodes are ready
kubectl get nodes
```

### Step 2: Deploy the CA Injector

```bash
# Apply the deployment manifest
kubectl apply -f deployment.yaml

# Verify the pod is running
kubectl get pods -l app=cert-manager-cainjector -w
```

**Expected Output:**
```
NAME                                      READY   STATUS    RESTARTS   AGE
cert-manager-cainjector-xxxxxxxxxx-xxxxx  1/1     Running   0          30s
```

### Step 3: Verify Deployment

```bash
# Check pod status
kubectl get pods -l app=cert-manager-cainjector

# View pod logs (once pod is running)
kubectl logs -l app=cert-manager-cainjector

# Describe the deployment
kubectl describe deployment cert-manager-cainjector

# Check ServiceAccount
kubectl get serviceaccount cert-manager-cainjector

# Check ClusterRole and ClusterRoleBinding
kubectl get clusterrole cert-manager-cainjector
kubectl get clusterrolebinding cert-manager-cainjector
```

## Testing the Functionality

### Test 1: Verify Pod is Running and Healthy

```bash
# Check pod status
kubectl get pods -l app=cert-manager-cainjector

# View logs to ensure it started successfully
kubectl logs -l app=cert-manager-cainjector

# Expected log output should show:
# - Controller starting up
# - Leader election messages (if enabled)
# - Watching for certificates and webhook configurations
```

**Success Indicators:**
- Pod status is `Running` and `READY 1/1`
- Logs show no errors
- Logs indicate the controller is watching resources

### Test 2: Verify RBAC Permissions

```bash
# Check that ClusterRole has the correct permissions
kubectl describe clusterrole cert-manager-cainjector

# Verify ClusterRoleBinding is correct
kubectl describe clusterrolebinding cert-manager-cainjector

# Test if the ServiceAccount can access required resources
kubectl auth can-i get mutatingwebhookconfigurations \
  --as=system:serviceaccount:default:cert-manager-cainjector

kubectl auth can-i get validatingwebhookconfigurations \
  --as=system:serviceaccount:default:cert-manager-cainjector

kubectl auth can-i get apiservices \
  --as=system:serviceaccount:default:cert-manager-cainjector
```

**Expected Output:**
All commands should return `yes`, indicating the ServiceAccount has the required permissions.

### Test 3: Test CA Bundle Injection (Requires cert-manager)

If you have cert-manager installed, you can test that the CA injector properly injects CA bundles into webhook configurations:

```bash
# Check if cert-manager is installed
kubectl get pods -n cert-manager

# Create a test Certificate resource
cat <<EOF | kubectl apply -f -
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: test-cert
  namespace: default
spec:
  secretName: test-cert-tls
  issuerRef:
    name: <your-issuer-name>
    kind: ClusterIssuer
  commonName: example.com
  dnsNames:
  - example.com
EOF

# Watch for webhook configurations to be updated
kubectl get mutatingwebhookconfigurations -o yaml | grep caBundle

# Check the CA injector logs to see if it processed the webhook
kubectl logs -l app=cert-manager-cainjector --tail=50
```

### Test 4: Verify Image Configuration

```bash
# Check the running pod's image
kubectl get pod -l app=cert-manager-cainjector -o jsonpath='{.items[0].spec.containers[0].image}'

# Expected output:
# cleanstart/cert-manager-cainjector:latest-dev

# Verify the binary exists and is executable
kubectl exec -it deployment/cert-manager-cainjector -- ls -la /usr/bin/cainjector

# Check environment variables
kubectl exec -it deployment/cert-manager-cainjector -- env | grep -E "SSL_CERT_FILE|PATH"
```

**Expected Output:**
- Image should be `cleanstart/cert-manager-cainjector:latest-dev`
- Binary should exist at `/usr/bin/cainjector` and be executable
- `SSL_CERT_FILE` should be set to `/etc/ssl/certs/ca-certificates.crt`
- `PATH` should include standard system paths

### Test 5: Test Security Context

```bash
# Verify the pod is running as non-root
kubectl exec -it deployment/cert-manager-cainjector -- id
# Expected output should show UID 1000 (non-root)

# Check that capabilities are dropped
kubectl exec -it deployment/cert-manager-cainjector -- capsh --print

# Verify SSL certificates are accessible
kubectl exec -it deployment/cert-manager-cainjector -- ls -la /etc/ssl/certs/ca-certificates.crt
```

### Test 6: Monitor Resource Usage

```bash
# Check resource usage
kubectl top pod -l app=cert-manager-cainjector

# Describe the pod to see resource requests/limits
kubectl describe pod -l app=cert-manager-cainjector | grep -A 10 "Limits\|Requests"
```

## Deployment Components

The `deployment.yaml` includes:

1. **ServiceAccount** - `cert-manager-cainjector`
   - Provides identity for the pod
   - Used by ClusterRoleBinding for RBAC

2. **ClusterRole & ClusterRoleBinding** - RBAC permissions
   - **ClusterRole** (not Role) because cainjector needs cluster-wide permissions
   - Allows the injector to:
     - Read secrets and configmaps (to get CA bundles)
     - Read and update mutating/validating webhook configurations
     - Read and update API services
     - Watch cert-manager resources (certificates, issuers, etc.)
     - Read CustomResourceDefinitions (to check if cert-manager CRDs are installed)
     - Manage leader election leases (for high availability)

3. **Deployment** - Main application deployment
   - Runs the cainjector binary
   - Configured with security best practices (non-root, dropped capabilities)
   - Resource limits and requests included

## Configuration

### Environment Variables

The deployment sets the following environment variables:
- `SSL_CERT_FILE=/etc/ssl/certs/ca-certificates.crt` - SSL certificate bundle location
- `PATH` - Standard system PATH
- `POD_NAME` - Automatically set from pod metadata
- `POD_NAMESPACE` - Automatically set from pod metadata

### Resource Limits

- **Requests:** CPU: 100m, Memory: 128Mi
- **Limits:** CPU: 500m, Memory: 512Mi

### Security Context

- Runs as non-root user (UID 1000)
- All capabilities dropped
- No privilege escalation allowed
- Read-only root filesystem: false (may be required for some operations)

This ensures that when cert-manager issues certificates for webhooks, the webhook configurations automatically reference the correct CA bundle for validation.

## Integration with Cert-Manager

To use this custom cainjector image with cert-manager, you typically need to:

1. **Replace the default cainjector** in your cert-manager installation
2. **Or run alongside cert-manager** for testing purposes

### Option 1: Replace Default CA Injector

If you want to replace the default cert-manager cainjector:

```bash
# Scale down the default cainjector
kubectl scale deployment cert-manager-cainjector -n cert-manager --replicas=0

# Deploy this custom cainjector in the cert-manager namespace
# (You'll need to update the namespace in deployment.yaml)
```

### Option 2: Run for Testing

For testing purposes, you can run this cainjector alongside the default one (in a different namespace or with different labels). However, having multiple cainjectors may cause conflicts.

## Cleanup

To remove the deployment:

```bash
kubectl delete -f deployment.yaml
```
Or delete individual resources:

```bash
kubectl delete deployment cert-manager-cainjector
kubectl delete serviceaccount cert-manager-cainjector
kubectl delete clusterrole cert-manager-cainjector
kubectl delete clusterrolebinding cert-manager-cainjector
```

Deleting the ClusterRole and ClusterRoleBinding will affect cluster-wide permissions, so ensure no other resources depend on them before deletion.

