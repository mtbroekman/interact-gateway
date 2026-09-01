Prereq: 


ArgoCD installed - via Helm

ibm-entitlement-key secret created.

Apply bootstrap.yaml.

See:

https://www.ibm.com/docs/en/api-connect/software/12.1.0?topic=requirements-kubernetes-prerequisites-installing-datapower-nano-gateway-subsystem

https://www.ibm.com/docs/en/api-connect/software/12.1.0?topic=subsystems-installing-datapower-nano-gateway-subsystem-kubernetes

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

See IBM documentation for OCP-specific installation steps:

- https://www.ibm.com/docs/en/dp-interact-gateway/12.1.1?topic=openshift-installing-prerequisite-components
- https://www.ibm.com/docs/en/dp-interact-gateway/12.1.1?topic=openshift-installing-operators
- https://www.ibm.com/docs/en/dp-interact-gateway/12.1.1?topic=openshift-deploying-idig-cluster

For Kubernetes installation (envs/iks):

- https://www.ibm.com/docs/en/dp-interact-gateway/12.1.1?topic=kubernetes-installing-prerequisite-components
