VMware PowerVDI PowerShell Module
=================================

Written by Nick Reed (reednj77@gmail.com)

Purpose
-------
For remotely automating common VMware View Administrator tasks.

Prerequisites
-------------
  - [VMware vSphere PowerCLI](https://my.vmware.com/web/vmware/details?downloadGroup=VSP510-PCLI-510&productId=285) installed
  - PowerShell Remoting (WinRM) enabled on View Connection Server

This module was written for and tested with VMware vSphere PowerCLI 5.0 and VMware View PowerCLI 5.0.  

Installation
----------
Copy the PowerVDI folder to one of these folders:
* `C:\Windows\System32\WindowsPowerShell\v1.0\Modules\`
* `%USERPROFILE%\Documents\WindowsPowerShell\Modules\`

Default Settings File
---------------------

Define defaults in the settings.ini file located in the module folder.  This saves you from having to type certain parameters over and over.

Sample **settings.ini** file:

```
organization=My Company
defaultvcenter=vcenter.example.com
defaultbroker=broker.example.com
defaultdomain=.example.com
domaincontrollerpath=dc=example,dc=com
```

###### Before:

```powershell
PS> Connect-VDI -vcenter vcenter.example.com -broker broker.example.com
```

###### After:

```powershell
PS> Connect-VDI

# Override the default broker, but free to omit the domain
PS> Connect-VDI broker2
```

Usage
-----
```powershell
Import-Module PowerVDI
Connect-VDI <broker fqdn> -vcenter <vcenter fqdn> [-as domain\username]
```

Run `Get-Help <command-name> -full` to get more detailed documentation.

Connecting
-----------

#### Connect-VDI <small>// _Alias:_ **cvdi**</small>

> `[-broker] <string>`
> `[[-vcenter] <string>]`
> `[[-credential | -as] <object>]`
	
Loads the VDI environment by connecting to the specified vCenter and View Connection Server instances.  This command must be run before attempting to run any other commands.

```powershell
PS> Connect-VDI -vcenter vcenter01.company.com -broker broker01.company.com

Connecting to broker (broker01.company.com)...Success
Connecting to vCenter (vcenter01.company.com)...Success
```

A number of steps can be taken to streamline this command:

* If all of your servers share a common domain, add this line to the settings.ini file:

  ```
  defaultdomain=.company.com
  ```
  With this, you can trim the command from the previous example to:
  ```powershell
  PS> Connect-VDI -vcenter vcenter01 -broker broker01
  ```

* If you only use one vCenter and/or one broker, add either of these lines to the settings.ini file:

  ```
  defaultvcenter=vcenter01.company.com
  defaultbroker=broker01.company.com
  ```
  With these, you can trim the command from the previous example to any one of these:
  ```powershell
  PS> Connect-VDI broker01
  PS> Connect-VDI -vcenter vcenter01
  PS> Connect-VDI 
  ```

**Authentication**

By default, the command will attempt to connect to the servers as the user running the shell (or script).  If that user doesn't have access, you will be prompted to enter credentials.  Alternatively, you can pass credentials directly to the command:

```powershell
PS> Connect-VDI broker01 -as domain\johnsmith  #-> will be prompted for password
```
-- or --
```powershell
PS> $johnsmith = Get-Credential domain\johnsmith  #-> will be prompted for password
PS> Connect-VDI broker01 -as $johnsmith
```
**Tip:** Store the password (securely) in a file, then set up your PowerShell profile to use it to create a credential object and store it in a variable.  Here's how:

* Save the password to a file

  ```powershell
  PS> $credential = Get-Credential domain\adminuser
  PS> $credential.Password | ConvertFrom-SecureString | Set-Content "C:\Users\johnsmith\Documents\WindowsPowerShell\adminuser.txt"
  ```

* Update your PowerShell profile script:

  ```powershell
  # C:\Users\johnsmith\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1
  $adminuser = New-Object System.Management.Automation.PsCredential("domain\adminuser", (Get-Content "C:\Users\johnsmith\Documents\WindowsPowerShell\adminuser.txt" | ConvertTo-SecureString)) 
  ```

* Now, from a new PowerShell console, you can run the following command without being prompted for a password: 

  ```powershell
  PS> Connect-VDI broker01 -as $adminuser
  ```
  
#### Disconnect-VDI <small>// _Alias:_ **dvdi**</small>

Unloads the VDI environment by disconnecting from the vCenter and View Connection Server instances.

```powershell
PS> Disconnect-VDI

Disconnecting from vcenter01.company.com...Done
Disconnecting from broker01.company.com...Done
```

#### Confirm-VDIConnected

Returns true or false depending on if the current session is connected to vSphere and the View Connection Server.  

```powershell
PS> Confirm-VDIConnected
False

PS> Connect-VDI broker01

...output omitted...

PS> Confirm-VDIConnected
True 
```


Information
-----------

#### Show-Pools <small>// _Alias:_ **shp**</small>

Outputs a list of pools on the broker.  

```powershell
PS> Show-Pools

pool_id           displayName
-------           -----------
Pool1             First Pool
Pool2             Second Pool
```

#### Show-PoolsWithOutdatedSnapshots

Outputs a list of linked-clone pools in which the snapshot set for the pool does not match the most recent snapshot on the master VM.  

```powershell
PS> Show-PoolsWithOutdatedSnapshots

Pool      Master          Current Snapshot         Latest Snapshot
----      ------          ----------------         ---------------
Pool1     MASTER-VM       /Snapshot1               /Snapshot1/Snapshot2
```

#### Show-VDICommands <small>// _Alias:_ **shcmd**</small>

Outputs a list of available commands from this module, the VMware View PowerCLI, and the VMware vSphere PowerCLI.



Entitlements
------------

#### Add-Entitlement <small>// _Alias:_ **entitle**</small>

> `[-name | -username | -user] <string>`
> `[[-pools | -pool | -pool_id] <object>]`

Entitles a user or group on the specified desktop pools, or on all pools if none are specified.

```powershell
PS> Add-Entitlement -name johnsmith -pools Pool1,Pool2

Smith, John:
   Pool1: Entitled
   Pool2: Entitled
```

#### Remove-Entitlement <small>// _Alias:_ **unentitle**</small>

> `[-name | -username | -user] <string>`
> `[[-pools | -pool | -pool_id] <object>]`

Unentitles a user or group on the specified desktop pools, or on all pools if none are specified.

```powershell
PS> Remove-Entitlement -name johnsmith -pools Pool1,Pool2

Smith, John:
   Pool1: Unentitled
   Pool2: Unentitled

PS> Remove-Entitlement -name johnsmith -pool Pool1

Smith, John:
   Pool1: Unentitled (no change)
```

#### Confirm-Entitlement

> `[-name | -username | -user] <string>`
> `[-pool | -pool_id] <object>`

Given a user or group name and a pool ID, returns true or false depending on if the user or group is entitled on the pool.

```powershell
PS> Confirm-Entitlement -user 'johnsmith' -pool 'Pool1'
True
```


Datastores
----------

#### Show-PoolDatastores

> `[-pool | -pool_id] <object>`

Displays the datastores configured for the specified pool.

```powershell
PS> Show-PoolDatastores Pool1

Name             Types        Overcommit     Path
----             -----        ----------     ----
DStore-1         {OS, data}   Aggressive     /path/to/DStore-1
DStore-2         {OS, data}   Aggressive     /path/to/DStore-2
DStore-Replica   {replica}    Conservative   /path/to/DStore-Replica
```

#### Add-PoolDatastore

> `[-datastore] <string>`
> `[[-pools | -pool | -pool_id | -to] <object>]`
> `[[-usage] <string>]`
> `[[-overcommit] <string>]`

Adds a datastore to the configuration of the specified desktop pools, or on all pools if none are specified.

Only use the name of the datastore.  The full path will be assumed to match the other datastores configured for the pool. 

The `usage` parameter defines how the datastore is to be used.  It can either be `OS,data` (as a string) or `replica`.  Defaults to `OS,data`.

The `overcommit` parameter defines the storage strategy.  It can be either `None`, `Conservative`, `Moderate`, or `Aggressive`.  Defaults to `None`.

```powershell
PS> Add-PoolDatastore DStore-3 -to Pool1 -overcommit Aggressive

   Pool1
   -----
   [Conservative,replica]      DStore-Replica
   [Aggressive,OS,data]        DStore-1
   [Aggressive,OS,data]        DStore-2
  +[Aggressive,OS,data]       +DStore-3: Success
```

#### Remove-PoolDatastore

> `[-datastore] <string>`
> `[[-pools | -pool | -pool_id | -from] <object>]`

Removes a datastore from the configuration of the specified desktop pools, or on all pools if none are specified.  Specify the datastore by name, without the path.

You will be prevented from performing a removal that will result in the pool having no datastores configured.  

```powershell
PS> Remove-PoolDatastore DStore-3 -from Pool1

   Pool1
   -----
   [Conservative,replica]      DStore-Replica
   [Aggressive,OS,data]        DStore-1
   [Aggressive,OS,data]        DStore-2
  -[Aggressive,OS,data]       -DStore-3: Success
```


Pool Management
---------------

#### Get-PoolMaster

> `[-pool | -pool_id] <object>`
> `[-Path]`

Returns the name of the master VM set for a given pool.  The `-pool` parameter can either be a pool ID as a string or a pool object.  Add the `-Path` switch to retrieve the path of the master VM instead of just the name.

```powershell
PS> Get-PoolMaster 'Pool1'
MASTER-VM-01

PS> Get-PoolMaster 'Pool1' -Path
/path/to/MASTER-VM-01

PS> Get-PoolMaster 'ManualPool1'
Error: ManualPool1 is not a linked-clone pool, and therefore does not utilize a master VM.
```

#### Get-PoolSnapshot

> `[-pool | -pool_id] <object>`	
> `[-Path]`

Returns the name of the current snapshot set for a given pool.  The `-pool` parameter can either be a pool ID as a string or a pool object.  Add the `-Path` switch to retrieve the path of the current snapshot instead of just the name.

```powershell
PS> Get-PoolSnapshot 'Pool1'
Snapshot2

PS> Get-PoolSnapshot 'Pool1' -Path
/Snapshot1/Snapshot2

PS> Get-PoolSnapshot 'ManualPool1'
Error: ManualPool1 is not a linked-clone pool, and therefore does not utilize a master VM snapshot.
```

#### Get-PoolsWithOutdatedSnapshots 

Returns an array of pool objects in which the snapshot set for the pool does not match the most recent snapshot on the master VM.

```powershell
PS> $outdated = Get-PoolsWithOutdatedSnapshots
```

#### Set-PoolSnapshot

> `[-snapshot] <object>`
> `[[-pools | -pool | -pool_id] <object>]`

Given a snapshot path, changes the snapshot on all the linked-clone pools on the broker.  For more control over which pools are affected, use the `-pools` parameter.

```powershell
PS> Set-PoolSnapshot -snapshot '/Snapshot1/Snapshot2' -pools 'Pool1'
Changing Pool1 snapshot from Snapshot1 to Snapshot2...Success
```


Pool Master VM Management
-------------------------

#### Confirm-PoolSnapshotOutdated

> `[-pool | -pool_id] <object>`

Given either a pool ID as a string or a pool object, checks if the pool's master VM has a newer snapshot than the one set for the pool. 

```powershell
PS> Confirm-PoolSnapshotOutdated 'Pool1'
True
```

#### Confirm-ReplicaProvisioned

> `[-vm] <object>`
> `[-snapshot] <object>`

Given a VM name and a snapshot name, returns true or false depending on if a replica has been provisioned for it.  This is useful when scheduling the recompose of a pool, since the provisioning of the replica will take a fair amount of time.  

```powershell
PS> Confirm-ReplicaProvisioned 'MASTER-VM-01' -snapshot 'Snapshot2'
True
```

#### Confirm-SnapshotExists

> `[-vm] <object>`
> `[-snapshot] <object>`

Given a VM name and a snapshot name, returns true or false depending on if the snapshot exists.

```powershell
PS> Confirm-SnapshotExists 'MASTER-VM-01' -snapshot 'Snapshot2'
True
```

#### Confirm-SnapshotPathExists

> `[-vm] <object>`
> `[-snapshot] <object>`

Given a VM name and a snapshot path, returns true or false depending on if the snapshot path exists.

```powershell
PS> Confirm-SnapshotPathExists 'MASTER-VM-01' -snapshot '/Snapshot1/Snapshot2'
True
```

#### Get-LatestSnapshot

> `[-vm] <object>`
> `[-Path]`

Returns the name of the latest snapshot for a given virtual machine.  Add the `-Path` switch to retrieve the path of the latest snapshot instead of just the name.  Returns false if the VM has no snapshots.

```powershell
PS> Get-LatestSnapshot 'MASTER-VM-01'
Snapshot2

PS> Get-LatestSnapshot 'MASTER-VM-01' -Path
/Snapshot1/Snapshot2

PS> Get-LatestSnapshot 'MASTER-VM-02'
Could not find any snapshots for the VM named MASTER-VM-02.
```


Linked-Clone Management
-----------------------

#### Get-OutdatedVMs

> `[[-pools | -pool | -pool_id] <object>]`
> `[[-except] <string[]>]`

Returns the linked-clones that were built off a different snapshot than the one currently set for their pool.  Defaults to checking all pools, but can be restricted with the `-pools` and/or `-except` parameters.

```powershell
PS> Get-OutdatedVMs 'Pool1','Pool2','Pool3'

Name             CurrentSnapshot          PoolSnapshot
----             ---------------          ------------
Pool1-VM-01      /Snapshot1               /Snapshot1/Snapshot2
Pool2-VM-01      /Snapshot1/Snapshot2     /Snapshot1/Snapshot2/Snapshot3

PS> Get-OutdatedVMs -except 'Pool2'

Name             CurrentSnapshot          PoolSnapshot
----             ---------------          ------------
Pool1-VM-01      /Snapshot1               /Snapshot1/Snapshot2

PS> Get-OutdatedVMs 'Pool3'

Could not find any outdated VMs.
```

#### Send-ConsoleSessionRefresh

Refreshes all VMs on the broker with connected sessions that are using the CONSOLE protocol.  

> Periodically, we see remote sessions drop into console mode when a user disconnects instead of simply logging out, where it will stay indefinitely.  It's a bug, and running this command every few minutes via a scheduled task takes care of it for us.

#### Send-PoolRecompose <small>// _Alias:_ **recompose**</small>

> `[[-pools | -pool | -pool_id] <object>]`
> `[[-except] <string[]>]`
> `[[-forceLogoff] <boolean>]`
> `[[-stopOnError] <boolean>]`
> `[-Simulate]`

Schedules the recompose of all the linked-clones in the given pools.  

**There are two main goals of the scheduling algorithm:**
* Maintain desktop availability for each pool at all times throughout the process
* Prevent overwhelming vCenter with too many tasks at once

**How it works:**

1. Each pool's clones are split into groups.
   * The number of groups is proportional to the number of clones in the pool.
     * 1 clone => 1 group
     * 2-29 clones => 2 groups
     * 30-59 clones => 3 groups
     * 60-89 clones => 4 groups
2. Corresponding clone groups from each pool are combined to form a wave.
   * The first wave contains the first group of clones from each pool, the second wave contains the second group from each pool, etc.
   * If a replica has not yet been provisioned for a pool's snapshot, an ___N___-wave delay is added after the first wave to allow time for the replica to be provisioned, where ___N___ is inversely related to the number of pools being recomposed (at least 2)
   * **Ex.** How a pool with 30 clones (3 groups) would be added to waves:
     ```
     Replica already provisioned
        Waves: [ Group 1 ] [ Group 2 ] [ Group 3 ]
     
     Replica not provisioned
        Waves: [ Group 1 ] [         ] [         ] [ Group 2 ] [ Group 3 ]
     ```
3. Each wave is scheduled.
   * The first wave begins in 5 minutes
   * For each wave, clones are recomposed in groups of 20, with each group scheduled for 10 minutes after the last
   * A delay of 15 minutes separates each wave

**Command Options**

By default, all pools are recomposed, but that can be overridden with the `-pools` and `-except` parameters.  Also by default, the recompose command will be told not to forcefully logoff current sessions and to not stop on error.  Override these by adding `-forceLogoff $true` or `-stopOnError $true` to the command.

Add the `-Simulate` switch if you want to see how your pools will be scheduled without actually taking any action.  

```
PS> Send-PoolRecompose RedPool,BluePool,GreenPool -Simulate

Note: This is a simulation.  No action will be taken.

Scheduling pools for recomposition:
   Red Pool (63)
   Blue Pool (44)
   Green Pool (19)

In 5 minutes (12:05 PM)
********************************
Red-01      Red-04      Red-07      Red-10      Red-13       Red-16       Blue-03
Red-02      Red-05      Red-08      Red-11      Red-14       Blue-01      Blue-04
Red-03      Red-06      Red-09      Red-12      Red-15       Blue-02 

In 15 minutes (12:15 PM)
********************************
Blue-05     Blue-08     Blue-11     Blue-14     Green-02     Green-05     Green-08
Blue-06     Blue-09     Blue-12     Blue-15     Green-03     Green-06     Green-09
Blue-07     Blue-10     Blue-13     Green-01    Green-04     Green-07

In 25 minutes (12:25 PM)
********************************
Green-10

In 40 minutes (12:40 PM)
********************************
Red-17      Red-20      Red-23      Red-26      Red-29       Red-32       Blue-18
Red-18      Red-21      Red-24      Red-27      Red-30       Blue-16      Blue-19
Red-19      Red-22      Red-25      Red-28      Red-31       Blue-17

In 50 minutes (12:50 PM)
********************************
Blue-20     Blue-23     Blue-26     Blue-29     Green-12     Green-15     Green-18
Blue-21     Blue-24     Blue-27     Blue-30     Green-13     Green-16     Green-19
Blue-22     Blue-25     Blue-28     Green-11    Green-14     Green-17

In 65 minutes (1:05 PM)
********************************
Red-33      Red-36      Red-39      Red-42      Red-45       Red-48       Blue-33
Red-34      Red-37      Red-40      Red-43      Red-46       Blue-31      Blue-34
Red-35      Red-38      Red-41      Red-44      Red-47       Blue-32

In 75 minutes (1:15 PM)
********************************
Blue-35     Blue-37     Blue-39     Blue-41     Blue-43
Blue-36     Blue-38     Blue-40     Blue-42     Blue-44

In 90 minutes (1:30 PM)
********************************
Red-49      Red-51      Red-53      Red-55      Red-57       Red-59       Red-61      Red-63
Red-50      Red-52      Red-54      Red-56      Red-58       Red-60       Red-62
```

#### Send-PoolRefresh <small>// _Alias:_ **refresh**</small>

> `[[-pools | -pool | -pool_id] <object>]`
> `[[-except] <string[]>]`
> `[[-forceLogoff] <boolean>]`
> `[[-stopOnError] <boolean>]`	
> `[-Simulate]`

Schedules the refresh of all the linked-clones in the given pools.  

**There are two main goals of the scheduling algorithm:**
* Maintain desktop availability for each pool at all times throughout the process
* Prevent overwhelming vCenter with too many tasks at once

**How it works:**

1. Each pool's clones are split into groups.
   * The number of groups is proportional to the number of clones in the pool.
     * 1 clone => 1 group
     * 2-29 clones => 2 groups
     * 30-59 clones => 3 groups
     * 60-89 clones => 4 groups
2. Corresponding clone groups from each pool are combined to form a wave.
   * The first wave contains the first group of clones from each pool, the second wave contains the second group from each pool, etc.
3. Each wave is scheduled.
   * The first wave begins in 5 minutes
   * For each wave, clones are refreshed in groups of 20, with each group scheduled for 5 minutes after the last
   * A delay of 10 minutes separates each wave

**Command Options**

By default, all linked-clone pools are refreshed, but that can be overridden with the `-pools` and `-except` parameters.  Also by default, the refresh command will be told not to forcefully logoff current sessions and to not stop on error.  Override these by adding `-forceLogoff $true` or `-stopOnError $true` to the command.

Add the `-Simulate` switch if you want to see how your pools will be scheduled without actually taking any action.  

```
PS> Send-PoolRefresh RedPool,BluePool,GreenPool -Simulate

Note: This is a simulation.  No action will be taken.

Scheduling pools for refresh:
   Red Pool (63)
   Blue Pool (44)
   Green Pool (19)

In 5 minutes (12:05 PM)
********************************
Red-01      Red-04      Red-07      Red-10      Red-13       Red-16       Blue-03
Red-02      Red-05      Red-08      Red-11      Red-14       Blue-01      Blue-04
Red-03      Red-06      Red-09      Red-12      Red-15       Blue-02 

In 10 minutes (12:10 PM)
********************************
Blue-05     Blue-08     Blue-11     Blue-14     Green-02     Green-05     Green-08
Blue-06     Blue-09     Blue-12     Blue-15     Green-03     Green-06     Green-09
Blue-07     Blue-10     Blue-13     Green-01    Green-04     Green-07

In 15 minutes (12:15 PM)
********************************
Green-10

In 25 minutes (12:25 PM)
********************************
Red-17      Red-20      Red-23      Red-26      Red-29       Red-32       Blue-18
Red-18      Red-21      Red-24      Red-27      Red-30       Blue-16      Blue-19
Red-19      Red-22      Red-25      Red-28      Red-31       Blue-17

In 30 minutes (12:30 PM)
********************************
Blue-20     Blue-23     Blue-26     Blue-29     Green-12     Green-15     Green-18
Blue-21     Blue-24     Blue-27     Blue-30     Green-13     Green-16     Green-19
Blue-22     Blue-25     Blue-28     Green-11    Green-14     Green-17

In 40 minutes (12:40 PM)
********************************
Red-33      Red-36      Red-39      Red-42      Red-45       Red-48       Blue-33
Red-34      Red-37      Red-40      Red-43      Red-46       Blue-31      Blue-34
Red-35      Red-38      Red-41      Red-44      Red-47       Blue-32

In 45 minutes (12:45 PM)
********************************
Blue-35     Blue-37     Blue-39     Blue-41     Blue-43
Blue-36     Blue-38     Blue-40     Blue-42     Blue-44

In 55 minutes (12:55 PM)
********************************
Red-49      Red-51      Red-53      Red-55      Red-57       Red-59       Red-61      Red-63
Red-50      Red-52      Red-54      Red-56      Red-58       Red-60       Red-62
```


Pool Backup/Restore/Copy
------------------------

#### Export-Pools <small>// _Alias:_ **export-pool**</small>

> `[-path | -file] <string>`
> `[[-pools | -pool | -pool_id] <object>]`

Exports pool settings, desktops (where applicable), and entitlements to an XML file.  The file can be used to recreate the pools (see: Import-Pools).  Omit the -pools parameter to export all pools on the broker.

```powershell
PS> Export-Pools Pools.xml -pools Pool1,Pool2

Gathering pools...
Gathering VM info...
   Pool1...Success
   Pool2...Success

Exporting to Pools.xml...Success
```

#### Import-Pools <small>// _Alias:_ **import-pool**</small>

> `[-path | -file] <string>`
> `[[-pools | -pool | -pool_id] <object>]`
> `[[-settings] <hashtable>]`

Imports pool information from an XML file and creates each one on the broker.  Once the settings have been read from the file, you will be prompted to confirm that they are accurate before creating each pool.  

```
PS> Import-Pools Pool1.xml

Pool1

Name                          Value
----                          -----
allowMultipleSessions         False
allowProtocolOverride         True
autoLogoffTime                Never
composer_ad_id                1a2b3c4d-1a2b-1a2b-1a2b-1a2b3c4d5e6f
datastoreSpecs                [Conservative,replica]/Path/to/datastore;[Conservative,OS,...
defaultProtocol               PCOIP
deletePolicy                  Default
disabled                      True
flashQuality                  NO_CONTROL
flashThrottling               DISABLED
folderId                      Public
headroomCount                 1
isProvisioningEnabled         True
isUserResetAllowed            False
maximumCount                  1
minimumCount                  1
namePrefix                    Pool1-VM-{n:fixed=2}
organizationalUnit            OU=Pool1,OU=VDI
parentSnapshotPath            /path/to/snapshot
parentVMPath                  /path/to/master
persistence                   NonPersistent
pool_id                       Pool1
powerPolicy                   RemainOn
refreshPolicyType             Never
resourcePoolPath              /path/to/resource/pool
suspendProvisioningOnError    True
tempDiskSize                  4096
useTempDisk                   True
vc_id                         1a2b3c4d-1a2b-1a2b-1a2b-1a2b3c4d5e6f
vmFolderPath                  /path/for/vms

Pool Creation
Create the pool with the settings listed above?
[Y] Yes  [N] No  [?] Help (default is "Y"): Y

Creating new pool Pool1...Success
Copying entitlements...
   Entitling Smith, John...Success
   Entitling VDI Group...Success
```

The settings listed will be different depending on the type of pool it is.  You can override any of these settings by passing a hash to the function.  For example:

```powershell
PS> Import-Pool Pool1.xml -settings @{"defaultProtocol" = "RDP"; "disabled" = $false}
```

Due to the limitations of VMware's PowerCLI, automatic pools with a manual naming scheme can not be created with this command.  Also, a handful of pool settings can only be set though the View Administrator console.  These settings include:

* Connection Server restrictions
* Windows 7 3D Rendering settings
* Max number of monitors
* Max resolution of any one monitor
* Automatic user assignment
* ThinApp assignments

#### New-PoolFromCopy <small>// _Alias:_ **cppool**</small>

> `[-source] <object>`
> `[-pool_id] <string>`
> `[[-description] <string>]`
> `[[-displayName] <string>]`
> `[[-namePrefix] <string>]`
> `[[-settings] <hashtable>]`

Given an existing pool to use as a template, a new pool will be created and entitled.  The pool creation logic is the same as the `Import-Pools` command.  The difference is that since this is a copy, you must specify a pool ID, display name, and description to prevent conflicts.  The pool's display name and description will not be copied, instead being left blank unless you specify them using the `-displayName` and `-description` parameters.  You must also specify a naming scheme for automatic linked-clone pools.  As with the `Import-Pools` command, you can override specific settings via the `-settings` parameter.  

**Note** that you will be shown the settings and prompted for confirmation before the pool is created.

```powershell
PS> New-PoolFromCopy -source 'Pool1' -pool_id 'Pool2' -namePrefix 'Pool2-VM-{n:fixed=2}' -settings @{"disabled" = $true}

Pool2

Name                          Value
----                          -----
allowMultipleSessions         False
allowProtocolOverride         True
autoLogoffTime                Never
composer_ad_id                1a2b3c4d-1a2b-1a2b-1a2b-1a2b3c4d5e6f
datastoreSpecs                [Conservative,replica]/Path/to/datastore;[Conservative,OS,...
defaultProtocol               PCOIP
deletePolicy                  Default
disabled                      True
flashQuality                  NO_CONTROL
flashThrottling               DISABLED
folderId                      Public
headroomCount                 1
isProvisioningEnabled         True
isUserResetAllowed            False
maximumCount                  1
minimumCount                  1
namePrefix                    Pool2-VM-{n:fixed=2}
organizationalUnit            OU=Pool2,OU=VDI
parentSnapshotPath            /path/to/snapshot
parentVMPath                  /path/to/master
persistence                   NonPersistent
pool_id                       Pool2
powerPolicy                   RemainOn
refreshPolicyType             Never
resourcePoolPath              /path/to/resource/pool
suspendProvisioningOnError    True
tempDiskSize                  4096
useTempDisk                   True
vc_id                         1a2b3c4d-1a2b-1a2b-1a2b-1a2b3c4d5e6f
vmFolderPath                  /path/for/vms

Pool Creation
Create the pool with the settings listed above?
[Y] Yes  [N] No  [?] Help (default is "Y"): Y

Creating new pool Pool2...Success
Copying entitlements...
   Entitling Smith, John...Success
   Entitling VDI Group...Success
```

The settings listed will be different depending on the type of pool it is.

Questions or Problems?
----------------------

If you have any issues with this module which you cannot find the solution to in the documentation, please add an [issue on GitHub](https://github.com/reed/vmware-powervdi/issues) or fork the project and send a pull request.
