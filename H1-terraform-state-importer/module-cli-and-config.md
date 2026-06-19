# Module: CLI + configuration (`cmd/` + YAML config)

| Field | Value |
|-------|-------|
| Repository | `Azure/terraform-state-importer` |
| Flavor | Go (Cobra + Viper) |
| Key files | `main.go`, `cmd/root.go`, `cmd/run.go`, `.config/alz.*.yaml` |
| Source URL | <https://github.com/Azure/terraform-state-importer/blob/main/cmd/run.go> |
| Mode | deep (source-verified) |
| Last reviewed | 2026-06-17 |

## Purpose

The command surface and the YAML configuration schema. `cmd/` is a **Cobra** CLI bound to **Viper**; the single
`run` subcommand parses the config into typed structs, composes the per-concern clients, and invokes the mapping
engine.

## Command structure

```
terraform-state-importer [global-flags] run [command-flags]
```

`main.go` → `cmd.Execute()` → `rootCmd` (root.go) → `runCmd` (run.go). There is **one** subcommand: `run`.

### Global flags (`cmd/root.go`)

| Flag | Short | Default | Meaning |
|------|-------|---------|---------|
| `--config` | | `$HOME/.terraform-state-importer.yaml` | Path to the YAML config |
| `--verbosity` | `-v` | `info` | Log level (`trace`/`debug`/`info`/`warn`/`error`/`fatal`/`panic`) |
| `--structuredLogs` | | `false` | Emit JSON logs (logrus `JSONFormatter`) |
| `--help` | `-h` | | Help |

### Run flags (`cmd/run.go`)

| Flag | Short | Default | Meaning |
|------|-------|---------|---------|
| `--terraformModulePath` | `-t` | `.` | The Terraform module to import into |
| `--workingFolderPath` | `-w` | `.` | Where temp files + outputs are written |
| `--issuesCsv` | `-c` | `""` | Resolved-issues CSV → **switches to import-generation mode** (empty = analysis mode) |
| `--planAsTextOnly` | `-p` | `false` | Just produce a text `terraform plan` (early exit) |
| `--planSubscriptionID` | `-s` | (az cli default) | Override the subscription for the plan |
| `--skipInitPlanShow` | `-x` | `false` | Skip `init`/`plan`/`show` (debug) |
| `--skipInitOnly` | `-k` | `false` | Skip only `terraform init` |
| `--skipInitUpgrade` | `-u` | `false` | Drop the `-upgrade` flag on `init` |

> **The `--issuesCsv` toggle is central:** `analyzer.NewMappingClient(..., issuesCsv != "", ...)` — passing a
> CSV flips the engine from **analysis** (discover + write issues) to **import generation** (read resolutions +
> write `imports.tf`).

## Client composition (`run.go`)

After parsing the config, `run.go` wires the clients (dependency injection), then calls `Map()`:

```go
resourceGraphClient := azure.NewResourceGraphClient(cloud, managementGroupIDs, subscriptionIDs,
    ignoreResourceIDPatterns, resourceGraphQueries, log)
jsonClient  := json.NewJsonClient(workingFolderPath, log)
planClient  := terraform.NewPlanClient(terraformModulePath, workingFolderPath, planSubscriptionID,
    ignoreResourceTypePatterns, skipInitPlanShow, skipInitOnly, skipInitUpgrade,
    propertyMappings, nameFormats, jsonClient, log)

if planAsTextOnly { planClient.PlanAsText(); return }   // early exit

issueCsvClient := csv.NewIssueCsvClient(workingFolderPath, issuesCsv, log)
hclClient      := hcl.NewHclClient(terraformModulePath, deleteCommands, log)
mappingClient  := analyzer.NewMappingClient(workingFolderPath, issuesCsv != "",
    resourceGraphClient, planClient, issueCsvClient, jsonClient, hclClient, log)
mappingClient.Map()
```

## Configuration YAML schema

