# Teleport on Kubernetes , HAProxy, Ingress Nginx

This repository documents a working setup for deploying [Teleport](https://goteleport.com/) on a 6-node Kubernetes cluster . It integrates HAProxy as an external load balancer, Ingress Nginx for routing, Cert-Manager for TLS, and uses Rancher’s `local-path` storage for session recordings. this config is production-ready and supports SSH, Kubernetes access, and the Teleport web UI—all multiplexed on port 443.

## Architecture
- **Cluster:** 6 nodes (e.g., 172.x.x.x) running Kubernetes via Rancher.
- **HAProxy:** Forwards ports 80 and 443 to Ingress Nginx NodePorts (32080, 32443).
- **Ingress Nginx:** Routes traffic to Teleport, terminates TLS with Cert-Manager.
- **Cert-Manager:** Provides Let’s Encrypt certificates.
- **Teleport:** Runs in `standalone` mode with 3 replicas, using `multiplex` mode for all traffic on port 443.
- **Storage:** `local-path` provisioner for session recordings.

Flow: `Client → HAProxy (443) → Ingress Nginx (32443) → Teleport (3080 internally)`.

## Prerequisites
- Kubernetes cluster (Rancher-managed recommended).
- Helm installed.
- HAProxy configured (see `haproxy.cfg` below).
- Ingress Nginx deployed with NodePorts 32080 (HTTP) and 32443 (HTTPS).
- Cert-Manager installed with a `ClusterIssuer` (e.g., `letsencrypt-prod`).
- Domain (e.g., `tele.kube.yeganloo.com`) pointing to HAProxy’s IP (e.g., 185.105.184.251).

## Installation

### 1. Deploy Teleport with Helm
Add the Teleport Helm repo:
```bash
helm repo add teleport https://charts.releases.teleport.dev
helm repo update
```
### 2. Install Teleport with Helm
```bash
helm upgrade --install teleport-cluster teleport/teleport-cluster \
  --namespace teleport-cluster --create-namespace \
  -f teleport-values.yaml
```
### 3. apply Configure Ingress Config
```bash
kubectl apply -f teleport-ingress.yaml
```
### 4. Create User 
```bash
kubectl --namespace teleport-cluster exec deployment/teleport-cluster-auth -- tctl users add <your-UserName> --roles=access,editor
tsh login --proxy=<your-clustername>:443 --auth=local --user=<your-username>
```
you can use this [link](https://goteleport.com/docs/connect-your-client/tsh/#installing-tsh) to install tsh 

### 5. Add Your Resource 
Register resources (e.g., SSH nodes, Kubernetes clusters) with Teleport:

SSH Node Example: Install the Teleport agent on a target server and join it to the cluster (see Teleport Docs)[https://goteleport.com/docs/setup/guides/joining-nodes/].
Kubernetes Cluster: Already enabled via proxy.kube.enabled. Use tsh kube login to access
;-) Good Luck
##HAProxy Configuration
Ensure HAProxy forwards traffic to your nodes
