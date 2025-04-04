# Teleport on Kubernetes , HAProxy, Ingress Nginx
![Kubernetes Logo](https://raw.githubusercontent.com/kubernetes-sigs/kubespray/master/docs/img/kubernetes-logo.png) 
![Teleport Logo](https://avatars.githubusercontent.com/u/10781132?s=48&v=4) 

Teleport works with SSH, Kubernetes, databases, RDP, and web services.

* Architecture: https://goteleport.com/docs/reference/architecture/
* Getting Started: https://goteleport.com/docs/get-started/

<div align="center">
   <a href="https://goteleport.com/download">
   <img src="https://github.com/gravitational/teleport/blob/master/assets/img/hero-teleport-platform.png" width=750/>
   </a>
   <div align="center" style="padding: 25px">
      <a href="https://goteleport.com/download">
      <img src="https://img.shields.io/github/v/release/gravitational/teleport?sort=semver&label=Release&color=651FFF" />
      </a>
      <a href="https://golang.org/">
      <img src="https://img.shields.io/github/go-mod/go-version/gravitational/teleport?color=7fd5ea" />
      </a>
      <a href="https://github.com/gravitational/teleport/blob/master/CODE_OF_CONDUCT.md">
      <img src="https://img.shields.io/badge/Contribute-🙌-green.svg" />
      </a>
      <a href="https://www.gnu.org/licenses/agpl-3.0.en.html">
      <img src="https://img.shields.io/badge/AGPL-3.0-red.svg" />
      </a>
   </div>
</div>
</br>

This repository documents a working setup for deploying [Teleport](https://goteleport.com/) on a 6-node Kubernetes cluster . It integrates HAProxy as an external load balancer, Ingress Nginx for routing, Cert-Manager for TLS, and uses Rancher’s `local-path` storage for session recordings. this config is production-ready and supports SSH, Kubernetes access, and the Teleport web UI—all multiplexed on port 443.

## Architecture
- **Cluster:** 6 nodes (e.g., 172.x.x.x) running Kubernetes. you can use link for config cluster [Ahmad Rafiee](https://github.com/AhmadRafiee/DevOps_Certification/tree/bb05e45e1232fe76f579a60151dee07d04002d03/kubernetes/cluster-setup/multi-node/kubespray)
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

SSH Node Example: Install the Teleport agent on a target server and join it to the cluster [see Teleport Docs](https://goteleport.com/docs/setup/guides/joining-nodes/).
Kubernetes Cluster: Already enabled via proxy.kube.enabled. Use tsh kube login to access
;-) Good Luck
##HAProxy Configuration
Ensure HAProxy forwards traffic to your nodes

# 🔗 Links
[![linkedin](https://img.shields.io/badge/linkedin-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/public-profile/settings?lipi=urn%3Ali%3Apage%3Ad_flagship3_profile_self_edit_contact-info%3ByClokCpERJCU%2FNTJb47Yqg%3D%3D)
[![Telegram](https://img.shields.io/badge/telegram-0A66C2?style=for-the-badge&logo=telegram&logoColor=white)](https://t.me/Yeganloo)
