# Diagram: OpenAPI v3 Structural Schema Type System (Level 3)

```mermaid
graph TD
    ROOT["OpenAPI v3 Schema Object\n(used in openAPIV3Schema)"]

    ROOT --> SCALAR["Scalar Types"]
    ROOT --> COLLECTION["Collection Types"]
    ROOT --> COMMON["Common Keywords\n(apply to all types)"]

    SCALAR --> STR["string\n────────────────\nformat: date | date-time\n         password | byte\n         uri | hostname\nenum: [val1, val2]\npattern: regex\nminLength / maxLength\ndefault: 'value'"]

    SCALAR --> INT["integer\n────────────────\nformat: int32 | int64\nminimum / maximum\nexclusiveMin / exclusiveMax\nmultipleOf\ndefault: 0"]

    SCALAR --> NUM["number\n────────────────\nformat: float | double\nminimum / maximum\ndefault: 0.0"]

    SCALAR --> BOOL["boolean\n────────────────\ndefault: true | false"]

    COLLECTION --> OBJ["object\n────────────────\nproperties:\n  fieldName: {schema}\nadditionalProperties:\n  false (closed)\n  {schema} (typed map)\nrequired: [field1, field2]\nminProperties / maxProperties\nx-kubernetes-preserve-unknown-fields: true"]

    COLLECTION --> ARR["array\n────────────────\nitems: {schema}\nminItems / maxItems\nuniqueItems: true"]

    COMMON --> DESC["description:\n  Human-readable docs"]
    COMMON --> NULL["nullable: true\n  (alongside type)"]
    COMMON --> XKINT["x-kubernetes-int-or-string: true\n  Allows int or string value"]
    COMMON --> XKPUF["x-kubernetes-preserve-unknown-fields: true\n  Accept any subfields\n  (escape hatch for opaque objects)"]
    COMMON --> XKVAL["x-kubernetes-validations:\n  (CEL rules — advanced)\n  - rule: self.minReplicas <= self.maxReplicas\n    message: min must be <= max"]

    style ROOT fill:#4A90D9,color:#fff,font-weight:bold
    style SCALAR fill:#27AE60,color:#fff
    style COLLECTION fill:#E67E22,color:#fff
    style COMMON fill:#8E44AD,color:#fff
    style STR fill:#D5F5E3
    style INT fill:#D5F5E3
    style NUM fill:#D5F5E3
    style BOOL fill:#D5F5E3
    style OBJ fill:#FDEBD0
    style ARR fill:#FDEBD0
```

---

## Quick reference: type + validation combinations

```
string  ──► enum          → validated list of allowed values
string  ──► pattern       → regex validation
string  ──► format        → semantic hint (date-time, uri, etc.)
string  ──► min/maxLength → length bounds
integer ──► minimum       → number lower bound (inclusive)
integer ──► maximum       → number upper bound (inclusive)
object  ──► properties    → typed known fields
object  ──► additionalProperties: {type: string} → string map (tags pattern)
object  ──► required      → mandatory field list (sibling to properties)
array   ──► items         → element schema
array   ──► minItems      → require at least N elements
```

---

## Structural schema constraint visualised

```mermaid
graph LR
    FULL["Full OpenAPI v3"]
    STRUCT["Kubernetes Structural Schema\n(subset used in XRDs)"]
    CROSS["Crossplane openAPIV3Schema"]

    FULL -->|"removes: $ref, allOf/anyOf/oneOf\nat root, free-form objects"| STRUCT
    STRUCT -->|"same — Crossplane\ndoes not add further constraints"| CROSS

    style FULL fill:#E74C3C,color:#fff
    style STRUCT fill:#E67E22,color:#fff
    style CROSS fill:#27AE60,color:#fff
```

### What structural schema removes from full OpenAPI v3

| Removed | Reason |
|---------|--------|
| `$ref` to external schemas | No remote schema resolution |
| `allOf` / `anyOf` / `oneOf` replacing `properties` | Must always have explicit properties at each level |
| Objects without `type` | Every node must be explicitly typed |
| Free-form JSON (without `x-kubernetes-preserve-unknown-fields`) | Kubernetes prunes unknown fields |
