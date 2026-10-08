# Part C — Decommissioning Review

## Completed Lab Work

The logical decommissioning required by the Week 8 lab was completed. Backups were completed and verified before destructive work began. The eight lab VMs, their verified storage remnants, student shares, lab files, course applications, and stale profile data were removed.

`WINTHIRTYFOUR` was confirmed as the last Domain Controller and holder of all five FSMO roles. It was demoted through the normal last-DC workflow without forced removal. The process removed the domain and its DNS/application partitions, and the server was restarted after demotion. AD DS and DNS binaries and management tools were then removed. Final checks confirmed workgroup status, no registered VMs, no non-system shares, absent lab paths, preserved backups, and final server shutdown. This order follows Microsoft guidance to demote a Domain Controller before removing the AD DS role binaries ([Microsoft Learn — Demote Domain Controllers and Domains](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/deploy/demoting-domain-controllers-and-domains--level-200-)).

## Production Controls Outside This Lab

### 1. Media Sanitization or Secure Erase

The lab removed logical artifacts, but it did not sanitize or destroy the physical storage behind `C:` or `E:`. This was intentional because the assignment required the backups under `E:\CapstoneBackup` and `E:\WindowsImageBackup` to remain available. A production retirement would select and document an appropriate sanitization method based on the media type, data sensitivity, and reuse or disposal decision. NIST describes media sanitization as making access to target data infeasible for a defined level of effort ([NIST SP 800-88 Rev. 2 — Guidelines for Media Sanitization](https://csrc.nist.gov/pubs/sp/800/88/r2/final)).

### 2. Backup Recovery Testing

The file, Group Policy, and System State backups completed and their presence was verified. An actual restore test was not performed. In production, critical backups should be restored in a controlled test so the organization knows that the data and system state can be recovered, not only that backup files exist. Microsoft documents recovery by backup version and target through `wbadmin start systemstaterecovery` ([Microsoft Learn — wbadmin start systemstaterecovery](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/wbadmin-start-systemstaterecovery)).

### 3. External Asset and Service Offboarding

The isolated SierraLab environment did not include a real configuration management database (CMDB), enterprise monitoring platform, license inventory, or external service registry. A production decommission should update those systems so the retired server, software licenses, monitoring targets, support records, and ownership data do not remain active. This supports the requirement to maintain an accurate and current inventory of system components ([NIST SP 800-53 Rev. 5.1 — CM-8 System Component Inventory](https://csrc.nist.gov/CSRC/media/Projects/risk-management/800-53%20Downloads/800-53r5/SP_800-53_v5_1-derived-OSCAL.pdf)).

## Conclusion

The SierraLab exercise completed the required logical decommissioning in a safe order: backup, workload removal, normal Domain Controller demotion, role removal, verification, and shutdown. Media sanitization, restore testing, and enterprise asset/service offboarding remain separate production controls because they were outside the lab scope.
