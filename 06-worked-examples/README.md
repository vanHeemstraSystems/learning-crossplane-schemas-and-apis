# 06 — Worked Examples

Two complete Azure examples, each with:
1. The XRD (schema definition)
2. The Composition (implementation)
3. An XR instance (what the developer creates)

---

## Example A: Azure Resource Group (minimal)

Files:
- `xrd-azure-resourcegroup.yaml` — XRD
- `composition-azure-resourcegroup.yaml` — Composition
- *(no XR instance — see Example B for that)*

This is the simplest possible real example. One XRD, one Composition, one Azure Managed Resource.

---

## Example B: Azure Storage Account (realistic)

Files:
- `xrd-azure-storage-account.yaml` — XRD with nested parameters, enums, defaults, status fields
- `composition-azure-storage-account.yaml` — Composition with multiple resources (RG + Storage Account), patches, transforms
- `xr-instance-storage-account.yaml` — What a developer actually applies to the cluster

---

## How to use these examples

### Apply to a cluster (assuming Crossplane v2 + Azure provider installed)

```bash
# 1. Install providers
kubectl apply -f https://marketplace.upbound.io/providers/upbound/provider-azure-azure/...

# 2. Configure credentials (create the secret first)
kubectl create secret generic azure-secret \
  --from-file=creds=./azure-creds.json \
  -n crossplane-system

# 3. Apply ProviderConfig
kubectl apply -f - <<EOF
apiVersion: azure.upbound.io/v1beta1
kind: ProviderConfig
metadata:
  name: azure-provider-config
spec:
  subscriptionID: "YOUR-SUBSCRIPTION-ID"
  credentials:
    source: Secret
    secretRef:
      namespace: crossplane-system
      name: azure-secret
      key: creds
EOF

# 4. Apply XRD
kubectl apply -f 06-worked-examples/xrd-azure-storage-account.yaml

# 5. Apply Composition
kubectl apply -f 06-worked-examples/composition-azure-storage-account.yaml

# 6. Developer creates XR
kubectl apply -f 06-worked-examples/xr-instance-storage-account.yaml

# 7. Watch it reconcile
kubectl get azurestorageaccount
kubectl get managed
```

---

## Building from scratch: the mental checklist

Use this checklist when authoring any new XRD + Composition from scratch:

### XRD checklist
- [ ] `apiVersion: apiextensions.crossplane.io/v2`
- [ ] `metadata.name` = `<plural>.<group>`
- [ ] `spec.scope: Namespaced` (or Cluster)
- [ ] `spec.group` = your platform API group
- [ ] `spec.names.kind` in CamelCase, `plural` in lowercase
- [ ] At least one version with `served: true` AND `referenceable: true`
- [ ] `openAPIV3Schema.type: object` at root
- [ ] `spec:` defined under `properties:` with `type: object`
- [ ] All parameter fields have `type:` specified
- [ ] `required:` lists placed as siblings of `properties:`, not inside them
- [ ] `status:` defined under `properties:` to document expected outputs

### Composition checklist
- [ ] `compositeTypeRef.apiVersion` matches XRD group + referenceable version
- [ ] `compositeTypeRef.kind` matches XRD `spec.names.kind`
- [ ] `mode: Pipeline`
- [ ] `function-patch-and-transform` step defined
- [ ] Each composed resource has `name:` (logical), `base:` (MR template), `patches:`
- [ ] `providerConfigRef.name` set in base
- [ ] All XR → MR patches use correct `fromFieldPath` (XR paths)
- [ ] All MR → XR status patches use `ToCompositeFieldPath` with `status.atProvider.*` sources
- [ ] `function-auto-ready` step added as last step
