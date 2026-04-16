# Gatekeeper Policies for OpenShift Virtualization Resource Protection

This directory contains Gatekeeper ConstraintTemplates and Constraints that protect OpenShift Virtualization resources from accidental deletion. They are the Gatekeeper equivalents of the [Kyverno policies](https://github.com/michaelkotelnikov/nad-policy) for VM resource protection.

## Prerequisites

1. **Gatekeeper installed** on your cluster
2. **Apply the Config resource** to sync VirtualMachine objects into the OPA cache:

```bash
oc apply -f config-sync.yaml
```

Without this, Gatekeeper cannot look up VirtualMachine resources during admission.

## Policies

| Policy | Protects | Blocks DELETE when... |
|---|---|---|
| **DenyNADDeleteIfAttached** | NetworkAttachmentDefinition | A VM references it via `spec.template.spec.networks[].multus.networkName` |
| **DenyPVCDeleteIfAttached** | PersistentVolumeClaim | A VM uses it via `persistentVolumeClaim.claimName` or `dataVolume.name` |
| **DenyInstancetypeDeleteIfReferenced** | VirtualMachineClusterInstancetype | A VM references it via `spec.instancetype` |
| **DenySecretDeleteIfUsedByVM** | Secret | A VM uses it for cloud-init (`cloudInitNoCloud`/`cloudInitConfigDrive`) or SSH access credentials |

## Deployment Order

```bash
# 1. Sync VirtualMachine resources into Gatekeeper cache
oc apply -f config-sync.yaml

# 2. Apply ConstraintTemplates (wait for them to be ready)
oc apply -f deny-nad-delete-template.yaml
oc apply -f deny-pvc-delete-template.yaml
oc apply -f deny-instancetype-delete-template.yaml
oc apply -f deny-secret-delete-template.yaml

# 3. Apply Constraints
oc apply -f deny-nad-delete-constraint.yaml
oc apply -f deny-pvc-delete-constraint.yaml
oc apply -f deny-instancetype-delete-constraint.yaml
oc apply -f deny-secret-delete-constraint.yaml
```

## Key Differences from Kyverno

| Aspect | Kyverno | Gatekeeper |
|---|---|---|
| **Resource lookup** | Live API calls via `apiCall` context | Cached resources via `data.inventory` (requires Config sync) |
| **Language** | JMESPath expressions | Rego (OPA policy language) |
| **Cluster-scoped resources** | Scoped API URL | Iterates all namespaces in `data.inventory.namespace[ns]` |
| **Trade-off** | Real-time data, higher API load | Cached data (slight delay), lower API load |

## Kyverno Equivalents

The original Kyverno policies are maintained at [michaelkotelnikov/nad-policy](https://github.com/michaelkotelnikov/nad-policy).
