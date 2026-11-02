# Runbooks

## 1. Restore a VM from Azure Backup

**Use when:** the VM is corrupted, deleted, or a file must be recovered.

1. Open the Recovery Services vault (`rsv-azureops-lab`), then Backup items, then Azure Virtual Machine, then `vm-web-01`.
2. Check the last backup status and the available restore points.
3. Click **Restore VM** and choose a restore point.
4. Choose the restore type: **Create new VM** (safe, preferred), **Replace existing**, or **Restore disks**.
5. Select the target resource group, virtual network, and subnet (`rg-spoke-web`, `vnet-spoke-web`, `snet-web`), then start the restore.
6. Track progress under Backup jobs.
7. Verify: VM Running, disks attached, app responding, NSG applied through the subnet.
8. Record the restore point used and the time taken in the ticket.

**If the backup job failed:** check the VM agent status, check that the VM was running, check the Activity log, retry the job once, then escalate to L2 with the job ID.

## 2. Disaster recovery with Azure Site Recovery (documented, not executed)

**Status:** an attempt in this lab failed because the target region was not supported for Azure-to-Azure replication (see INC-003).

1. **Check region support first.** Confirm the source and target regions are supported for A2A replication in the Site Recovery support matrix. Use the paired region where possible.
2. Confirm prerequisites: VM running with the agent healthy, vault in the target region, outbound connectivity for replication, permissions to create resources.
3. On the VM, open **Disaster recovery**, choose the target region, review the target resource group, VNet, and the cache storage account, then start replication.
4. Wait for initial replication to complete and the replication health to show Healthy.
5. **Test failover:** use an isolated test network, confirm the VM boots and the application responds, then **clean up the test failover**.
6. **Planned failover (cutover):** shut down the source if possible, fail over to the latest recovery point, validate the application, DNS and connectivity, then **commit**.
7. **Rollback:** re-protect the VM in the original direction, then fail back and commit.
8. **Cleanup:** disable replication and delete the vault, cache storage, and target resource groups when the exercise is over.

**Escalate to L2/Architect when:** replication health is Critical, failover fails validation, or RPO is breached.
