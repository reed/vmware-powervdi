### 1.1.0 (1 November 2012)
  - Added `defaultbroker` setting
  - Added `domaincontrollerpath` setting
  - Better error handling when connecting to broker
  - Better handling of non-linked-clone pools
  - `Get-LatestSnapshot` fix for when only one snapshot exists
  - `Confirm-SnapshotExists` prevent error output to console
  - `Confirm-SnapshotPathExists` handle case where VM has no snapshots
  - `Send-PoolRecompose` retrieve all clones in one call instead of once for each pool
  - `Send-PoolRefresh` retrieve all clones in one call instead of once for each pool

### 1.0.0 (24 October 2012)
  - Added `Export-Pools`
  - Added `Import-Pools`
  - Added support for all pool types to `New-PoolFromCopy`
  - Added settings.ini file

### 0.1.0 (27 September 2012)
  - First release