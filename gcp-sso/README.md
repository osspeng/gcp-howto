# How to set an external IDP with Cloud Identity for GCP

To authenticate user to log on GCP console, the Google Cloud Identity is the default setup. It's easy for anybody who has a Google account to login. However for business users, many of them already have their existing authentication systems, the Microsoft Active Directory(AD) is a well-known example. To fulfil the use case, Google Cloud Identity can also integrate 3rd-party solution as its' identity provider(IDP).

In the following article, I'm going to show how to setup an external IDP for Cloud Identity to authenticate users log on to GCP console. The whole demo will all build from the ground as you just opened for business. The external IDP I select here is Active Directory (AD) with Active Directory Federation Service (ADFS), you may be using other IDP solutions like Okta, Keycloak, etc.


![GCP and ADFS](https://cloud.google.com/solutions/images/federating-gcp-with-ad-signing-in-to-gcp-console.svg)

This article is inspired by the [Google Cloud hybrid cloud solution tutorial](https://cloud.google.com/solutions/federating-gcp-with-active-directory-configuring-single-sign-on), you may find more detail explanation there.

## Getting started
Before following the steps to setup the sample environment, please make sure you got all the prerequisites ready on hands.

The installation process will covers:
1. Install and configure an Active Directory (AD) server.
2. Install Google Cloud Directory Sync and configure it to sync AD identities to Google Cloud Identity.
3. Install and configure an Active Directory Federation Servicer(ADFS) server.
4. Configure Cloud Identity to use ADFS for user authentication.
5. Testing on the deployment.

### Prerequisites
- A domain name which can be resolved on the internet.
- Prepare a valid certificate for the ADFS domain.
- [Register](https://cloud.google.com/identity/signup/premium/welcome) the Google Cloud Identity using the domain name.
- Register Google Cloud Platform account for hosting the AD & ADFS.

### A. Install and configure an Active Directory (AD) server.

[![Video: Install Microsoft Active Directory(AD) on Google Cloud Platform](https://img.youtube.com/vi/ztP-Yvn8TZE/hqdefault.jpg)](https://youtu.be/ztP-Yvn8TZE)

- provision a window server 2016 instance on GCP console.
- install AD and register a custom domain.

*Tips*:
- check the box of enable Https when creating Win2016 instance on GCP for later ADFS use.
- install the Microsoft Remote Desktop client will help to copy/paste command and share local folder to remote instance.
  - for  [Windwos](https://www.microsoft.com/en-us/p/microsoft-remote-desktop/9wzdncrfj3ps#activetab=pivot:overviewtab)
  - for [Mac](https://apps.apple.com/tw/app/microsoft-remote-desktop/id1295203466?mt=12)

### B. Create Account on Cloud Identity for Directory Sync
1. Login to [Google Admin](https://admin.google.com/) with your administrator account.

<img src="images/cloud-identity-1.png" alt="Google Admin Console" width="800"/>

2. Click the `Users` -> `Add new user`, then create a user for Directory Sync.

<img src="images/cloud-identity-2.png" alt="Create user" width="400"/>

3. Locate the newly created user and click it to open.
4. Under *Admin roles and privileges*, click `Assign Roles`.
5. Assign `Super Admin` role to the user.

<img src="images/cloud-identity-3.png" alt="Assign Super Admin" width="800"/>

### C. Install Google Cloud Directory Sync and configure it to sync AD identities to Google Cloud Identity.

[![Video: Install/Configure Google Cloud Directory Sync(GCDS) for AD sync](https://img.youtube.com/vi/u3fiFLDf4Tg/hqdefault.jpg)](https://youtu.be/u3fiFLDf4Tg)

- create an google identity account for AD sync.
- create an AD account for AD sync.
- install and config Google Cloud Directory Sync.
- create a AD user and test sync.

*Resource*:
- download Directory Sync [here](https://tools.google.com/dlpage/dirsync/thankyou.html)
- PowerShell command for creating envirnment folder(`c:\ProgramData\gcds`) for storing GCDS config files.

````
$gcdsDataFolder = "$Env:ProgramData\gcds"
New-Item -ItemType directory -Path  $gcdsDataFolder
&icacls "$gcdsDataFolder" /inheritance:r
&icacls "$gcdsDataFolder" /grant:r "CREATOR OWNER:(OI)(CI)F" /T
&icacls "$gcdsDataFolder" /grant   "BUILTIN\Administrators:(OI)(CI)F" /T
&icacls "$gcdsDataFolder" /grant   "Domain Admins:(OI)(CI)F" /T
&icacls "$gcdsDataFolder" /grant   "LOCAL SERVICE:(OI)(CI)F" /T

Write-Host $Env:ProgramData
````

*Tips*:
1. you can either use the `domain\username` or `username@[FQDN]` format for Authorized User under the LDAP Configuration.
2. The default query rule querying all users who have email attribute set.

``(&(objectCategory=person)(objectClass=user)(mail=*)(!(userAccountControl:1.2.840.113556.1.4.803:=2)))``

3. When create users for sync, the **email** attribute must be set as it's the user ID in Cloud Identity.

### D. Install and configure an Active Directory Federation Services(ADFS).

[![Video: Install and configure Active Directory Federated Services & Cloud Identity](https://img.youtube.com/vi/af_mjXHbSQQ/hqdefault.jpg)](https://youtu.be/af_mjXHbSQQ)

- install and configure ADFS to be available to access.
- configure mapping of AD email attribute to SAML NameID
- configure Cloud Identity to use ADFS as external IDP.

*Resource*:

*Tips*:

- a SSL cert in ptx format will be required in the installation, you can [Generate SSL certificate using Let's Encrypt](https://medium.com/@saurabh6790/generate-wildcard-ssl-certificate-using-lets-encrypt-certbot-273e432794d7)

### E. Testing on the deployment.

[![Video: Verify ADFS and Cloud Identity integration](https://img.youtube.com/vi/6u0WblK6Kb8/hqdefault.jpg)](https://youtu.be/6u0WblK6Kb8)

- Login to GCP console with an AD user account.


### F. Scheduling the AD sync

[![Video: Schedule GCDS to run periodically](https://img.youtube.com/vi/ssvLMPBh1WQ/hqdefault.jpg)](https://youtu.be/ssvLMPBh1WQ)

- create a powershell script(ps1) for GCDS.
- create a schedule task for the ps1 script.
- manual trigger the task.

*Resource*:

- powershell script sample

````
[CmdletBinding()]
Param(
    [Parameter(Mandatory=$True, Position=1)]
    [string]$config,

    [Parameter(Mandatory=$True, Position=1)]
    [string]$gcdsInstallationDir
)

import-module ActiveDirectory

# Stop on error.
$ErrorActionPreference ="stop"

# Ensure it's an absolute path.
$rawConfigPath = [System.IO.Path]::Combine((pwd).Path, $config)

# Discover closest GC in current domain.
$dc = Get-ADDomainController -discover -Service "GlobalCatalog" -NextClosestSite
Write-Host ("Using Global Catalog server {0} of domain {1} as LDAP source" -f [string]$dc.HostName, $dc.Domain)

# Load XML and replace the endpoint.
$dom = [xml](Get-Content $rawConfigPath)
$ldapConfigNode = $dom.SelectSingleNode("//plugin[@class='com.google.usersyncapp.plugin.ldap.LDAPPlugin']/config")

# Tweak the endpoint.
$ldapConfigNode.hostname = [string]$dc.HostName
$ldapConfigNode.ldapCredMachineName = [string]$dc.HostName
$ldapConfigNode.port = "3268"   # Always use Global Catalog port

# Tweak the tsv files location
$googleConfigNode = $dom.SelectSingleNode("//plugin[@class='com.google.usersyncapp.plugin.google.GooglePlugin']/config")
$googleConfigNode.nonAddressPrimaryKeyMapFile = [System.IO.Path]::Combine((pwd).Path, "nonAddressPrimaryKeyFile.tsv")
$googleConfigNode.passwordTimestampFile = [System.IO.Path]::Combine((pwd).Path, "passwordTimestampCache.tsv")

# Save resulting config.
$targetConfigPath = $rawConfigPath + ".autodiscover"

$writer = New-Object System.IO.StreamWriter($targetConfigPath, $False, (New-Object System.Text.UTF8Encoding($False)))
$dom.Save($writer)
$writer.Close()

# Run synchronization.
Start-Process -FilePath "$gcdsInstallationDir\sync-cmd" `
    -Wait -ArgumentList "--apply --loglevel INFO --config ""$targetConfigPath"""
````

- copying GCDS required key from your own profile to the profile of `NT AUTHORITY\LOCAL SERVICE`:

````
New-Item -Path Registry::HKEY_USERS\S-1-5-19\SOFTWARE\JavaSoft\Prefs\com\google\usersyncapp -Force

Copy-Item -Path Microsoft.PowerShell.Core\Registry::HKEY_CURRENT_USER\SOFTWARE\JavaSoft\Prefs\com\google\usersyncapp\util `
    -Destination Microsoft.PowerShell.Core\Registry::HKEY_USERS\S-1-5-19\SOFTWARE\JavaSoft\Prefs\com\google\usersyncapp\util
````

- task scheduler sample

````
$taskName = "Synchronize to Cloud Identity"
$gcdsDir = "$Env:ProgramData\gcds"

$action = New-ScheduledTaskAction -Execute 'PowerShell.exe' `
  -Argument "-ExecutionPolicy Bypass -NoProfile $gcdsDir\sync.ps1 -config $gcdsDir\config.xml -gcdsInstallationDir '$Env:Programfiles\Google Cloud Directory Sync'" `
  -WorkingDirectory $gcdsDir
$trigger = New-ScheduledTaskTrigger `
  -Once `
  -At (Get-Date) `
  -RepetitionInterval (New-TimeSpan -Minutes 60) `
  -RepetitionDuration (New-TimeSpan -Days (365 * 20))

$principal = New-ScheduledTaskPrincipal -UserID "NT AUTHORITY\LOCAL SERVICE" -LogonType ServiceAccount
Register-ScheduledTask -Action $action -Trigger $trigger -Principal $principal -TaskName $taskName

$task = Get-ScheduledTask -TaskName "$taskName"
$task.Settings.ExecutionTimeLimit = "PT12H"
Set-ScheduledTask $task
````

### Usfel Note
- CLI for running GCDS
````
// Simulate
C:\Program Files\Google Cloud Directory Sync> .\sync-cmd.exe -c c:\ProgramData\gcds\config.xml
// Commit
C:\Program Files\Google Cloud Directory Sync> .\sync-cmd.exe -c c:\ProgramData\gcds\config.xml
````
