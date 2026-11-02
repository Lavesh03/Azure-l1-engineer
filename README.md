# Azure L1 Ops Lab

Hands-on lab that mirrors the day-to-day work of an **Azure L1 Engineer**: VM administration, virtual networking, backup, monitoring and alerts, identity and access (RBAC), App Service, storage, and incident handling.

Built on an Azure for Students subscription in **Central India**. Every resource is tagged `owner`, `env=lab`, `project=AzureOps-L1`.

## Architecture

```
                  vnet-hub 10.0.0.0/16
                  snet-mgmt 10.0.1.0/24  (nsg-mgmt)
                    |                  |
          VNet peering            VNet peering
          (Connected)             (Connected)
                    |                  |
   vnet-spoke-web 10.1.0.0/16     vnet-spoke-data 10.2.0.0/16
   snet-web 10.1.1.0/24           snet-data 10.2.1.0/24
   (nsg-web)                      (nsg-data)
   vm-web-01 (Ubuntu 24.04)       stazureopslavesh01 (Blob, LRS)
```

Also deployed: Recovery Services vault `rsv-azureops-lab`, Azure Monitor alert rules, an Entra ID group with RBAC roles, and App Service `app-azureops-lavesh` (Free F1).

## What I built

| Area | What I did | Evidence |
|---|---|---|
| Resource organisation | 3 resource groups (`rg-hub`, `rg-spoke-web`, `rg-spoke-data`) with a consistent tagging standard | `screenshots/01-resource-groups.png` |
| Networking | Hub-and-spoke design: 3 VNets with non-overlapping address spaces, subnets, hub-to-spoke peering (both Connected) | `02-vnets.png`, `04-peerings.png` |
| Network security | NSGs attached at subnet level. Web tier open on 80/443, data tier reachable only from the web and management subnets | `03-nsg-rules.png` |
| VM administration | Deployed `vm-web-01` (Ubuntu 24.04, B2ats_v2, no public IP, auto-shutdown 23:00 IST). Start, stop and deallocate, resize planning, Run Command | `05-vm-overview.png` |
| Backup | Recovery Services vault, Enhanced backup policy, on-demand backup of the VM | `06-backup-completed.png` |
| Monitoring and alerts | Azure Monitor alerts for high CPU and VM availability, email action group. Triggered both and received the notifications | `07-cpu-spike.png`, `08-alerts-fired.png`, `09-alert-email.png` |
| Identity and RBAC | Entra group `L1-Support` with **Reader** and **Virtual Machine Contributor** scoped to `rg-spoke-web` (least privilege) | `10-rbac-roles.png` |
| App Service | Linux web app (Node 22 LTS) on the Free F1 plan, reviewed log stream | `11-app-service.png` |
| Storage | Standard LRS general-purpose v2 account, anonymous blob access disabled, TLS 1.2 minimum, soft delete on | `12-storage-account.png` |
| Disaster recovery | Attempted Azure Site Recovery. Blocked by an unsupported target region, so documented as a runbook (see `runbooks.md` and `incidents.md`) | |

## Key configuration

**NSG rules**

| NSG | Priority | Rule | Source | Port | Action |
|---|---|---|---|---|---|
| nsg-web | 100 | Allow-web | Any | 80, 443 (TCP) | Allow |
| nsg-data | 100 | Allow-From-Web | 10.1.1.0/24 | 1433 (TCP) | Allow |
| nsg-data | 110 | Allow-From-Mgmt | 10.0.1.0/24 | 22, 3389 (TCP) | Allow |

Admin access to `vm-web-01` uses Azure Run Command, so no inbound admin port is exposed to the internet and the VM has no public IP.

**Alert test**
- Ran a CPU load through Run Command. Max CPU reached about 99%.
- The CPU alert fired (Sev 3) and the email notification arrived.
- Stopping the VM fired the VM availability alert.

## Real-world constraints and what I did about them

| Problem | Cause | Fix |
|---|---|---|
| `Standard_D2s_v3` rejected | Size not available for the subscription in Central India | Checked the size list, chose `B2ats_v2` |
| Deployments blocked by policy | Azure for Students region restrictions | Used the allowed region and documented the limits |
| Overlapping VNet address spaces | Default ranges reused across VNets | Re-addressed so every VNet is unique, then peering worked |
| Site Recovery failed | Target vault region not supported for Azure-to-Azure DR (code 908) | Cleaned up partial resources, wrote the DR runbook |
| App Service created on Basic B1 | Wizard default | Scaled down to Free F1 |

## Cost control
Small B-series VM with auto-shutdown, VM deallocated after each session, Free App Service plan, LRS storage, short backup retention, failed and partial resources deleted, and all resource groups removed after documenting the lab.

## Repo layout

```
README.md
incidents.md        3 incident tickets (alert, triage, fix, closure)
runbooks.md         backup restore and disaster recovery runbooks
screenshots/        evidence for every step
```
