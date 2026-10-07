# Backup Plan

## Purpose and Scope

This plan protects the important data and configuration from the `WINTHIRTYFOUR` Windows Server 2025 host before decommissioning. All new backup sets will be stored on the secondary drive, `E:` (`HyperVData`). The plan includes important files, Active Directory, and Group Policy.

The existing contents of `E:` must not be deleted or overwritten during backup preparation. Because `E:\HyperV`, `E:\Shares`, `E:\Software`, and an earlier System State backup may already exist, the new backup will use a separate, dated folder such as `E:\CapstoneBackup\YYYY-MM-DD`.

## Preparation

1. Sign in to `WINTHIRTYFOUR` with an authorized administrative account.
2. Confirm that `E:` is the secondary disk labeled `HyperVData`:

   ```powershell
   Get-Volume -DriveLetter E | Select-Object DriveLetter, FileSystemLabel, FileSystem, HealthStatus, SizeRemaining, Size
   ```

3. Inventory the backup sources and estimate their size without changing them:

   ```powershell
   Get-ChildItem -LiteralPath 'C:\Shares','E:\Shares','E:\Software' -Force -ErrorAction SilentlyContinue
   Get-WindowsFeature Windows-Server-Backup
   ```

4. Verify that `E:` has enough free space for the files and System State backup. Record the date, available space, and intended destination folder.
5. Create a new dated destination folder on `E:`. Do not reuse or erase the earlier backup until the new backup is verified.

   ```powershell
   $BackupRoot = "E:\CapstoneBackup\$(Get-Date -Format 'yyyy-MM-dd')"
   New-Item -ItemType Directory -Path $BackupRoot -Force
   ```

## Back Up Important Files

1. Copy important shared data from `C:\Shares` and `E:\Shares` into separate folders under the dated backup root. Use `robocopy` so that errors and skipped files are logged.

   ```powershell
   robocopy 'C:\Shares' "$BackupRoot\Files\C_Shares" /E /COPY:DAT /DCOPY:DAT /R:2 /W:5 /XJ /LOG:"$BackupRoot\C_Shares.log"
   robocopy 'E:\Shares' "$BackupRoot\Files\E_Shares" /E /COPY:DAT /DCOPY:DAT /R:2 /W:5 /XJ /LOG:"$BackupRoot\E_Shares.log"
   ```

2. Copy the software deployment source files from `E:\Software` because they may be needed to document or rebuild the lab configuration.

   ```powershell
   robocopy 'E:\Software' "$BackupRoot\Files\Software" /E /COPY:DAT /DCOPY:DAT /R:2 /W:5 /XJ /LOG:"$BackupRoot\Software.log"
   ```

3. Review each `robocopy` log. Exit codes `0` through `7` can represent success with differences or skipped items; an exit code of `8` or higher indicates at least one copy failure that must be corrected.

> **Optional note:** VM export is not required for Part A and should not be performed unless the instructor specifically requests it. Exporting the VMs to the same `E:` drive could duplicate large VHDX or AVHDX files and consume needed backup space.

## Back Up Active Directory

`WINTHIRTYFOUR` is the Domain Controller for `it115.test`. Use Windows Server Backup to create a System State backup on `E:`. System State includes the AD DS database and other required system components for domain-controller recovery.

1. Confirm that Windows Server Backup is installed:

   ```powershell
   Get-WindowsFeature Windows-Server-Backup
   ```

2. Start a new System State backup to the secondary drive. This is a planned command and must be run only during the approved backup window:

   ```powershell
   wbadmin start systemstatebackup -backuptarget:E: -quiet
   ```

3. Wait for the operation to finish. Do not begin decommissioning while the backup is running.

4. Verify that Windows Server Backup reports a successful backup and list the recorded versions:

   ```powershell
   wbadmin get versions -backuptarget:E:
   Get-WinEvent -LogName 'Microsoft-Windows-Backup' -MaxEvents 20 |
     Select-Object TimeCreated, Id, LevelDisplayName, Message
   ```

## Back Up Group Policy

System State protects Group Policy as part of Active Directory, but a separate GPO backup makes individual policies easier to restore and inspect.

1. Create a dedicated GPO backup folder inside the dated backup root.
2. Load the Group Policy PowerShell module and back up every GPO in `it115.test`:

   ```powershell
   Import-Module GroupPolicy
   New-Item -ItemType Directory -Path "$BackupRoot\GPO_Backup" -Force
   Backup-GPO -All -Domain 'it115.test' -Path "$BackupRoot\GPO_Backup" -Comment 'IT-0115 Week 8 Capstone backup'
   ```

3. Export a readable inventory for verification:

   ```powershell
   Get-GPO -All -Domain 'it115.test' |
     Select-Object DisplayName, Id, GpoStatus, CreationTime, ModificationTime |
     Export-Csv "$BackupRoot\GPO_Inventory.csv" -NoTypeInformation
   ```

## Final Verification

Before approving decommissioning:

1. Confirm that the dated backup folder exists on `E:` and contains the expected file, software, and GPO backup folders.
2. Compare source and destination file counts and sizes. Review all copy logs for failures.
3. Confirm that `wbadmin get versions -backuptarget:E:` lists the new System State backup.
4. Confirm that the GPO backup folder contains backup data and that `GPO_Inventory.csv` lists the expected domain policies.
5. Open a small sample of backed-up documents directly from the destination to confirm readability. Do not alter the originals.
6. Save the backup logs and verification notes in the dated backup folder.
7. Record approval to begin decommissioning only after every required backup passes verification.

## Expected Result

The expected result is one verified, dated backup set on the `HyperVData` secondary drive. It will contain important files, a current Active Directory System State backup, a separate backup of all Group Policy objects, and verification logs. No source data will be removed as part of this plan.

