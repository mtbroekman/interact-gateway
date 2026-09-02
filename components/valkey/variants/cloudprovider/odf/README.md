# OCP Security Context Constraint for Valkey

## Why this is needed

The Valkey operator creates a `StatefulSet` with `podSecurityContext.fsGroup: 1000`
hardcoded in [`valkey_standalone_cr.yaml`](../../base/valkey_standalone_cr.yaml).

On **OpenShift**, the default SCC (`restricted-v2`) only allows UIDs and fsGroups
within the namespace-allocated range (e.g. `1000910000–1000919999`). Any pod
requesting `fsGroup: 1000` is rejected at admission with:

```
unable to validate against any security context constraint:
  provider restricted-v2: .spec.securityContext.fsGroup:
    Invalid value: []int64{1000}: 1000 is not an allowed group
```

## The fix

[`scc-anyuid.yaml`](scc-anyuid.yaml) grants the `anyuid` SCC to the `default`
ServiceAccount in the `idig` namespace. The Valkey `StatefulSet` runs as `default`
SA (no explicit `serviceAccountName` is set by the operator), so `anyuid` allows
it to use `fsGroup: 1000`.

## Scope

This RoleBinding is **OCP-only** — it lives in `variants/cloudprovider/odf/` and
is not included in IKS or AKS variants where `restricted-v2` SCC does not exist.

## Reference

- [OCP: Managing SCCs](https://docs.openshift.com/container-platform/latest/authentication/managing-security-context-constraints.html)
- [Valkey operator CR source](../../base/valkey_standalone_cr.yaml) — `spec.podSecurityContext.fsGroup`
