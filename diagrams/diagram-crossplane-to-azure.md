# Diagram: XRD → Composition → MR → Azure ARM (End-to-End)

This diagram shows **how schema knowledge flows** from the XRD definition right through to Azure ARM API calls.

```mermaid
graph LR
    subgraph "XRD Schema (authored by platform engineer)"
        XRD_SPEC["spec.versions[].schema\n.openAPIV3Schema\n\nspec:\n  parameters:\n    location: string  ← enum\n    sku: string       ← enum\n    tags: object\nstatus:\n  id: string\n  endpoint: string"]
    end

    subgraph "XR Instance (created by developer)"
        XR_INST["AzureStorageAccount\nplatform.example.org/v1alpha1\n\nspec:\n  parameters:\n    location: westeurope\n    sku: Standard_LRS\n    tags:\n      env: prod"]
    end

    subgraph "Composition Patch"
        PATCH["FromCompositeFieldPath\n\nspec.parameters.location\n  → spec.forProvider.location\n\nspec.parameters.sku\n  → spec.forProvider.accountReplicationType\n  (transform: Standard_LRS → LRS)\n\nstatus.atProvider.id\n  ← status.id"]
    end

    subgraph "Azure MR (created by Crossplane)"
        MR["Account\nstorage.azure.upbound.io/v1beta2\n\nspec:\n  forProvider:\n    location: westeurope\n    accountTier: Standard\n    accountReplicationType: LRS\n    resourceGroupNameRef:\n      name: my-rg\nstatus:\n  atProvider:\n    id: /subscriptions/.../mysa\n    primaryBlobEndpoint: https://..."]
    end

    subgraph "Azure ARM"
        ARM["Azure Storage Account\n\nresourceGroup: my-rg\nlocation: West Europe\nkind: StorageV2\nsku: Standard_LRS\nproperties:\n  primaryEndpoints:\n    blob: https://mysa.blob.core..."]
    end

    XRD_SPEC -->|"validates"| XR_INST
    XR_INST -->|"read by"| PATCH
    PATCH -->|"writes into"| MR
    MR -->|"Azure REST API\nPUT /subscriptions/..."| ARM
    ARM -->|"observed state"| MR
    MR -->|"ToCompositeFieldPath"| XR_INST

    style XRD_SPEC fill:#4A90D9,color:#fff
    style XR_INST fill:#27AE60,color:#fff
    style PATCH fill:#8E44AD,color:#fff
    style MR fill:#E67E22,color:#fff
    style ARM fill:#0078D4,color:#fff
```

---

## Field name mapping: XRD → Azure ARM

| XRD field (what developer sets) | Composition patch | MR forProvider field | Azure ARM property |
|--------------------------------|------------------|---------------------|-------------------|
| `spec.parameters.location` | direct copy | `spec.forProvider.location` | `location` |
| `spec.parameters.sku` | map transform | `spec.forProvider.accountReplicationType` | `sku.name` |
| `spec.parameters.tags` | direct copy | `spec.forProvider.tags` | `tags` |
| *(set in Composition base)* | — | `spec.forProvider.accountTier` | `sku.tier` |
| *(set in Composition base)* | — | `spec.forProvider.minTlsVersion` | `properties.minimumTlsVersion` |

---

## The schema "stack"

```
OpenAPI v3 Structural Schema (standard)
        │
        ▼
Kubernetes CRD structural schema (subset of OpenAPI v3)
        │
        ▼
Crossplane XRD openAPIV3Schema (same as CRD structural schema)
        │  validates
        ▼
Your XR instance (developer YAML)
        │  patches map to
        ▼
Azure MR spec.forProvider (from provider CRD — also OpenAPI v3)
        │  reconciled to
        ▼
Azure ARM REST API (OpenAPI 2.0 / Swagger internally)
```

The schema language is consistent top to bottom: everything is OpenAPI v3 (or a subset). The provider CRDs (Azure MR schemas) are generated directly from the Azure REST API specification.
