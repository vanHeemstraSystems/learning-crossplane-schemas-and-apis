# 02 — XRD Anatomy: Every Field Explained

## The five top-level blocks

You were correct that an XRD consists of these YAML blocks. Here is the full picture:

```yaml
apiVersion: apiextensions.crossplane.io/v2     # (1) API contract version
kind: CompositeResourceDefinition              # (2) Object type
metadata:                                      # (3) Kubernetes identity
  name: <plural>.<group>
spec:                                          # (4) Crossplane configuration
  scope: Namespaced
  group: ...
  names: ...
  versions: ...
  # optional: defaultCompositionRef, connectionSecretKeys, etc.
status:                                        # (5) Controller-managed state
  conditions: ...
```

`status` is **written by Crossplane**, not by you. But you need to understand it for observability and debugging.

---

## (1) apiVersion

```yaml
apiVersion: apiextensions.crossplane.io/v2
```

- Use `v2` for all new XRDs (Crossplane v2+).
- `v1` still works for backward compatibility but defaults to legacy cluster-scoped mode.

---

## (2) kind

```yaml
kind: CompositeResourceDefinition
```

Always this value. Abbreviated as **XRD** in documentation and CLI (`kubectl get xrd`).

---

## (3) metadata

```yaml
metadata:
  name: <plural>.<group>     # REQUIRED — must match spec.names.plural + "." + spec.group
  annotations:               # optional
  labels:                    # optional
```

The `name` is a Kubernetes naming convention for CRDs: `<plural>.<group>`. Example: `storageaccounts.platform.example.org`.

---

## (4) spec — the main configuration block

See `diagram-xrd-spec-deep.md` for the full tree. Summary of all spec fields:

### 4a. scope (v2 — required)

```yaml
spec:
  scope: Namespaced    # or: Cluster
```

- `Namespaced`: XRs live in a namespace. **Default in v2.** No Claims needed.
- `Cluster`: XRs are cluster-wide. Can compose resources in any namespace.

### 4b. group

```yaml
spec:
  group: platform.example.org
```

The API group for your XR. Becomes part of the XR's `apiVersion`: `platform.example.org/v1alpha1`.

### 4c. names

```yaml
spec:
  names:
    kind: AzureStorageAccount      # CamelCase; what developers write in `kind:`
    plural: azurestorageaccounts   # lowercase plural; used in kubectl and metadata.name
    singular: azurestorageaccount  # optional; defaults to lowercase kind
    categories:                    # optional; lets kubectl get platform show this
      - platform
    shortNames:                    # optional; e.g. - asa
      - asa
```

### 4d. versions (array — one or more)

```yaml
spec:
  versions:
    - name: v1alpha1           # semantic version name
      served: true             # if false, API calls for this version are rejected
      referenceable: true      # exactly ONE version must be true; Compositions use this
      schema:                  # OpenAPI v3 structural schema
        openAPIV3Schema:
          type: object
          properties:
            spec:   { ... }   # what developers set
            status: { ... }   # what Crossplane/providers write back
      additionalPrinterColumns:  # optional; extra columns in kubectl output
        - name: LOCATION
          type: string
          jsonPath: .spec.parameters.location
      deprecated: false          # optional; warns users of this version
      deprecationWarning: ""     # optional; message shown when deprecated: true
```

**Important rules:**
- Exactly one version must have `referenceable: true`. This is the version Compositions reference.
- Multiple versions can have `served: true` simultaneously (for gradual migration).
- To deprecate: set `served: false` on the old version (users get errors).

### 4e. defaultCompositionRef (optional)

```yaml
spec:
  defaultCompositionRef:
    name: my-azure-composition
```

Default Composition when a developer does not specify one in their XR.

### 4f. enforcedCompositionRef (optional)

```yaml
spec:
  enforcedCompositionRef:
    name: my-azure-composition
```

Forces all XRs to use this Composition. Developers cannot override it.

### 4g. defaultCompositionUpdatePolicy (optional)

```yaml
spec:
  defaultCompositionUpdatePolicy: Automatic   # or: Manual
```

Controls whether XRs automatically pick up new Composition revisions.

### 4h. connectionSecretKeys (optional)

```yaml
spec:
  connectionSecretKeys:
    - connectionString
    - primaryAccessKey
```

Declares which connection secret keys can be propagated from composed resources up to the XR.

---

## (5) status — controller-managed

You do not author `status`. Crossplane writes it. Key fields:

```yaml
status:
  conditions:
    - type: Established          # XRD has been converted to a CRD in the cluster
      status: "True"
      reason: WatchingCompositeResource
      lastTransitionTime: "..."
    - type: Offered              # (v1 only) Claims are available
      status: "True"
```

Check XRD status with:
```bash
kubectl describe xrd <name>
kubectl get xrd <name> -o jsonpath='{.status.conditions}'
```

---

## Common mistakes

| Mistake | Symptom | Fix |
|---------|---------|-----|
| `metadata.name` does not match `<plural>.<group>` | XRD rejected | Set name to `azurestorageaccounts.platform.example.org` |
| No version has `referenceable: true` | Composition cannot bind | Add `referenceable: true` to exactly one version |
| `scope:` missing in v2 | Defaults to LegacyCluster | Always set `scope: Namespaced` or `scope: Cluster` |
| `spec.names.kind` starts with lowercase | Kubernetes rejects it | Use CamelCase: `AzureStorageAccount` |
| `required:` inside `openAPIV3Schema` placed at wrong level | Fields always optional | Place `required:` as sibling of `properties:` in same object |
