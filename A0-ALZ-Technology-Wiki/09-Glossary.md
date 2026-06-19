# 9. Glossary

[← Back to index](./README.md)

Acronyms and terms used across ALZ, collected from the source wiki and standard CAF/ALZ usage.

## Core acronyms

| Term | Full name / meaning |
|---|---|
| **ALZ** | Azure Landing Zones |
| **CAF** | Cloud Adoption Framework (the methodology ALZ implements; *Ready* phase) |
| **AAC** | Azure Architecture Center (documentation home for implementation guides) |
| **RI** | Reference Implementation — deployable code: **Portal Accelerator, ALZ Bicep, ALZ Terraform** |
| **LZA** | Landing Zone Accelerator — a scenario-specific accelerator built on ALZ principles |
| **TF** | Terraform |
| **IaC** | Infrastructure as Code |
| **PR** | Pull Request |
| **MG** | Management Group |
| **RBAC / IAM** | Role-Based Access Control / Identity & Access Management |

## Architecture & platform

| Term | Meaning |
|---|---|
| **Landing zone** | A pre-provisioned, governed environment that workloads "land" in. |
| **Platform landing zone** | Shared platform services (management, connectivity, identity) owned by the platform team. |
| **Application landing zone** | A subscription handed to an app team, pre-wrapped in guardrails. |
| **Intermediate root (MG)** | The top ALZ management group (named with the ALZ prefix) that anchors all ALZ policy/RBAC. |
| **ALZ prefix** | The short name used for the intermediate-root management group and hierarchy. |
| **Corp** | Landing-zone archetype for workloads needing corporate/on-prem connectivity. |
| **Online** | Landing-zone archetype for internet-facing workloads. |
| **Sandbox** | Loosely-governed MG for experimentation. |
| **Decommissioned** | MG holding subscriptions being retired. |
| **Hub & spoke** | Network topology using a customer-managed hub **VNet**. |
| **Virtual WAN (vWAN)** | Microsoft-managed networking topology using virtual hubs + routing intent. |
| **Design area** | A category of decisions in ALZ (identity, network, security, governance, management, etc.). |
| **Feature area** | How ALZ engineering organizes work: Bicep, Terraform, Policy, Networking, Monitor, Security, Portal, Evergreen, Resource Management, Azure Enablement Score, VBD. |
| **Layer-0 / Layer-1** | Layer-0 = the technology (reference architecture + implementation); Layer-1 = delivery offerings (VBD). |

## Governance & policy

| Term | Meaning |
|---|---|
| **Azure Policy** | The engine ALZ uses for guardrails (audit/deny/deployIfNotExists, etc.). |
| **Policy definition** | A single policy rule. |
| **Initiative (policy set definition)** | A group of policy definitions assigned together. |
| **Policy assignment** | Applying a policy/initiative at a scope (usually a management group). |
| **ALZ custom policy** | Policy authored by the ALZ team; has a **readable name**; source of truth in `Enterprise-Scale` → ALZ Library. |
| **Built-in policy** | Policy authored by a product group; has a **GUID** name; owned by that PG. |
| **Override** | Mechanism to adjust a built-in policy's effect without authoring a custom one. |
| **Policy alias** | A property path a policy can target; must exist for a policy to be authorable. |
| **Archetype** | A named bundle of policy + role assignments + defaults applied to a management group. |
| **Policy refresh** | Quarterly batched policy release (`policy-refresh-fyXXqX` branch). |
| **EPaC** | Enterprise Policy as Code (`Azure/enterprise-azure-policy-as-code`). |

## Implementations & tooling

| Term | Meaning |
|---|---|
| **Portal Accelerator** | The ARM/portal RI; home `Azure/Enterprise-Scale`. |
| **Enterprise-Scale (ESLZ)** | The main ALZ repo; Portal Accelerator + source of truth for ALZ custom policies. |
| **caf-enterprise-scale** | The "classic" Terraform ALZ module. |
| **v.Next** | The newer AVM-based ALZ tooling generation (Terraform). |
| **AVM** | Azure Verified Modules — Microsoft's standardized module library (**pattern** modules built from **resource** modules). |
| **ALZ Library** | `Azure/Azure-Landing-Zones-Library` — data source of truth for policies, archetypes, defaults. |
| **alzlib** | Go library that reads ALZ Library definitions (v.Next engine). |
| **terraform-provider-alz** | The ALZ Terraform provider (v.Next). |
| **arm-template-parser** | Tool converting upstream policy defs/assignments to Terraform-compatible format. |
| **Accelerator** | Bootstrap automation (CI/CD, repo, identity) wrapping an RI for fast deployment. |
| **Subscription / LZ Vending** | Automated creation of application landing zones (`terraform-azurerm-lz-vending`, `bicep-lz-vending`→AVM). |
| **AzGovViz** | Azure Governance Visualizer — governance reporting tool. |
| **AzAdvertizer** | azadvertizer.net — searchable index of built-in policies/roles/aliases. |
| **AMBA** | Azure Monitor Baseline Alerts — optional monitoring layer integrated into ALZ. |
| **AzOps** | PowerShell module for GitOps management of ARM at MG scope. |

## Principles & lifecycle

| Term | Meaning |
|---|---|
| **Subscription democratization** | Subscriptions as the unit of scale/management; governed by inherited policy. |
| **Policy-driven governance** | Guardrails expressed as Azure Policy at MG scope, not manual enforcement. |
| **Federated model** | Platform team owns shared services; app teams own their landing zones within guardrails. |
| **Evergreen** | Keeping a deployed ALZ continuously updated with new ALZ releases. |
| **What's New** | The ALZ changelog (`aka.ms/alz/whatsnew`); updated for every change. |

## Clouds & support

| Term | Meaning |
|---|---|
| **Mooncake** | Azure China (operated by 21Vianet) sovereign cloud. |
| **Fairfax** | Azure US Government sovereign cloud. |
| **PG** | Product Group (the engineering team owning an Azure service/policy). |
| **RP** | Resource Provider. |
| **ServiceTree** | Internal service registry used to find a policy/service's owning team. |
| **ICM** | Incident Management (internal incident system). |
| **TFT** | A feedback/work-tracking item raised against a service. |
| **VBD** | Value Based Delivery — Microsoft delivery offerings for ALZ (Layer-1). |
| **ACP** | ALZ on Azure Connections Program — field/community feedback channel. |

---

**Prev:** [← 8. Operations & Lifecycle](./08-Operations-and-Lifecycle.md) · [Back to index](./README.md)
