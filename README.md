Prereq: 


ArgoCD installed - via Helm

ibm-entitlement-key secret created.

Apply bootstrap.yaml.

See:

https://www.ibm.com/docs/en/api-connect/software/12.1.0?topic=requirements-kubernetes-prerequisites-installing-datapower-nano-gateway-subsystem

https://www.ibm.com/docs/en/api-connect/software/12.1.0?topic=subsystems-installing-datapower-nano-gateway-subsystem-kubernetes

---

## Domain names / placeholders

The following files contain `REPLACE_ME_*` placeholders that must be set before deployment.
Placeholders in `components/` are shared across all environments; those in `argocd/install-helm/` are environment-specific.

| Placeholder | File | IKS example | AKS example | OCP example |
|---|---|---|---|---|
| `REPLACE_ME_ENDPOINT_DOMAIN` | `components/idig/base/idig-cluster-dev.yaml` | `envoy.mycluster.eu-de.containers.appdomain.cloud` | `envoy.mycluster.westeurope.cloudapp.azure.com` | `apps.mycluster.example.com` |
| `REPLACE_ME_ENDPOINT_DOMAIN` | `components/idig/base/collector-config-patch.yaml` | same as above | same as above | same as above |
| `REPLACE_ME_ENDPOINT_DOMAIN` | `components/coredns/base/coredns-custom-hosts.yaml` | — | `envoy.mycluster.westeurope.cloudapp.azure.com` | — |
| `REPLACE_ME_ENVOY_IP` | `components/coredns/base/coredns-custom-hosts.yaml` | — | `<Envoy Gateway LoadBalancer IP>` | — |
| `REPLACE_ME_CLUSTER_SUBDOMAIN` | `argocd/install-helm/values.yaml` | `mycluster.eu-de.containers.appdomain.cloud` | — | — |
| `REPLACE_ME_CLUSTER_SECRET` | `argocd/install-helm/values.yaml` | `<IKS cluster TLS secret name>` | — | — |
| `REPLACE_ME_CLUSTER_SUBDOMAIN` | `argocd/install-helm/values-aks.yaml` | — | `mycluster.westeurope.cloudapp.azure.com` | — |

---

## Environment differences

| Concern | IKS (`envs/iks`) | AKS (`envs/aks`) | OCP (`envs/odf`) |
|---|---|---|---|
| cert-manager | Jetstack Helm chart | Jetstack Helm chart | OLM Subscription (`openshift-cert-manager-operator`) |
| Ingress / routing | Envoy Gateway (HTTPRoutes) | Envoy Gateway (HTTPRoutes) | Native OCP Routes — `envoy-operator` not used |
| CoreDNS patch | Not used | Applied (`envs/aks/nonprod/coredns`) | Not applicable |
| Valkey storage | `ibmc-vpc-block-10iops-tier` | `ocs-storagecluster-ceph-rbd` | `ocs-storagecluster-ceph-rbd` |
| Loki / Tempo storage | `ibmc-vpc-block-10iops-tier` | `ibmc-vpc-block-10iops-tier` | `ocs-storagecluster-ceph-rbd` |
| Wildcard routes | Not required | Not required | Must enable on `IngressController` (see below) |
| IDIG version | 12.1.1.2 | 12.1.1.2 | 12.1.1.2 |

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

See IBM documentation:

OpenShift:
- https://www.ibm.com/docs/en/dp-interact-gateway/12.1.1?topic=openshift-installing-prerequisite-components
- https://www.ibm.com/docs/en/dp-interact-gateway/12.1.1?topic=openshift-installing-operators
- https://www.ibm.com/docs/en/dp-interact-gateway/12.1.1?topic=openshift-deploying-idig-cluster
- https://www.ibm.com/docs/en/dp-interact-gateway/12.1.1?topic=openshift-enabling-wildcard-routes-datapower-nanogateway

Kubernetes (IKS/AKS):
- https://www.ibm.com/docs/en/dp-interact-gateway/12.1.1?topic=kubernetes-installing-prerequisite-components
- https://www.ibm.com/docs/en/dp-interact-gateway/12.1.1?topic=kubernetes-installing-operators
- https://www.ibm.com/docs/en/dp-interact-gateway/12.1.1?topic=kubernetes-deploying-idig-cluster
