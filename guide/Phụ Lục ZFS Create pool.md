## 3. Tạo phân vùng ZFS cho 1 ổ đĩa còn lại.

1. Format ổ đĩa trên shell

Kiểm tra các block sử dụng lệnh lsblk

```
root@pve2:~# lsblk

NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0 279.5G  0 disk
├─sda1   8:1    0  1007K  0 part
├─sda2   8:2    0   512M  0 part
└─sda3   8:3    0   279G  0 part
sdb      8:16   0 894.3G  0 disk
├─sdb1   8:17   0 894.3G  0 part
└─sdb9   8:25   0     8M  0 part
sdc      8:32   0   1.8T  0 disk
├─sdc1   8:33   0   1.8T  0 part
└─sdc9   8:41   0     8M  0 part
sr0     11:0    1  1024M  0 rom

```

Tại đây:  sda đã sử dụng cài đặt OS => sdb, sdc lưu trữ dữ liệu. Cần format
         
Format disk như sau:

```
root@pve2:~# fdisk /dev/sdc
```

Command P: print ra partition đã có trên ổ đĩa.
Tại đây thấy có partition sdc1, sdc9

```
Command (m for help): p
Disk /dev/sdb: 894.3 GiB, 960197124096 bytes, 1875385008 sectors
Disk model: Micron_5100_MTFD
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: B8FB76E2-FE67-C64D-AF7B-513104A0C536

Device          Start        End    Sectors   Size Type
/dev/sdc1        2048 1875367935 1875365888 894.3G Solaris /usr & Apple ZFS
/dev/sdc9  1875367936 1875384319      16384     8M Solaris reserved 1

```


Xoá partion gõ tiếp command d

```

Command (m for help): d
Partition number (1,9, default 9):

Partition 9 has been deleted.

Command (m for help): d
Selected partition 1
Partition 1 has been deleted.

```
Sau khi xoá xong ấn p để hiển thị như bên dưới là đã xoá hoàn toàn.

```
Command (m for help): p
Disk /dev/sdb: 894.3 GiB, 960197124096 bytes, 1875385008 sectors
Disk model: Micron_5100_MTFD
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: B8FB76E2-FE67-C64D-AF7B-513104A0C536

```

Lưu lại các thao tác vừa làm 

```

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

```

 **Đặc biệt lưu ý khi sử dụng lệnh fdisk, chỉ format khi chưa có dữ liệu chạy thật của hệ thống.**

Kiểm tra lại bằng lệnh

```
lsblk
```

> Sau khi format disk, tạo ZFS trên shell hoặc giao diện

Cách 1: Tạo phân vùng ZFS trên shell

- List các zpool:

```
zpool list

```

- Xoá phân vùng zfs neu can:

```
zpool destroy [ten zfs]
```

- Tạo phân vùng zfs cho ổ sata:

```
zpool create -f -o ashift=12 zpoolsata /dev/sdc
``` 


** Lưu ý đặt tên zfs pool trên các máy giống nhau.

Cách 2: Tạo ZFS trên giao diện để hiển thị


- Vào phần pve1/Disk để Initialize Disk with GPT
- Vào tiếp phần pev1/Disk/ZFS => Create ZFS 
- Tạo zfs cho sdc
- Phần giao diện có thể sẽ có lỗi nên thao tác trên dòng lệnh bằng shell