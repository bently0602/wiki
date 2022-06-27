- Install 7-zip.
- Install Orca, a Microsoft-provided MSI editor. Heres the link, you only need to install the Orca part.
  * https://docs.microsoft.com/en-us/windows/win32/msi/orca-exe?redirectedfrom=MSDN
- iCloud = https://updates.cdn-apple.com/2020/windows/001-39935-20200911-1A70AA56-F448-11EA-8CC0-99D41950005E/iCloudSetup.exe
- Open iCloudSetup.exe with 7-zip. Right-click it, select 7-zip > Open with 7-zip, Extract the appropriate version of iCloud somewhere (usually this is iCloud64)
- Open Orca, and open the iCloud MSI you just extracted
  1. Go to the LaunchCondition table
  2. Change this line:
    From: (VersionNT >= 601) AND (MsiNTProductType = 1)
    To: (VersionNT >= 601) AND (MsiNTProductType = 3)
- Save and quit
- install these first
  * AppleApplicationSupport64.msi
  * AppleApplicationSupport.msi
  * AppleSoftwareUpdate.msi
- install your new iCloud MSI
