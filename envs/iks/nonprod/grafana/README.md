# Grafana — IKS nonprod environment values

This folder contains all environment-specific configuration for Grafana on IKS nonprod.

## Helm values — `values.yaml`

| Key | Current value | Description |
|-----|---------------|-------------|
| `ingress.ingressClassName` | `public-iks-k8s-nginx` | IKS public ingress class — fixed for all IKS clusters |
| `ingress.hosts` | `[]` (catch-all) | Matches all hostnames on the ingress controller |
| `persistence.storageClassName` | `ibmc-vpc-block-10iops-tier` | IKS VPC block storage — fixed for all IKS clusters |

## Enabling TLS with the cluster certificate

IKS clusters get a TLS certificate managed by IBM Cloud when you register an NLB DNS subdomain via:

```sh
ibmcloud ks nlb-dns create cluster --cluster <cluster-name> --ip <lb-ip> --secret-namespace default
```

This deposits a TLS secret into the `default` namespace named after the cluster ID, e.g.:
`idig-install-cluster-e7d3d93b8b317d269525bf063b24f98d-0000`

To use it for Grafana:

### Step 1 — Copy the cluster TLS secret into the `grafana` namespace

The ingress controller requires the TLS secret to be in the same namespace as the Ingress resource.
Copy it as `grafana-tls-secret` (the name referenced in `values.yaml`):

```sh
kubectl get secret idig-install-cluster-e7d3d93b8b317d269525bf063b24f98d-0000 \
  -n default -o json \
  | jq 'del(.metadata.resourceVersion, .metadata.uid, .metadata.creationTimestamp, .metadata.ownerReferences) | .metadata.name = "grafana-tls-secret" | .metadata.namespace = "grafana"' \
  | kubectl apply -f -
```

> Note: this is a one-time manual step. The secret will not auto-renew when IBM Cloud rotates the cert.
> To automate renewal, consider a `ClusterSecretStore` + `ExternalSecret` (see git history for a prior attempt).

### Step 2 — Uncomment and set `hosts` and `tls` in `values.yaml`

```yaml
ingress:
  hosts:
    - grafana.<cluster-id>.<region>.containers.appdomain.cloud
  tls:
    - secretName: <secret-name-from-default-namespace>
      hosts:
        - grafana.<cluster-id>.<region>.containers.appdomain.cloud
```

For this cluster:
- Hostname: `grafana.idig-install-cluster-e7d3d93b8b317d269525bf063b24f98d-0000.eu-de.containers.appdomain.cloud`
- Secret name: `idig-install-cluster-e7d3d93b8b317d269525bf063b24f98d-0000`
