# learning-crossplane-schemas

A structured learning repository for understanding Crossplane v2 schemas — from the highest-level concept map down to the field-level OpenAPI v3 validation rules — with Azure as the provider. By the end you will be able to write any Crossplane YAML configuration file from scratch, grounded in schema and API knowledge.

---

## Repository structure

```
learning-crossplane-schemas/
│
├── README.md                          ← you are here
│
├── 01-schema-overview/
│   ├── README.md                      ← The Crossplane object model: what kinds exist
│   └── diagram-object-model.md        ← Level 1 diagram: entire Crossplane object model
│
├── 02-xrd-anatomy/
│   ├── README.md                      ← XRD field-by-field breakdown (Crossplane v2, Namespaced)
│   ├── diagram-xrd-structure.md       ← Level 2 diagram: XRD top-level blocks
│   └── diagram-xrd-spec-deep.md       ← Level 3 diagram: spec sub-fields drill-down
│
├── 03-openapi-v3-primer/
│   ├── README.md                      ← How OpenAPI v3 structural schemas work inside XRDs
│   └── diagram-openapi-v3-types.md    ← Level 3 diagram: OpenAPI v3 type system
│
├── 04-composition-anatomy/
│   ├── README.md                      ← Composition field-by-field breakdown
│   └── diagram-composition.md         ← Level 2 diagram: Composition structure
│
├── 05-provider-azure/
│   ├── README.md                      ← Azure provider overview and MR schema pattern
│   ├── diagram-azure-mr-schema.md     ← Level 3 diagram: Managed Resource schema layout
│   └── diagram-crossplane-to-azure.md ← How XRD → Composition → MR → Azure ARM maps
│
├── 06-worked-examples/
│   ├── README.md                      ← Walk-through: build from schema knowledge
│   ├── xrd-azure-resourcegroup.yaml   ← Minimal XRD: Azure Resource Group
│   ├── composition-azure-resourcegroup.yaml
│   ├── xrd-azure-storage-account.yaml ← Richer XRD with nested parameters
│   ├── composition-azure-storage-account.yaml
│   └── xr-instance-storage-account.yaml  ← The XR a developer would create
│
└── diagrams/
    └── README.md                      ← Index of all diagrams in the repo
```

---

## Learning path

Work through the sections in order. Each builds on the previous.

| Step | Section | What you learn |
|------|---------|----------------|
| 1 | `01-schema-overview` | The five Crossplane object types and how they relate |
| 2 | `02-xrd-anatomy` | Every field of a v2 XRD, from `apiVersion` to `status` |
| 3 | `03-openapi-v3-primer` | The OpenAPI v3 structural schema sub-language used inside XRDs |
| 4 | `04-composition-anatomy` | How a Composition references an XRD and creates real Azure resources |
| 5 | `05-provider-azure` | The Managed Resource (MR) schema pattern used by `provider-azure-upbound` |
| 6 | `06-worked-examples` | Put it all together: two complete, annotated Azure examples |

---

## Key facts about Crossplane v2 schemas

### Your intuition about top-level blocks is correct — with one important detail

A Crossplane **XRD** (the _definition_ object) has these top-level YAML blocks:

```
apiVersion: apiextensions.crossplane.io/v2
kind:       CompositeResourceDefinition
metadata:   ...           ← standard Kubernetes object metadata
spec:       ...           ← everything Crossplane cares about
status:     ...           ← set by Crossplane; you don't author this
```

`status` is managed by the controller and is not authored by you, but it is part of the schema and important to understand for observability.

### Crossplane v2 differences from v1

| Topic | v1 behaviour | v2 behaviour |
|-------|-------------|--------------|
| Default scope | Cluster (implicit) | `Namespaced` (explicit `scope:` field required) |
| Claims | Supported via `claimNames:` | **Removed** — no Claims in v2 |
| `apiVersion` | `apiextensions.crossplane.io/v1` | `apiextensions.crossplane.io/v2` |
| Backward compat | — | v2 is backward compatible: v1-style XRDs still work |

### The schema inside an XRD is OpenAPI v3 Structural Schema

Crossplane uses the same mechanism as Kubernetes CRDs: the `openAPIV3Schema` field follows the [OpenAPI v3 structural schema](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/#specifying-a-structural-schema) specification. This is a **subset** of full OpenAPI v3 — Kubernetes enforces "structural" constraints (no `$ref`, no `allOf` at top level, etc.).

---

## Quick-start: minimal working XRD (Azure Resource Group)

```yaml
apiVersion: apiextensions.crossplane.io/v2
kind: CompositeResourceDefinition
metadata:
  name: azureresourcegroups.platform.example.org   # must be plural.group
spec:
  scope: Namespaced
  group: platform.example.org
  names:
    kind: AzureResourceGroup         # the CamelCase name developers use
    plural: azureresourcegroups
  versions:
    - name: v1alpha1
      served: true
      referenceable: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                parameters:
                  type: object
                  properties:
                    location:
                      type: string
                      description: Azure region, e.g. westeurope
                      enum: [westeurope, northeurope, eastus, eastus2]
                  required:
                    - location
              required:
                - parameters
            status:
              type: object
              properties:
                resourceGroupName:
                  type: string
```

See `06-worked-examples/` for the matching Composition and developer XR instance.

---

## References

- [Crossplane v2 documentation](https://docs.crossplane.io/latest/)
- [What's new in Crossplane v2](https://docs.crossplane.io/latest/whats-new/)
- [XRD reference](https://docs.crossplane.io/latest/composition/composite-resource-definitions/)
- [Composition reference](https://docs.crossplane.io/latest/composition/compositions/)
- [Upbound provider-azure-upbound](https://marketplace.upbound.io/providers/upbound/provider-azure-upbound)
- [OpenAPI v3 structural schema (Kubernetes docs)](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/#specifying-a-structural-schema)
- [OpenAPI Specification v3.0](https://swagger.io/specification/)
