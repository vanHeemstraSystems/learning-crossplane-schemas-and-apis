# 01 — Schema Overview: The Crossplane Object Model

## The five kinds you need to know

Crossplane adds five main object kinds to a Kubernetes cluster. Understanding how they relate is the foundation for everything else.

| Kind | API group | Who authors it | Purpose |
|------|-----------|---------------|---------|
| `CompositeResourceDefinition` (XRD) | `apiextensions.crossplane.io/v2` | Platform engineer | Declares a custom API (schema) |
| `Composition` | `apiextensions.crossplane.io/v1` | Platform engineer | Describes _how_ to fulfil the API |
| `<YourKind>` (XR) | your custom group | Developer | Instantiates the custom API |
| `<Provider>` Managed Resource (MR) | e.g. `azure.upbound.io/v1beta1` | Crossplane (via Composition) | Maps 1-to-1 to a real cloud resource |
| `Provider` / `ProviderConfig` | `pkg.crossplane.io/v1` | Platform engineer | Installs and configures the provider |

## See diagram

→ [`diagram-object-model.md`](./diagram-object-model.md)

## How they connect

```
Developer creates XR
        │
        ▼
Crossplane reads matching Composition
        │
        ▼
Composition pipeline creates Managed Resources
        │
        ▼
Provider reconciles each MR → real Azure resource
```

The XRD is the **contract** (schema). The Composition is the **implementation**. The XR is the **request**. The MR is the **real thing**.

## What the XRD schema controls

The XRD schema (using OpenAPI v3) controls:
- What fields developers can set on their XR
- Type and validation of each field (string, integer, enum, pattern…)
- Which fields are required vs optional
- What fields appear in `status` (outputs from the composed resources)

The XRD does **not** control:
- Which Azure resources get created (that is the Composition's job)
- Azure-specific field names and types (those come from the provider CRDs)
