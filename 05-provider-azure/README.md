# 05 — Provider Azure: Managed Resource Schema Pattern

## Which provider to use

Use **Upbound's official Azure provider** (not the community one). It is split into sub-providers by service family:

| Sub-provider | API group prefix | Covers |
|-------------|-----------------|--------|
| `provider-azure-azure` | `azure.upbound.io` | Resource groups, subscriptions |
| `provider-azure-storage` | `storage.azure.upbound.io` | Storage accounts, containers, blobs |
| `provider-azure-network` | `network.azure.upbound.io` | VNets, subnets, NSGs, PIPs |
| `provider-azure-compute` | `compute.azure.upbound.io` | VMs, disks, scale sets |
| `provider-azure-keyvault` | `keyvault.azure.upbound.io` | Key Vaults, secrets, keys |
| `provider-azure-containerservice` | `containerservice.azure.upbound.io` | AKS clusters |
| `provider-azure-sql` | `sql.azure.upbound.io` | Azure SQL, managed instances |

Install via a `Provider` object:
```yaml
apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
  name: provider-azure-storage
spec:
  package: xpkg.upbound.io/upbound/provider-azure-storage:v1.x.x
```

## ProviderConfig — Azure credentials

```yaml
apiVersion: azure.upbound.io/v1beta1
kind: ProviderConfig
metadata:
  name: azure-provider-config
spec:
  subscriptionID: "00000000-0000-0000-0000-000000000000"
  credentials:
    source: Secret
    secretRef:
      namespace: crossplane-system
      name: azure-secret
      key: creds
```

The secret contains a JSON service principal:
```json
{
  "clientId": "...",
  "clientSecret": "...",
  "subscriptionId": "...",
  "tenantId": "...",
  "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
  "resourceManagerEndpointUrl": "https://management.azure.com/",
  "activeDirectoryGraphResourceId": "https://graph.windows.net/",
  "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
  "galleryEndpointUrl": "https://gallery.azure.com/",
  "managementEndpointUrl": "https://management.core.windows.net/"
}
```

---

## Managed Resource schema pattern

Every Azure MR follows the same schema pattern. Understanding this lets you read any MR CRD.

```yaml
apiVersion: <service>.azure.upbound.io/v1beta1   # or v1beta2
kind: <ResourceKind>
metadata:
  name: <unique-name>
  annotations:
    crossplane.io/external-name: <azure-resource-name>  # optional: pin exact Azure name
spec:
  forProvider:           # ← everything Azure needs to create/manage the resource
    location: westeurope
    resourceGroupName: my-rg
    # ... resource-specific fields ...
    tags:
      environment: production
  initProvider:          # ← fields set only at creation, never updated
    adminPassword: ...
  providerConfigRef:     # ← which ProviderConfig to use
    name: azure-provider-config
  managementPolicies:    # ← default: [*] = full lifecycle management
    - Observe
    - Create
    - Update
    - Delete
  deletionPolicy: Delete  # or: Orphan (leave Azure resource when MR deleted)
  writeConnectionSecretToRef:
    name: my-resource-connection
    namespace: crossplane-system
status:
  atProvider:            # ← what Azure reports back (read-only)
    id: /subscriptions/.../resourceGroups/...
    provisioningState: Succeeded
  conditions:
    - type: Ready
      status: "True"
    - type: Synced
      status: "True"
```

---

## MR spec.forProvider vs status.atProvider

| Field area | Who writes it | Content |
|-----------|-------------|---------|
| `spec.forProvider` | You (via Composition patches) | Desired state: what you want Azure to create |
| `spec.initProvider` | You (via Composition patches) | One-time init fields (passwords, keys) |
| `status.atProvider` | Azure provider controller | Observed state from Azure ARM API |
| `status.conditions` | Crossplane controller | Ready, Synced conditions |

The patch `ToCompositeFieldPath` reads from `status.atProvider.*` to pass outputs back to the XR.
