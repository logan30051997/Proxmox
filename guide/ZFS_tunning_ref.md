# ZFS tunning

## Short summary
    1. Limit ARC min/max size
    2. Turn off atime on all zpool
    3. Turn off compression on all zpool

## 1. ZFS ARC size
We aggressively limit the ZFS ARC size, as it has led to **several spontaneous reboots** in the past when left unlimited. Basically, we add up all the memory the system uses without caches and buffers (like all the KVM maximum RAM combined), subtract that from total host RAM, and set the ARC to something a bit less than that, so it has to compete with system cache only. For example: on a 32GB server the maximum RAM allocation of KVM guests is 26 GB, so we set the ARC to max out at 4GB (leaving 2GB for anything else). We also set a lower limit of 1GB to the ARC, as it has been reported that it helps performance.  
  
To do that, you have add the following lines to `/etc/modprobe.d/zfs.conf`

```
options zfs zfs_arc_max=4294967296
options zfs zfs_arc_min=1073741824
```

and after that run:  

```
update-initramfs -u
```

and reboot.  
  
Looking at the ARC of this very server with `arc_summary.py` you can see it stays between the limits:  

    ARC Size:                              30.72%  1.54    GiB
           Target Size: (Adaptive)         30.72%  1.54    GiB
           Min Size (Hard Limit):          20.00%  1.00    GiB
           Max Size (High Water):          4:1     4.00    GiB
    
    ARC Size Breakdown:
           Recently Used Cache Size:       35.27%  554.85  MiB
           Frequently Used Cache Size:     64.73%  1018.10 MiB


## 2. SWAP on ZFS zvol
You also have to make sure that swap behaves well if it resides on a ZFS zvol (default Proxmox installation places it there). Most important is disabling ARC caching the swap volume, but the other tweaks are important as well (and endorsed by the ZFS on Linux community):  
https://github.com/zfsonlinux/zfs/wiki/FAQ
  
Execute these commands in your shell (left out the # so you can copy all lines at once):  

```shell
zfs set primarycache=metadata rpool/swap
zfs set secondarycache=metadata rpool/swap
zfs set compression=zle rpool/swap
zfs set checksum=off rpool/swap
zfs set sync=always rpool/swap
zfs set logbias=throughput rpool/swap
```
  
You can verify these settings by running:  

```shell
zfs get all rpool/swap
```


## Some Basic Tuning
A few general procedures can tune a ZFS filesystem for performance, such as disabling file access time updates in the file metadata. Historically, filesystems have always tracked when a user or application accesses a file and logs the most recent time of access, even if that file was only read and not modified. This activity can affect metadata performance when updating this field. To avoid this unnecessary I/O, simply turn off the atime parameters:

```shell
zfs set atime=off myvol
```
To verify that it has been turned off, use the zfs get atime command:

```shell
zfs get atime myvol

NAME   PROPERTY  VALUE  SOURCE
myvol  atime     off    local
```
Another parameter that can affect performance is compression, and although some algorithms (e.g., LZ4) are known to perform extremely well, it still sucks up a bit of CPU time compared with its counterparts. Therefore, disable filesystem compression,

```shell
zfs set compression=off myvol
```
and verify that compression has been turned off:

```shell
sudo zfs get compression myvol
NAME   PROPERTY     VALUE     SOURCE
myvol  compression  off       default
```
To view all available parameters, use zfs get all (Listing 3).

Listing 3: View Parameters

```shell
zfs get all myvol
NAME   PROPERTY              VALUE                  SOURCE
myvol  type                  filesystem             -
myvol  creation              Sat Feb 22 22:09 2020  -
myvol  used                  471K                   -
[ ... ]
```

Ref: https://www.admin-magazine.com/HPC/Articles/Tuning-ZFS-for-Speed-on-Linux