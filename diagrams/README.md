# Diagrams Index

All diagrams in this repository, organised by abstraction level.

| Level | Description | File |
|-------|-------------|------|
| 1 — High | Entire Crossplane object model | [01-schema-overview/diagram-object-model.md](../01-schema-overview/diagram-object-model.md) |
| 2 — Middle | XRD top-level blocks | [02-xrd-anatomy/diagram-xrd-structure.md](../02-xrd-anatomy/diagram-xrd-structure.md) |
| 2 — Middle | Composition structure + data flow | [04-composition-anatomy/diagram-composition.md](../04-composition-anatomy/diagram-composition.md) |
| 3 — Low | XRD `versions[].schema.openAPIV3Schema` deep dive | [02-xrd-anatomy/diagram-xrd-spec-deep.md](../02-xrd-anatomy/diagram-xrd-spec-deep.md) |
| 3 — Low | OpenAPI v3 type system | [03-openapi-v3-primer/diagram-openapi-v3-types.md](../03-openapi-v3-primer/diagram-openapi-v3-types.md) |
| 3 — Low | Azure Managed Resource schema layout | [05-provider-azure/diagram-azure-mr-schema.md](../05-provider-azure/diagram-azure-mr-schema.md) |
| End-to-end | XRD → Composition → MR → Azure ARM | [05-provider-azure/diagram-crossplane-to-azure.md](../05-provider-azure/diagram-crossplane-to-azure.md) |

## Diagram formats

All diagrams use **Mermaid** syntax, which renders natively in:
- GitHub (`.md` files with mermaid code blocks)
- GitLab
- VS Code with the Mermaid extension
- Any Markdown renderer with Mermaid support

Sequence diagrams (in `diagram-composition.md`) use `sequenceDiagram` syntax.
All other diagrams use `graph TD` (top-down) or `graph LR` (left-right).