| Section | Type | Purpose |
|---------|------|---------|
| `cloud` | string | `AzurePublic` (default) / `AzureUSGovernment` / `AzureChina` — selects ARM endpoints |
| `subscriptionIDs` **xor** `managementGroupIDs` | `[]string` | the Azure scope to discover |
| `ignoreResourceIDPatterns` | `[]regex` | Azure resource **IDs** to skip (e.g. all policy assignments, NetworkWatcherRG) |
| `ignoreResourceTypePatterns` | `[]regex` | Terraform resource **types/addresses** to skip (e.g. `random_uuid.telemetry`, `modtm`, `time_sleep`, `module.management_groups.*`) |
| `resourceGraphQueries` | `[]{name, scope, query}` | KQL discovery queries (`scope` = `Subscription`/`ManagementGroup`) |
| `nameFormats` | `[]{type, subType?, nameFormat, nameMatchType, nameFormatArguments}` | custom name-mapping rules |
| `propertyMappings` | `[]{type, subType?, mappings[…]}` | cross-resource property lookups |
| `deleteCommands` | `[]{type, command}` | cleanup commands (`%s` = resource ID) for `Destroy` actions |

### Resource Graph queries

Each query is KQL and **must** `project id, name, type, location, subscriptionId, resourceGroup`. Complex
shapes (subnets, peerings) are flattened with `mv-expand`:

```yaml
resourceGraphQueries:
  - name: "Virtual Network Subnets"
    scope: "Subscription"
    query: |
      resources
      | where type == "microsoft.network/virtualnetworks"
      | project id, name, type, location, subscriptionId, resourceGroup, subnets = properties.subnets
      | mv-expand subnets
      | project name = tostring(subnets.name), id = tostring(subnets.id),
                type = tostring(subnets.type), location, subscriptionId, resourceGroup
```

### Name formats

For resources whose match key isn't a plain `name` (composite names, ID-pattern matches):

```yaml
nameFormats:
  - type: "Microsoft.Authorization/policyAssignments"
    nameFormat: "%s/providers/Microsoft.Authorization/policyAssignments/%s"
    nameMatchType: "IDEndsWith"          # Exact | IDEndsWith | IDContains
    nameFormatArguments: ["parent_id", "name"]
```

### Property mappings + meta properties

`propertyMappings` resolve a value from a **related** resource in the plan (e.g. a private-DNS-zone name from a
VNet link) using regex `replacements` over a `meta.*` lookup. The tool auto-injects `meta.*` during plan
processing:

| Meta property | Meaning |
|---------------|---------|
| `meta.type` | Terraform resource type (`azurerm_resource_group`) |
| `meta.name` | resource name in config (`this`) |
| `meta.address` | full TF address incl. modules (`module.network.azurerm_virtual_network.main`) |
| `meta.location` | Azure region |
| `meta.subtype` | Azure type for `azapi_resource` (`Microsoft.Network/privateDnsZones/virtualNetworkLinks`) |
| `meta.apiversion` | Azure API version for `azapi_resource` |

> `meta.*` are read-only and can be used as `nameFormatArguments` or in `propertyMappings` source lookups.

## Pre-built ALZ configs

`.config/alz.management-groups.config.yaml`, `.config/alz.connectivity.hub-and-spoke.config.yaml`,
`.config/alz.connectivity.virtual-wan.config.yaml` — copy, set your subscription/MG ids, adjust filters, set
`cloud` for sovereign clouds.

## Dependencies

**Upstream:** Cobra/Viper, the Azure CLI (auth + default subscription), Terraform, the Azure Resource Graph API.
**Downstream:** the parsed config + clients feed `analyzer.MappingClient.Map()`.

## Notes & Gotchas

- **Viper key casing** — `run.go` reads nested keys lower-cased (`targetproperties`, `sourcelookupproperties`,
  `nameformatarguments`); the YAML uses camelCase but Viper normalises.
- **`--planAsTextOnly` short-circuits** before any mapping (useful to sanity-check the module plans).
- **Skip flags are for iteration/debugging** — once `init` has run you can `-k`/`-x` to speed up repeated runs.
- **Sovereign clouds** — set `cloud` *and* `az cloud set` before running so endpoints + auth match.

## Open Questions

- [ ] `TODO: verify` the exact Viper precedence (flag vs config vs env) — `BindPFlag` is used, env binding not confirmed.
