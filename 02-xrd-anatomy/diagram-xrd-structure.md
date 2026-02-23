# Diagram: XRD Structure (Level 2 — Middle)

This diagram shows the **five top-level blocks** of an XRD and the key fields inside each.

```mermaid
graph LR
    XRD["CompositeResourceDefinition"]

    AV["apiVersion\napiextensions.crossplane.io/v2"]
    K["kind\nCompositeResourceDefinition"]

    META["metadata\n─────────────\nname: plural.group\nannotations: …\nlabels: …"]

    SPEC["spec\n─────────────\nscope: Namespaced | Cluster\ngroup: platform.example.org\nnames: { kind, plural … }\nversions: [ … ]\ndefaultCompositionRef: …\nenforcedCompositionRef: …\nconnectionSecretKeys: …"]

    STATUS["status  ← written by Crossplane\n─────────────\nconditions:\n  - type: Established\n    status: True\n    reason: WatchingCompositeResource"]

    XRD --- AV
    XRD --- K
    XRD --- META
    XRD --- SPEC
    XRD --- STATUS

    style XRD fill:#4A90D9,color:#fff,font-weight:bold
    style AV fill:#EBF5FB
    style K fill:#EBF5FB
    style META fill:#EBF5FB
    style SPEC fill:#D5F5E3
    style STATUS fill:#FDEBD0
```

---

## Expanded: spec fields

```mermaid
graph TD
    SPEC["spec"]

    SCOPE["scope\nNamespaced  ←  v2 default\nCluster"]
    GROUP["group\nplatform.example.org"]
    NAMES["names\n────────────\nkind: AzureStorageAccount\nplural: azurestorageaccounts\nsingular: azurestorageaccount\ncategories: [platform]\nshortNames: [asa]"]
    VERS["versions  (array)\n────────────\n- name: v1alpha1\n  served: true\n  referenceable: true\n  schema:\n    openAPIV3Schema: { … }"]
    DCR["defaultCompositionRef\nname: my-composition"]
    ECR["enforcedCompositionRef\nname: my-composition"]
    DCUP["defaultCompositionUpdatePolicy\nAutomatic | Manual"]
    CSK["connectionSecretKeys\n- connectionString\n- primaryAccessKey"]

    SPEC --> SCOPE
    SPEC --> GROUP
    SPEC --> NAMES
    SPEC --> VERS
    SPEC --> DCR
    SPEC --> ECR
    SPEC --> DCUP
    SPEC --> CSK

    style SPEC fill:#4A90D9,color:#fff
    style VERS fill:#D5F5E3,font-weight:bold
```

The `versions` array is where the real schema work happens — it contains the `openAPIV3Schema`. See the next diagram for a deep-dive into that.
