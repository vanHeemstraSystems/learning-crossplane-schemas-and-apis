# Diagram: Crossplane Object Model (Level 1 — High)

This diagram shows the **entire Crossplane object model** and how the five main kinds relate to each other and to Azure.

```mermaid
graph TD
    subgraph "Platform Engineer authors"
        XRD["CompositeResourceDefinition (XRD)\napiextensions.crossplane.io/v2\n\nDefines the custom API schema\nusing OpenAPI v3 Structural Schema"]
        COMP["Composition\napiextensions.crossplane.io/v2\n\nDefines the pipeline of\ncomposition functions"]
        PC["ProviderConfig\npkg.crossplane.io/v1\n\nAzure credentials &\nsubscription config"]
    end

    subgraph "Developer authors"
        XR["Composite Resource (XR)\nyour-group/v1alpha1\n\nInstance of the custom API\ne.g. AzureStorageAccount"]
    end

    subgraph "Crossplane creates (via Composition)"
        MR1["Managed Resource\ne.g. ResourceGroup\nazure.upbound.io/v1beta1"]
        MR2["Managed Resource\ne.g. StorageAccount\nazure.upbound.io/v1beta1"]
    end

    subgraph "Azure"
        ARM1["Azure Resource Group"]
        ARM2["Azure Storage Account"]
    end

    XRD -- "defines schema for" --> XR
    XRD -- "Composition references XRD\nvia compositeTypeRef" --> COMP
    XR -- "triggers pipeline in" --> COMP
    COMP -- "creates" --> MR1
    COMP -- "creates" --> MR2
    MR1 -- "reconciles to" --> ARM1
    MR2 -- "reconciles to" --> ARM2
    PC -- "credentials used by" --> MR1
    PC -- "credentials used by" --> MR2

    style XRD fill:#4A90D9,color:#fff
    style COMP fill:#4A90D9,color:#fff
    style XR fill:#27AE60,color:#fff
    style MR1 fill:#E67E22,color:#fff
    style MR2 fill:#E67E22,color:#fff
    style ARM1 fill:#0078D4,color:#fff
    style ARM2 fill:#0078D4,color:#fff
    style PC fill:#8E44AD,color:#fff
```

## Key points from this diagram

- **Blue (Platform)**: Objects the platform engineer writes. The XRD is the schema contract; the Composition is the implementation.
- **Green (Developer)**: The XR is what developers create. Its structure is defined by the XRD schema.
- **Orange (Managed Resources)**: Created automatically by Crossplane. The developer never writes these directly.
- **Azure Blue**: The real cloud resources. Each MR maps 1-to-1 to an Azure resource.
- The `compositeTypeRef` in a Composition binds it to an XRD — this is the critical link between schema and implementation.
