# ZFS tunning shorthand

```shell
echo 'options zfs zfs_arc_max=4294967296
options zfs zfs_arc_min=1073741824' > /etc/modprobe.d/zfs.conf
```

```shell
update-initramfs -u
```

```shell
echo 1073741824 > /sys/module/zfs/parameters/zfs_arc_min
echo 4294967296 > /sys/module/zfs/parameters/zfs_arc_max
```

```shell
zfs set compression=off rpool
zfs set atime=off rpool
```