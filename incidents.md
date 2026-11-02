# Incident Tickets

Format: alert, triage, action, closure. INC-001 and INC-002 were created on purpose to test the alerting setup, and are marked as simulated.

---

## INC-001: High CPU on vm-web-01 (Simulated)

| Field | Value |
|---|---|
| Severity | Sev 3 |
| Source | Azure Monitor alert rule "High cpu alert" |
| Affected resource | vm-web-01 (rg-spoke-web) |
| Fired | 31 Oct 2026, 11:27 AM IST |
| Status | Closed |

**Alert:** Percentage CPU exceeded the alert threshold. Email notification received.

**Triage**
1. Opened the VM, then Monitoring, then Metrics. Max Percentage CPU reached about 99%.
2. Checked VM availability. VM was running and reachable (no outage).
3. Used Run Command to confirm which process was using CPU: `ps aux --sort=-%cpu | head`.
4. Found the planned test load (`yes` processes started for the alert test).

**Action:** No fix needed because it was a planned load test. The load was started with a 600-second timeout and ended by itself. Confirmed CPU returned to normal in Metrics.

**Closure notes:** Alert path works end to end (rule, action group, email). For a real incident: identify the process, check whether it is expected, restart or stop the service, resize the VM if load is sustained, and escalate to L2 if CPU stays high.

---

## INC-002: VM availability alert on vm-web-01 (Simulated)

| Field | Value |
|---|---|
| Severity | Sev 3 |
| Source | Azure Monitor alert rule "VM Availability - vm-web-01" |
| Affected resource | vm-web-01 |
| Fired | 30 Oct 2026, 7:32 PM IST |
| Status | Closed |

**Alert:** VM availability dropped below 100%.

**Triage**
1. Checked the Overview page. Status showed Stopped (deallocated).
2. Checked the Activity log to see who stopped the VM and when. It was my own stop operation.
3. Confirmed no platform outage in Monitor, Service health.

**Action:** Started the VM from the portal and confirmed status Running and availability Available.

**Closure notes:** Expected behaviour for a deallocated VM. In production, check the Activity log first to decide between an intended stop, an auto-shutdown schedule, and a real fault. Escalate to L2 if the VM does not start or the platform reports an issue.

---

## INC-003: Azure Site Recovery replication failed for vm-web-01

| Field | Value |
|---|---|
| Severity | Sev 3 |
| Source | Failed deployment (Microsoft.RecoveryServices.IaaSDR) |
| Affected resource | vm-web-01 disaster recovery |
| Status | Closed with workaround |

**Symptom:** Enabling replication failed. The step `asr-a2a-default-centralindia` showed Conflict.

**Triage**
1. Opened the failed deployment, then Error details.
2. Error: the recovery services vault in the chosen target region is not yet supported for Azure Site Recovery Azure-to-Azure replication (code 908).
3. Root cause: the target region picked for the vault does not support A2A disaster recovery. A retry with the same region cannot succeed.

**Action**
1. Deleted the partial resources created by the failed run (`Site-recovery-vault-RG` with the vault and cache storage account, plus the DR resource group) to avoid background charges.
2. Wrote the DR runbook (see `runbooks.md`) with the region support check as step 1.
3. Documented that backup protects the VM in the meantime.

**Closure notes:** Always check the Azure Site Recovery supported-region matrix before choosing a target region. For a production ticket, escalate to L2 or the architect with the error code and the region options.
