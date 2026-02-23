# 03 — OpenAPI v3 Structural Schema Primer

## What OpenAPI version does Crossplane use?

Crossplane uses **OpenAPI v3.0** (specifically the Kubernetes "structural schema" profile of it). The `openAPIV3Schema` field in every XRD `versions[]` entry is an OpenAPI v3.0 Schema Object, constrained by Kubernetes' structural schema rules.

Reference: [Kubernetes structural schema docs](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/#specifying-a-structural-schema)

---

## The type system

Everything in an OpenAPI v3 structural schema has a `type`. The six scalar types and two collection types:

| Type | YAML example | Notes |
|------|-------------|-------|
| `string` | `type: string` | Can have `format`, `enum`, `pattern`, `minLength`, `maxLength` |
| `integer` | `type: integer` | Can have `format: int32` or `int64`, `minimum`, `maximum` |
| `number` | `type: number` | Float. `format: float` or `double` |
| `boolean` | `type: boolean` | `true` or `false` |
| `object` | `type: object` | Has `properties:` and/or `additionalProperties:` |
| `array` | `type: array` | Has `items:` defining the element schema |

See `diagram-openapi-v3-types.md` for the visual type system map.

---

## String formats and validations

```yaml
# Plain string
location:
  type: string

# Enum (validated server-side)
location:
  type: string
  enum:
    - westeurope
    - northeurope
    - eastus
    - eastus2

# Pattern (regex)
storageAccountName:
  type: string
  pattern: '^[a-z0-9]{3,24}$'
  description: Lowercase alphanumeric, 3-24 chars

# With default
sku:
  type: string
  default: Standard_LRS

# Date-time format
createdAt:
  type: string
  format: date-time
```

---

## Object types

```yaml
# Typed object — all known fields
tags:
  type: object
  properties:
    environment:
      type: string
    owner:
      type: string

# Map of string-to-string (for arbitrary tags)
tags:
  type: object
  additionalProperties:
    type: string

# Object with some known fields AND arbitrary extras
metadata:
  type: object
  properties:
    owner:
      type: string
  additionalProperties:
    type: string

# Opaque blob — accept any structure
rawConfig:
  type: object
  x-kubernetes-preserve-unknown-fields: true
```

---

## Array types

```yaml
# Array of strings
allowedIPs:
  type: array
  items:
    type: string

# Array of objects
subnets:
  type: array
  items:
    type: object
    properties:
      name:
        type: string
      addressPrefix:
        type: string
    required:
      - name
      - addressPrefix
  minItems: 1
  maxItems: 10
```

---

## The `required` keyword — placement matters

`required` is **per-object** and is a sibling of `properties`, not inside it:

```yaml
spec:
  type: object
  properties:
    parameters:
      type: object
      properties:
        location:
          type: string        # this field is required (see required list below)
        sku:
          type: string        # this field is optional
      required:               # <-- required for the parameters object
        - location
    writeConnectionSecretToRef:
      type: object
      properties:
        name:
          type: string
  required:                   # <-- required for the spec object
    - parameters
```

---

## Defaults

OpenAPI v3 structural schemas support `default` at any level:

```yaml
sku:
  type: string
  default: Standard_LRS

replicationCount:
  type: integer
  default: 3

enableHttps:
  type: boolean
  default: true
```

Kubernetes applies defaults server-side when an XR is created without those fields.

---

## Integer and number bounds

```yaml
replicaCount:
  type: integer
  minimum: 1
  maximum: 10
  default: 2

storageGB:
  type: integer
  format: int32
  minimum: 1
  maximum: 65536
```

---

## Connection secrets in the schema

When using `connectionSecretKeys` in the XRD spec, you should add the matching field to the spec schema to allow developers to name the secret:

```yaml
spec:
  type: object
  properties:
    writeConnectionSecretToRef:
      type: object
      description: Write connection details to this Kubernetes Secret
      properties:
        name:
          type: string
      required:
        - name
    parameters:
      type: object
      ...
```

For namespaced XRs, the secret lands in the same namespace as the XR automatically.
