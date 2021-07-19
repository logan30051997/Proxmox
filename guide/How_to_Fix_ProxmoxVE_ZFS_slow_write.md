# How to Fix Proxmox VE/ZFS Pool slow write performance issue

Last Updated on 17 September, 2020  
Ref: https://dannyda.com/2020/05/24/how-to-fix-proxmox-ve-zfs-pool-extremely-slow-write/

<!-- toc -->

- [The Issue](#The-Issue)
- [The Fix](#The-Fix)
- [Bonus](#Bonus)
  * [Read/Write buffer/cache & Dedicate ZIL device for ZFS pool](#ReadWrite-buffercache--Dedicate-ZIL-device-for-ZFS-pool)
  * [How big should the ZIL/SLOG device/HDD/SSD be?](#How-big-should-the-ZILSLOG-deviceHDDSSD-be)
  * [How to Remove/Delete/Replace ZIL/SLOG device/HDD/SSD?](#How-to-RemoveDeleteReplace-ZILSLOG-deviceHDDSSD)
  * [How to find out real device name from Proxmox?](#How-to-find-out-real-device-name-from-Proxmox)
  * [How to add read cache disks?](#How-to-add-read-cache-disks)
- [References](#References)

<!-- tocstop -->

## The Issue

If we just create the ZFS pool from Proxmox gui, then start to use it. (Especially for HDDs)

e.g. We write large datasets continuously.

Sooner or later (Depend on the ZFS pool usage), we will find out that the writes is around 1-10MB/s, which is extremely slow.

(Note: If we test the newly created pool with and without ZIL/SLOG, there probably won’t be much difference or even slower with ZIL/SLOG deice attached, after we have data filled in the ZFS pool, the pool with dedicate ZIL/SLOG device will perform better than the pool without dedicate ZIL/SLOG device)

## The Fix

This can happen due to ZFS Intent Log (ZIL)/Separate ZFS Intent Log (SLOG) is getting written to the same ZFS data pool which all our data are stored, which eventually caused “double write” issue.

To fix this issue is easy, best way to fix it properly is to grab a SSD, worst case, if we do not have one temporary, we can even grab a 5400RPM or 7200RPM HDD use it via HBA/SATA/SCSI or even USB 3.0 (USB is not suitable for long term for this purpose, but can work fine as an temporary solution/fix).

Once attached the disk to the Proxmox host, note down the device name e.g. /dev/sde, /dev/sdf etc.

Login to terminal from Proxmox host or via SSH or via Shell from web gui.

Use following command to use an dedicated HDD/SSD for ZIL/SLOG purpose

```shell
# For single HDD/SSD
zpool add -f [pool name] log [device name]
# e.g.
zpool add -f rpool log /dev/sdd
 
# For mirrored ZIL/SLOG
zpool add -f [pool name] log mirror [device 1 name] [device 2 name]
# e.g.
zpool add -f rpool log mirror /dev/sdd /dev/sde
```

Now if we have a look at write performance, it will be increased

Note: Best device for ZIL/SLOG is to have an datacenter grade SSD via HBA, so that we can get best performance

To check the pool status see if the ZIL/SLOG device is added use following commands

```shell
zpool status
```

## Bonus

### Read/Write buffer/cache & Dedicate ZIL device for ZFS pool

With ZFS pool

*   SLOG is used to cache synchronous ZIL data (Write) (before flushing to disk, Only in case of a crash occurred. In other words, ZIL only gets read if there is a crash and the data in the RAM was gone before been written to the array, so that those missing data from the RAM which didn’t get a chance to be written to the array will then be read from ZIL and written to the array) (**Write performance related**) [1]  
    **Note**: ZIL is not a cache, but rather a safety mechanism, the fact that using a dedicate storage for ZIL will improve the write performance is due to that we have removed/reduced impact of write amplification, without dedicate ZIL storage we will have 2x write operations happening at the same time, one is for data another is for ZIL, on the same storage, if we have two different storage for each purpose, each storage will gets 1x write operation for their own purpose, thus faster writing speed.

*   Adaptive Replacement Cache (ARC) and Second level adaptive replacement cache (L2ARC) are used to cache reads (**Read performance related**)

![image-22.png](:storage/66c548aa-f1f7-4d7b-97dc-c926b3fe2fb0/ef2aa216.png)
Graphical depiction of the Hybrid Storage Pool architecture that illustrates dynamic storage tiering [2]

![image-23.png](:storage/66c548aa-f1f7-4d7b-97dc-c926b3fe2fb0/a6720b26.png)
Comparing Oracle ZFS Storage Appliance and traditional storage architecture [1]

### How big should the ZIL/SLOG device/HDD/SSD be?

Usually the ZIL is default to flush every 5 seconds or when it reaches capacity, which means a SLOG that holds 5-10 seconds worth of the pool maximum throughput will be find, unless we are doing something extreme/special.

### How to Remove/Delete/Replace ZIL/SLOG device/HDD/SSD?

To remove ZIL/SLOG device
`zpool remove [pool name] [device name]`

```shell
# e.g.
zpool remove rpool /dev/sdd
# or
zpool remove rpool sdd
# or
# By using real device name
zpool remove rpool ata-xxxx_xxxx_Xxxx_x.....
```
To replace, simply remove the current HDDs/SSDs then add new disks again

### How to find out real device name from Proxmox?

Refer to this guide: [How to: Find drive name (real name) for /dev/sdb /dev/sdc from Proxmox (PVE)](https://dannyda.com/2020/05/21/how-to-find-drive-name-real-name-for-dev-sdb-dev-sdc-from-proxmox-pve)

### How to add read cache disks?

It is very similar to adding ZIL/SLOG drives

`zpool add -f [pool name] cache [device name]`
```shell
# e.g.
zpool add -f rpool cache /dev/sde
``` 
Remove cache disk
`zpool remove [pool name] [device name]`
```shell
zpool remove rpool /dev/sde
zpool remove rpool /sde
```
If the above remove command does not work, try remove the ZIL/SLOG first

## References

1. “Why Oracle ZFS Storage Appliance Optimizes Storage in Virtualized Environments”, _Oracle.com_, 2013\. \[Online\]. Available: https://www.oracle.com/technetwork/server-storage/sun-unified-storage/documentation/why-zfssa-for-virtual-92112-1851450.pdf

2. “Architectural Overview of the Oracle ZFS Storage Appliance”, _Oracle.com_, 2020\. \[Online\]. Available: https://www.oracle.com/technetwork/server-storage/sun-unified-storage/documentation/o14-001-architecture-overview-zfsa-2099942.pdf