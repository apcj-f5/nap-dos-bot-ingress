# NGINX App Protect v4 as Ingress Controller with DoS Protection - Implementation Guide

This guide walks through the deployment of NGINX App Protect (NAP) v4 as a Kubernetes Ingress Controller with DoS protection enabled using only Kubernetes manifests (no Helm) for maximum flexibility.

## Prerequisites

- Kubernetes cluster (v1.19 or newer)
- kubectl configured to communicate with your cluster
- Access to NGINX App Protect container registry
- Valid license for NGINX App Protect

## Step 1: Create Namespace and RBAC Configuration Files

First, create the namespace:

```bash
kubectl create namespace nginx-ingress
```

Apply the ServiceAccount and RBAC configuration files:

```bash
kubectl apply -f nginx-ingress-service-account.yaml
kubectl apply -f nginx-ingress-rbac.yaml
```

## Step 2: Prepare Authentication for NGINX Registry

NGINX App Protect images are hosted in a private registry. Create a Kubernetes secret to allow pulling these images:

```bash
kubectl create secret docker-registry regcred \
  --namespace=nginx-ingress \
  --docker-server=private-registry.nginx.com \
  --docker-username=<your-username> \
  --docker-password=<your-password> \
  --docker-email=<your-email>
```

## Step 3: Apply NGINX ConfigMap

Apply the ConfigMap with NGINX configuration:

```bash
kubectl apply -f nginx-config.yaml
```

## Step 4: Deploy NGINX Ingress Controller with App Protect

Apply the Ingress Controller deployment and service:

```bash
kubectl apply -f nginx-ingress-deployment.yaml
kubectl apply -f nginx-ingress-service.yaml
```

## Step 5: Deploy DoS Arbitrator

Deploy the DoS Arbitrator component:

```bash
kubectl apply -f dos-arbitrator.yaml
```

## Step 6: Apply Custom Resource Definitions (CRDs)

The necessary CRDs can be found in the NGINX Ingress Controller repository:

```bash
# From the cloned repository
kubectl apply -f default-templates-for-kubernetes-ingress/deployments/common/crds/appprotect.f5.com_aplogconfs.yaml
kubectl apply -f default-templates-for-kubernetes-ingress/deployments/common/crds/appprotect.f5.com_appolicies.yaml
kubectl apply -f default-templates-for-kubernetes-ingress/deployments/common/crds/appprotect.f5.com_apusersigs.yaml
kubectl apply -f default-templates-for-kubernetes-ingress/deployments/common/crds/appprotectdos.f5.com_apdoslogconfs.yaml
kubectl apply -f default-templates-for-kubernetes-ingress/deployments/common/crds/appprotectdos.f5.com_apdospolicies.yaml
kubectl apply -f default-templates-for-kubernetes-ingress/deployments/common/crds/appprotectdos.f5.com_dosprotectedresources.yaml
```

## Step 7: Create App Protect DoS Policy

Apply the DoS policy:

```bash
kubectl apply -f dos-policy.yaml
```

## Step 8: Apply App Protect DoS Logging Configuration

Apply the log configuration:

```bash
kubectl apply -f dos-logconf.yaml
```

## Step 9: Apply App Protect DoS Protected Resource

Apply the protected resource:

```bash
kubectl apply -f dos-protected-resource.yaml
```

## Step 10: Deploy Sample Application

Deploy the sample application:

```bash
kubectl apply -f sample-app.yaml
```

## Step 11: Create Ingress Resource with DoS Protection

Apply the Ingress resource:

```bash
kubectl apply -f protected-ingress.yaml
```

## Step 12: Configure WAF Protection (Optional)

If you also want to enable WAF protection in addition to DoS:

```bash
kubectl apply -f app-protect-policy.yaml
kubectl apply -f app-protect-logconf.yaml
kubectl apply -f ingress-with-waf.yaml
```

## Step 13: Verify the Installation

Check if all components are running:

```bash
kubectl get pods -n nginx-ingress
kubectl get pods -n default
kubectl get svc -n nginx-ingress
kubectl get ingress -n default
```

Test the protected application:

```bash
# Get the external IP of the NGINX Ingress Controller
EXTERNAL_IP=$(kubectl get svc nginx-ingress -n nginx-ingress -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

# Test the application with a host header
curl -H "Host: my-protected-app.example.com" http://$EXTERNAL_IP/
```

## Step 14: Set Up Monitoring and Logging

Apply the log monitoring setup:

```bash
kubectl apply -f log-pv.yaml
```

Check logs:

```bash
kubectl logs -f log-viewer -n nginx-ingress
```

## Advanced Configuration

### Configure External Syslog

For sending logs to an external syslog server:

```bash
kubectl apply -f syslog-config.yaml
```

Update your APDosLogConf and APDosProtectedResource to point to this service.

## Troubleshooting

1. **Image Pull Errors**:

   - Verify your registry credentials
   - Check imagePullSecrets is correctly configured

2. **DoS Protection Not Working**:

   - Verify the arbitrator is running
   - Check connections between ingress controller and arbitrator

3. **Logging Issues**:

   - Verify log configuration
   - Check volume mounts for logs

4. **Performance Tuning**:
   - Adjust resource limits based on traffic patterns
   - Modify DoS policy thresholds if needed

## References

- [NGINX Ingress Controller GitHub Repository](https://github.com/nginxinc/kubernetes-ingress)
- [NGINX App Protect Documentation](https://docs.nginx.com/nginx-app-protect/)
- [NGINX App Protect DoS Documentation](https://docs.nginx.com/nginx-app-protect-dos/)
- [Example Manifests Directory](https://github.com/nginxinc/kubernetes-ingress/tree/main/examples/custom-resources/app-protect-dos)
