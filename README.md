Prereq: 


ArgoCD installed - via Helm

ibm-entitlement-key secret created.

Apply bootstrap.yaml.

See:

https://www.ibm.com/docs/en/api-connect/software/12.1.0?topic=requirements-kubernetes-prerequisites-installing-datapower-nano-gateway-subsystem

https://www.ibm.com/docs/en/api-connect/software/12.1.0?topic=subsystems-installing-datapower-nano-gateway-subsystem-kubernetes

---

## Domain names / placeholders

The following files contain `REPLACE_ME_*` placeholders that must be set before deployment:

| Placeholder | Files | Description |
|---|---|---|
| `REPLACE_ME_ENDPOINT_DOMAIN` | `components/idig/base/idig-cluster-dev.yaml` | Base domain for all IDIG endpoints (e.g. `apps.mycluster.example.com` on OCP, Envoy subdomain on IKS/AKS) |
| `REPLACE_ME_ENDPOINT_DOMAIN` | `components/idig/base/collector-config-patch.yaml` | Analytics ingestion endpoint (`ai.<domain>`) |
| `REPLACE_ME_ENDPOINT_DOMAIN` | `components/coredns/base/coredns-custom-hosts.yaml` | CoreDNS host overrides — AKS only |
| `REPLACE_ME_ENVOY_IP` | `components/coredns/base/coredns-custom-hosts.yaml` | Envoy Gateway LoadBalancer IP — AKS only |
| `REPLACE_ME_CLUSTER_SUBDOMAIN` | `argocd/install-helm/values.yaml` | ArgoCD ingress hostname — IKS |
| `REPLACE_ME_CLUSTER_SECRET` | `argocd/install-helm/values.yaml` | IKS cluster TLS secret name — IKS |
| `REPLACE_ME_CLUSTER_SUBDOMAIN` | `argocd/install-helm/values-aks.yaml` | ArgoCD ingress hostname — AKS |

---

## OpenShift (envs/odf) — IDIG 12.1.1.2

The `envs/odf/nonprod/` environment targets OpenShift Container Platform (OCP).
It differs from the Kubernetes (`envs/iks/`) environment in the following ways:

| Concern | Kubernetes (IKS) | OpenShift (ODF) |
|---|---|---|
| cert-manager | Jetstack Helm chart (`argocd/operators/cert-manager-operator.yaml`) | OLM Subscription — `cert-manager-operator.yaml` replaced with OLM resources |
| Ingress / routing | Envoy Gateway (HTTPRoutes) | Native OCP Routes — `envoy-operator` not used |
| CoreDNS patch | Applied | Not applicable on OCP |
| Valkey storage | `ibmc-vpc-block-10iops-tier` | `ocs-storagecluster-ceph-rbd` (via `components/valkey/variants/cloudprovider/odf`) |
| Loki / Tempo storage | `ibmc-vpc-block-10iops-tier` | `ocs-storagecluster-ceph-rbd` |
| IDIG version | 12.1.1.2 | 12.1.1.2 |

### Manual post-deployment step: enable wildcard routes

The NanoGateway creates a wildcard OCP Route for dynamic API routing. OCP blocks
wildcard routes by default. After deploying the IDIG cluster, a cluster-admin must
run this once on the cluster:

```bash
oc -n openshift-ingress-operator patch ingresscontroller/default \
  --type=merge -p '{"spec":{"routeAdmission":{"wildcardPolicy":"WildcardsAllowed"}}}'
```

The `IngressController` pods restart automatically. This cannot be applied via
ArgoCD as it requires cluster-admin on an OCP operator-managed resource.

See: https://www.ibm.com/docs/en/dp-interact-gateway/12.1.1?topic=openshift-enabling-wildcard-routes-datapower-nanogateway

---

See IBM documentation for OCP-specific installation steps:

- https://www.ibm.com/docs/en/dp-interact-gateway/12.1.1?topic=openshift-installing-prerequisite-components
- https://www.ibm.com/docs/en/dp-interact-gateway/12.1.1?topic=openshift-installing-operators
- https://www.ibm.com/docs/en/dp-interact-gateway/12.1.1?topic=openshift-deploying-idig-cluster
- https://www.ibm.com/docs/en/dp-interact-gateway/12.1.1?topic=openshift-enabling-wildcard-routes-datapower-nanogateway

For Kubernetes installation (envs/iks):

- https://www.ibm.com/docs/en/dp-interact-gateway/12.1.1?topic=kubernetes-installing-prerequisite-components
