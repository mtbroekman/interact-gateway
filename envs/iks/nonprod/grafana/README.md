# Grafana — IKS nonprod environment values

This folder contains all environment-specific configuration for Grafana on IKS nonprod.
When deploying to a new cluster, update the values marked below.

## Helm values — `values.yaml`

| Key | Current value | Description |
|-----|---------------|-------------|
| `ingress.ingressClassName` | `public-iks-k8s-nginx` | IKS public ingress class — fixed for all IKS clusters |
| `ingress.hosts` | `grafana.idig-install-cluster-e7d3d93b8b317d269525bf063b24f98d-0000.eu-de.containers.appdomain.cloud` | NLB DNS hostname — cluster-specific |
| `ingress.tls[].secretName` | `grafana-tls-secret` | The ESO-mirrored secret name — does not change per cluster |
| `persistence.storageClassName` | `ibmc-vpc-block-10iops-tier` | IKS VPC block storage — fixed for all IKS clusters |

## Kustomize patch — `tls-secret-patch.yaml`

| Key | Current value | Description |
|-----|---------------|-------------|
| `remoteRef.key` | `idig-install-cluster-e7d3d93b8b317d269525bf063b24f98d-0000` | The TLS secret name in the `default` namespace, created by `ibmcloud ks nlb-dns create`. Matches the cluster ID. |

The secret name can be found with:
```sh
kubectl get secrets -n default | grep e7d3d93b8b317d269525bf063b24f98d
```
or
```sh
ibmcloud ks nlb-dns ls --cluster <cluster-name>
```
