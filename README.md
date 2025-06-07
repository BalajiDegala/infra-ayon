# AYON and Zulip Helm Charts

This repository contains example Helm charts for deploying [AYON](https://github.com/BalajiDegala/ayon-docker), [Zulip](https://zulip.com), and [Keycloak](https://www.keycloak.org) with ingress and MetalLB for bare-metal clusters.

The charts are located in the `helm/` directory:

- `helm/keycloak` – Keycloak instance providing SSO
- `helm/ayon` – AYON server configured to use Keycloak
- `helm/zulip` – Zulip chat server configured to use Keycloak

These charts are intentionally simple and meant as a starting point for a small team (≈250 users). Further tuning might be required for production.

## Prerequisites

1. A Kubernetes cluster (v1.25+) with [MetalLB](https://metallb.universe.tf/) installed and configured to provide external LoadBalancer IPs.
2. An Ingress controller (e.g. nginx). The charts assume the ingress class name `nginx`.
3. `helm` installed locally to deploy the charts.

## Quick Start

1. **Install Keycloak**

   ```bash
   helm install keycloak ./helm/keycloak
   ```

   After Keycloak is running, create a realm and OAuth clients named `ayon` and `zulip`. Note the client IDs and secrets and update `helm/ayon/values.yaml` and `helm/zulip/values.yaml` accordingly.

2. **Install AYON**

   Build and push the AYON Docker image from [BalajiDegala/ayon-docker](https://github.com/BalajiDegala/ayon-docker) to your registry and set `image.repository` in `helm/ayon/values.yaml`.

   ```bash
   helm install ayon ./helm/ayon
   ```

3. **Install Zulip**

   ```bash
   helm install zulip ./helm/zulip
   ```

## MetalLB Configuration

MetalLB should provide LoadBalancer addresses for each service. Example:

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.5/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.5/manifests/metallb.yaml
# Configure address pool
kubectl apply -f metallb-config.yaml
```

`metallb-config.yaml` might look like:

```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: default-pool
  namespace: metallb-system
spec:
  addresses:
    - 192.168.1.240-192.168.1.250
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: l2
  namespace: metallb-system
```

## Testing SSO

1. Access Keycloak (e.g. `http://keycloak.local`) and log in with the admin credentials defined in `helm/keycloak/values.yaml`.
2. Create a realm and clients named `ayon` and `zulip` with `http://ayon.local` and `http://zulip.local` as valid redirect URIs.
3. Update the corresponding client secrets in the AYON and Zulip chart values.
4. Deploy the charts and visit the ingress URLs. Login buttons for Keycloak should appear and allow authentication via Keycloak.

## Packaging

A helper script `package.sh` is provided to create `helm.zip` containing all charts:

```bash
./package.sh
```

The resulting `helm.zip` can be distributed for installation.
