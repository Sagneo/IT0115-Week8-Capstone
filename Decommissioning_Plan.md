# Decommissioning Plan

## Purpose and Safety Conditions

This plan removes the IT-0115 lab workloads and data from `WINTHIRTYFOUR` after the required backups have been completed and verified. It is a plan only; it does not claim that virtual machines, Active Directory, files, roles, or the server have already been removed.

Deletion is permanent and must not begin until the backup plan has passed verification. The `E:` drive contains both items scheduled for removal and material that must be preserved, so broad deletion commands must not be used. Every target must be checked individually.

## 1. Verify Backups and Record the Starting State

1. Confirm that the dated backup folder on `E:\CapstoneBackup` contains readable copies of important files, a successful System State backup, a separate GPO backup, inventories, and logs.
2. Confirm the System State backup version:

   ```powershell
   wbadmin get versions -backuptarget:E:
   ```

3. Confirm the GPO backup and inventory are present.
4. Record the current VMs, installed roles, applications, shares, and selected data paths:

   ```powershell
   Get-VM | Select-Object Name, State, Path
   Get-WindowsFeature | Where-Object InstallState -eq 'Installed'
   Get-SmbShare | Select-Object Name, Path, Description
   Get-Package | Select-Object Name, Version, ProviderName
   Get-ChildItem -LiteralPath 'C:\Shares','E:\Shares','E:\Software','E:\HyperV' -Force -ErrorAction SilentlyContinue
   ```

5. Do not continue if any required backup is missing, unreadable, incomplete, or stored inside a folder scheduled for deletion.

## 2. Decommission and Delete the Virtual Machines

1. Confirm that `WIN10-LAB` and `UBUNTU-LAB` are the only VMs assigned to this lab. Investigate any unexpected VM before continuing.
2. Shut down each guest from its operating system when possible. If a guest does not respond, document the condition before using a forced stop.
3. Verify both VMs are off:

   ```powershell
   Get-VM -Name 'WIN10-LAB','UBUNTU-LAB' | Select-Object Name, State, Status
   ```

4. Remove the two VMs from Hyper-V Manager or with PowerShell after their names and paths are verified:

   ```powershell
   Remove-VM -Name 'WIN10-LAB','UBUNTU-LAB' -Force
   ```

5. `Remove-VM` removes the Hyper-V configuration but may leave virtual disks and other files. Review `E:\HyperV` and delete only files that belong to these two VMs. Do not delete the dated capstone backup or unrelated content on `E:`.
6. Verify removal:

   ```powershell
   Get-VM
   Get-ChildItem -LiteralPath 'E:\HyperV' -Force -Recurse -ErrorAction SilentlyContinue
   ```

## 3. Remove Lab Applications, Shares, and Files

1. Compare installed applications with the starting inventory. Uninstall only applications installed for student lab activities, using **Settings > Apps > Installed apps**, the original installer, or the vendor's supported uninstaller.
2. List SMB shares before removal. Remove student-created share definitions first, but do not remove default administrative shares.

   ```powershell
   Get-SmbShare | Select-Object Name, Path, Special
   ```

3. After each target is confirmed, remove the student-created share with `Remove-SmbShare -Name '<share-name>' -Force`.
4. Review and delete only lab-related data from these known locations after the verified backup is protected:

   - `C:\Shares`
   - `E:\Shares`
   - `E:\Software`
   - remaining `WIN10-LAB` and `UBUNTU-LAB` files under `E:\HyperV`

5. Review scheduled tasks, scripts, installer packages, ISO files, temporary exports, and student-created local profiles. Remove only items associated with the course lab.
6. Empty the Recycle Bin only after the target list has been reviewed a second time. Keep `E:\CapstoneBackup\YYYY-MM-DD` until the instructor's retention requirement is satisfied.

## 4. Demote the Domain Controller Correctly

`WINTHIRTYFOUR` is the Domain Controller for the single lab domain `it115.test`. Demotion must occur before AD DS binaries and related components are removed.

1. Verify that the backups are still available and that no required workload depends on `it115.test`.
2. Review the domain-controller state and AD DS health. Any error that affects safe demotion must be resolved or documented before continuing:

   ```powershell
   Get-ADDomainController -Filter * | Select-Object HostName, Site, IsGlobalCatalog, OperationMasterRoles
   dcdiag /v
   netdom query fsmo
   ```

3. Because this is the lab's last Domain Controller, use **Server Manager > Manage > Remove Roles and Features**. Clear **Active Directory Domain Services**, select **Demote this domain controller**, and choose **Last domain controller in the domain** only after confirming that no other DC remains.
4. When prompted, set a secure local Administrator password. Do not record that password in this repository or in screenshots.
5. Confirm the removal of the `it115.test` domain and complete the demotion wizard. Restart when prompted.
6. If PowerShell is required instead of Server Manager, use the supported `Uninstall-ADDSDomainController` workflow for the last DC. Supply the local Administrator password interactively as a `SecureString`; never place it in a script, command history, or document.
7. After restart, sign in with the local Administrator account and verify that the server is no longer a Domain Controller or a member of `it115.test`.
8. Remove remaining AD DS role binaries only after successful demotion:

   ```powershell
   Uninstall-WindowsFeature AD-Domain-Services -IncludeManagementTools
   ```

9. Review DNS and Group Policy management components. Remove role services or tools that were installed only for the retired lab domain, but keep anything required by the remaining standalone server.

## 5. Verify That No Student Activity Remains

1. Confirm that no lab VMs are registered and no files for `WIN10-LAB` or `UBUNTU-LAB` remain outside the protected backup.
2. Confirm that the known student shares, software deployment folders, scripts, ISOs, scheduled tasks, and application data are removed.
3. Confirm that AD DS is not installed and that the server is not a Domain Controller:

   ```powershell
   Get-WindowsFeature AD-Domain-Services
   Get-CimInstance Win32_ComputerSystem | Select-Object Name, Domain, PartOfDomain
   ```

4. Review remaining local accounts and profiles. Remove only student-created accounts and profiles after confirming they are no longer needed. Preserve built-in and required administrative accounts.
5. Search the known data locations for lab names and the old domain name. Review every result before deletion:

   ```powershell
   Get-ChildItem -LiteralPath 'C:\Shares','E:\Shares','E:\Software','E:\HyperV' -Force -Recurse -ErrorAction SilentlyContinue |
     Where-Object FullName -Match 'WIN10-LAB|UBUNTU-LAB|it115\.test'
   ```

6. Review Event Viewer, Server Manager, Hyper-V Manager, installed roles, SMB shares, and installed applications for unexpected remnants or errors.
7. Document the final verification results without including passwords, tokens, product keys, private keys, or other secrets.

## 6. Final Shutdown

1. Confirm that the verified backup remains available on `E:` and is not part of the deletion list.
2. Confirm that all required decommissioning checks have passed and that no task is still running.
3. Obtain any required instructor or lab approval.
4. Shut down `WINTHIRTYFOUR` only after verification is complete:

   ```powershell
   Stop-Computer
   ```

## Expected Result

The expected result is a safely decommissioned lab server with the two Hyper-V VMs removed, student applications and files removed, `WINTHIRTYFOUR` correctly demoted from the `it115.test` domain, AD DS-related components removed as appropriate, and no student activity remnants left outside the protected backup. The server will be shut down only after all verification steps pass.

