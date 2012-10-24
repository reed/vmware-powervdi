MU Division of IT VDI PowerShell Module
=======================================

Written by Nick Reed (reednj@missouri.edu)

Purpose
-------
For automating common VMware View tasks.

Prerequisites
-------------
  - VMware PowerCLI installed
  - WinRM enabled on View Connection Server

Installation
----------
Copy the PowerVDI folder to `C:\Windows\System32\WindowsPowerShell\v1.0\Modules\`

Usage
-----
```powershell
Import-Module PowerVDI
Connect-VDI <broker fqdn> -vcenter <vcenter fqdn> [-as domain\username]
```

Changelog
---------
### 0.2.0 (24 October 2012)
  - Added `Export-Pools`
  - Added `Import-Pools`
  - Added support for all pool types to `New-PoolFromCopy`
  - Added settings.ini file

### 0.1.0 (27 September 2012)
  - First release

API
---

Run `Get-Help <command-name> -full` to get more detailed documentation.

### Action Commands

#### Add-Entitlement

> `[-name | -username | -user] <string>`

> `[[-pools | -pool | -pool_id] <object>]`

#### Connect-VDI

> `[-broker] <string>`

> `[[-vcenter] <string>]`

> `[[-credential | -as] <object>]`
	
#### Disconnect-VDI
  
#### Export-Pools

> `[-path] <string>`

> `[[-pools | -pool | -pool_id] <object>]`

#### Import-Pools

> `[-path] <string>`

> `[[-pools | -pool | -pool_id] <object>]`

> `[[-settings] <hashtable>]`

#### New-PoolFromCopy
	
> `[-source] <object>`

> `[-pool_id] <string>`

> `[[-description] <string>]`

> `[[-displayName] <string>]`

> `[[-namePrefix] <string>]`

> `[[-settings] <hashtable>]`
	
#### Remove-Entitlement

> `[-name | -username | -user] <string>`

> `[[-pools | -pool | -pool_id] <object>]`
	
#### Send-ConsoleSessionRefresh
	
#### Send-PoolRecompose
	
> `[[-pools] <object>]`

> `[[-except] <string[]>]`

> `[[-forceLogoff] <boolean>]`

> `[[-stopOnError] <boolean>]`

> `[-Simulate]`
	
#### Send-PoolRefresh
	
> `[[-pools] <object>]`

> `[[-except] <string[]>]`
	
> `[[-forceLogoff] <boolean>]`
	
> `[[-stopOnError] <boolean>]`
	
> `[-Simulate]`
	
#### Set-PoolSnapshot
	
> `[-snapshot] <object>`

> `[[-pools] <object>]`

### Helper Commands
  
#### Confirm-Entitlement

> `[-name] <string>`

> `[-pool] <object>`
	
#### Confirm-PoolSnapshotOutdated

> `[-pool] <object>`

#### Confirm-ReplicaProvisioned

> `[-vm] <object>`

> `[-snapshot] <object>`
	
#### Confirm-SnapshotExists
	
> `[-vm] <object>`

> `[-snapshot] <object>`
	
#### Confirm-SnapshotPathExists

> `[-vm] <object>`
	
> `[-snapshot] <object>`
	
#### Confirm-VDIConnected
  
#### Get-LatestSnapshot

> `[-vm] <object>`

> `[-Path]`
	
#### Get-OutdatedVMs
	
> `[[-pools] <object>]`

> `[[-except] <string[]>]`
	
#### Get-PoolMaster

> `[-pool] <object>`

> `[-Path]`
	
#### Get-PoolSnapshot

> `[-pool] <object>`
	
> `[-Path]`
	
#### Get-PoolsWithOutdatedSnapshots
  
#### Show-Pools
  
#### Show-PoolsWithOutdatedSnapshots
  
#### Show-VDICommands