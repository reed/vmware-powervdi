MU Division of IT VDI PowerShell Module
=======================================

Written by Nick Reed (reednj@missouri.edu)

Purpose
-------
For remotely automating common VMware View tasks.

Prerequisites
-------------
  - [VMware PowerCLI](https://my.vmware.com/web/vmware/details?downloadGroup=VSP510-PCLI-510&productId=285) installed
  - PowerShell Remoting (WinRM) enabled on View Connection Server

Installation
----------
Copy the PowerVDI folder to `C:\Windows\System32\WindowsPowerShell\v1.0\Modules\`

Usage
-----
```powershell
Import-Module PowerVDI
Connect-VDI <broker fqdn> -vcenter <vcenter fqdn> [-as domain\username]
```

Commands ([API](https://github.com/reednj77/vmware-powervdi/wiki/API))
---

Run `Get-Help <command-name> -full` to get more detailed documentation.

### Action Commands
***

##### Add-Entitlement

##### Connect-VDI

##### Disconnect-VDI

##### Export-Pools

##### Import-Pools

##### New-PoolFromCopy

##### Remove-Entitlement

##### Send-ConsoleSessionRefresh

##### Send-PoolRecompose

##### Send-PoolRefresh

##### Set-PoolSnapshot


### Helpers
***

##### Confirm-Entitlement

##### Confirm-PoolSnapshotOutdated

##### Confirm-ReplicaProvisioned

##### Confirm-SnapshotExists

##### Confirm-SnapshotPathExists

##### Confirm-VDIConnected

##### Get-LatestSnapshot

##### Get-OutdatedVMs

##### Get-PoolMaster

##### Get-PoolSnapshot

##### Get-PoolsWithOutdatedSnapshots

##### Show-Pools

##### Show-PoolsWithOutdatedSnapshots

##### Show-VDICommands