# 04 — Composition Anatomy

A Composition is the implementation side of the custom API defined by the XRD. Where the XRD says **what** a developer can request, the Composition says **how** Crossplane fulfils that request.

## Top-level structure

```yaml
apiVersion: apiextensions.crossplane.io/v2
kind: Composition
metadata:
  name: azure-storage-account-composition
  labels:
    provider: azure
    environment: production
spec:
  compositeTypeRef:        # ← binds this Composition to an XRD
    apiVersion: platform.example.org/v1alpha1
    kind: AzureStorageAccount
  mode: Pipeline           # always Pipeline in v2
  pipeline:                # list of composition function steps
    - step: patch-and-transform
      functionRef:
        name: function-patch-and-transform
      input: ...
    - step: auto-ready
      functionRef:
        name: function-auto-ready
```

## Key spec fields

### compositeTypeRef

This is the critical binding: it names the **exact apiVersion and kind** of the XR this Composition serves.

```yaml
spec:
  compositeTypeRef:
    apiVersion: platform.example.org/v1alpha1   # group/version from XRD
    kind: AzureStorageAccount                   # kind from XRD spec.names.kind
```

If multiple Compositions share the same `compositeTypeRef`, developers select one via `compositionRef.name` on their XR, or the XRD's `defaultCompositionRef` applies.

### mode

```yaml
spec:
  mode: Pipeline    # only mode in Crossplane v2 (Resources mode is removed)
```

### pipeline (array of steps)

Each step calls a **composition function**. The most common functions:

| Function | Helm chart name | Purpose |
|----------|----------------|---------|
| `function-patch-and-transform` | crossplane-contrib/function-patch-and-transform | Template resources with patches |
| `function-auto-ready` | crossplane-contrib/function-auto-ready | Mark XR ready when all composed resources are ready |
| `function-kcl` | crossplane-contrib/function-kcl | KCL language templating |
| `function-go-templating` | crossplane-contrib/function-go-templating | Go template rendering |

### writeConnectionSecretsToNamespace (optional)

```yaml
spec:
  writeConnectionSecretsToNamespace: crossplane-system
```

Only relevant for cluster-scoped XRs. Namespaced XRs use the XR's own namespace.

---

## function-patch-and-transform input

The most common function. Its `input` block defines:
1. The composed resources to create (with their full Azure MR specs)
2. Patches that copy values from the XR into those resources

```yaml
pipeline:
  - step: patch-and-transform
    functionRef:
      name: function-patch-and-transform
    input:
      apiVersion: pt.fn.crossplane.io/v1beta1
      kind: Resources
      resources:
        - name: resource-group          # logical name for this composed resource
          base:                         # the Azure MR template
            apiVersion: azure.upbound.io/v1beta1
            kind: ResourceGroup
            spec:
              forProvider:
                location: westeurope   # will be overridden by patch
              providerConfigRef:
                name: azure-provider-config
          patches:
            - type: FromCompositeFieldPath
              fromFieldPath: spec.parameters.location
              toFieldPath: spec.forProvider.location
            - type: ToCompositeFieldPath
              fromFieldPath: metadata.name
              toFieldPath: status.resourceGroupName
```

### Patch types

| Patch type | Direction | Use |
|-----------|-----------|-----|
| `FromCompositeFieldPath` | XR → MR | Copy developer input to the MR |
| `ToCompositeFieldPath` | MR → XR | Write MR output back to XR status |
| `CombineFromComposite` | XR → MR | Combine multiple XR fields into one MR field |
| `PatchSet` | — | Reuse a named set of patches across resources |

### Transforms

Patches can include transforms to convert values:

```yaml
patches:
  - type: FromCompositeFieldPath
    fromFieldPath: spec.parameters.sku
    toFieldPath: spec.forProvider.accountTier
    transforms:
      - type: map
        map:
          Standard_LRS: Standard
          Premium_LRS: Premium
```
