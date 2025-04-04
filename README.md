# Teleport on Kubernetes with Rancher, HAProxy, and Ingress Nginx

This repository documents a working setup for deploying [Teleport](https://goteleport.com/) on a 6-node Kubernetes cluster managed by Rancher. It integrates HAProxy as an external load balancer, Ingress Nginx for routing, Cert-Manager for TLS, and uses Rancher’s `local-path` storage for session recordings. After four days of troubleshooting, this config is production-ready and supports SSH, Kubernetes access, and the Teleport web UI—all multiplexed on port 443.

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

### 2. Install with Helm
```bash
helm upgrade --install teleport-cluster teleport/teleport-cluster \
  --namespace teleport-cluster --create-namespace \
  -f teleport-values.yaml

### 3. apply Ingress Config
```bash
kubectl apply -f teleport-ingress.yaml
