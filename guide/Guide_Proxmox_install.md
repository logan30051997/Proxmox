# Hướng dẫn cấu hình Proxmox
## 1. Tạo USB Boot 

* Link download: [Proxmox VE 6.2 ISO Installer](https://www.proxmox.com/en/downloads/item/proxmox-ve-6-2-iso-installer)

* Tạo usb boot trên Mac: dd if=/Volumes/Data/ngocbob/Downloads/proxmox-ve_6.2-1.iso of=/dev/rdisk2 bs=8k
* Tạo usb boot trên Window: [balenaEtcher - Flash OS images to SD cards & USB drives](https://www.balena.io/etcher/)

## 2. Cài đặt Proxmox

Cài đặt proxmox trên 1 ổ đĩa (/dev/sda)

## 3. Cài đặt Cluster proxmox

Mô tả: 
         pve1: 10.20.20.200 -> Cluster master
         pve2: 10.20.20.201 -> Join cluster pve1
         pve3: 10.20.20.202 -> Join cluster pve1
         
Tạo cluster trên pve1, từ pve2 và pve3 join vào cluster đã tạo trên pve1

###  Thực hiện trên cả 3 node

  - Ping 10.20.20.200 -> Kiểm tra pve1 đã thông với pve2, pve3
  - Nano etc/hosts -> Thêm địa chỉ ip vào file host
  
  
  ```
    $ nano etc/hosts
  
    10.20.20.200 pve1.linex.hn pve1
    10.20.20.201 pve2.linex.hn pve2
    10.20.20.202 pve3.linex.vn pve3
 
 ``` 
 
* Ping domain kiểm tra xem ping đã thông
    
    ```
    ping pve2
    ping pve3
    ping pve1
    ```
### Thực hiện trên pve1
 
* Tạo cluster cú pháp: pve create [tên_cluster]

Ví dụ:
 
 ```
  pvecm create clusterpve
 
 ```
 
* Kiểm tra trạng thái cluser
 
 ``` 
  pvecm status
 ```

 **Trường hợp tạo lỗi cần phải xoá Cluster sử dụng câu lệnh dưới đây:
 DELETE CLUSTER: Xoá Cluster**

```
systemctl stop pve-cluster corosync

pmxcfs -l

rm -r /etc/corosync/*

rm /etc/pve/corosync.conf

killall pmxcfs

systemctl start pve-cluster

```

### Thực hiện trên pve2


 * Join vào cluster *clusterpve* đã tạo trên pve1 bằng cú pháp:  pvecm add IP-ADDRESS-CLUSTER

IP-ADDRESS-CLUSTER: IP của pve1 đã tạo cluster

```

pvecm add 10.20.20.200

```

### Cấu hình trên pve3

```
 pvecm add 10.20.20.200
 
```

## 2. Cấu hình bond network trên 3 nodes proxmox 

- Thực hiện trên giao diện:

Click lựa chọn từng node -> System -> Network

Tại đây thấy các card mạng đã có và vmbr0.
Xác định các port cần bond.
Mô hình bond:

```
                                            vmbr1
                                             ||               
                       vmbr0                bond1
                        ||                   ||
                       bond0    ----------------------------
                        ||      |                          |
                 |---------------------------|             |
                 |              |            |             |
                ens0           ens1       ens2           ens3
```

Sau khi cài đặt vmbr0 đã được slave 1 port.

1. Edit vmbr0
    - Xoá port slave

2. Tạo Bond 
    -  Create Bond 
     
3. Cấu hình bond
 
     - Slave 2 card mạng (vd enp11s0 enp6s0)
     - Mode: LACP (802.3ad)

4. Edit vmbr0 
     - Phần Bridge ports: thay thành bond0
   
5. Reboot máy sau khi cấu hình
  
Thực hiện tương tự với các nodes proxmox còn lại. (Nếu cần với bond1)  
  
## 3. Tạo phân vùng LVM cho ổ đĩa /dev/sdb 

Kiểm tra volume

```
lvs
```

Xoá volume có sẵn

```
vgdisplay

vgremove [tên volume]
```


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
root@pve2:~# fdisk /dev/sdb
```

Command P: print ra partition đã có trên ổ đĩa.
Tại đây thấy có partition sdb1, sdb9

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
/dev/sdb1        2048 1875367935 1875365888 894.3G Solaris /usr & Apple ZFS
/dev/sdb9  1875367936 1875384319      16384     8M Solaris reserved 1

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

> Thực hiện tương tự với /dev/sdc

 **Đặc biệt lưu ý khi sử dụng lệnh fdisk, chỉ format khi chưa có dữ liệu chạy thật của hệ thống.**

Kiểm tra lại bằng lệnh

```
lsblk
```

> Sau khi format disk, tạo lvm như sau:

Tạo partition 1 cho /dev/sdb

```
sgdisk -N 1 /dev/sdb

hoặc sử dụng fdisk

```

Tạo physical volume

```
pvcreate /dev/sdb1

```

Tạo volume group vg1 trên partition sdb1

```

vgcreate -f vg1 /dev/sdb1

```

Tạo logical volume để trống size 5% so với dung lượng thật của ổ đĩa (850G) nhằm dự phòng trường hợp pool đầy có thể tự động extend tránh lỗi dữ liệu

```
lvcreate --chunksize 512k --poolmetadatasize 128m --poolmetadataspare y --profile thin-performance -L850g -T vg1/ssdpool
```

--- Đối với lỗi hiển thị giao diện không show pool đã tạo ( không cần sửa file /etc/pve/storage.cf) 

Giao diện proxmox -> Datacenter -> Storage -> ADD LVM thin -> Thêm disk lvm đã tạo -> Tích chọn pve cần show



** Lưu ý đặt tên lvm pool trên các máy giống nhau.


## 4. Tối ưu ZFS cho OS proxmox

Lần lượt thực hiện các bước sau

Đặt max RAM cho arc zfs là 4GB, min RAM cho arc zfs là 1GB
    
```
echo 'options zfs zfs_arc_max=4294967296
options zfs zfs_arc_min=1073741824' > /etc/modprobe.d/zfs.conf
```

Update lại nội dung vừa cập nhật

```
update-initramfs -u
```
Sau khi update lại nội dung cần phải reboot lại máy để nhận cấu hình mới. Nhưng có thể cập nhật luôn bằng 2 câu lệnh dưới đây mà không cần phải reboot lại máy.

```
echo 1073741824 > /sys/module/zfs/parameters/zfs_arc_min
echo 4294967296 > /sys/module/zfs/parameters/zfs_arc_max
```

Tắt nén và atime cho rpool zfs

```
zfs set compression=off rpool
zfs set atime=off rpool
```

## 5. Cài đặt máy ảo

Tại giao diện proxmox 

=> Create VM

** Lưu ý**

Hard disk bus: SCSI
CPU: 1 socket



## 6. Cài đặt Cloud-init trên máy ảo ubuntu và centos

- Kiểm tra Internet trên máy cần cài đặt (ping 8.8.8.8, ping google.com)
- Nếu máy không có internet, kiểm tra lại cài đặt mạng. 

1. Cài đặt trên máy ảo

* Trên CentOS

Update lại OS sau khi mới cài đặt xong

```
yum install update
yum install epel-release

```      
Cài đặt cloud-init

```
yum -y install cloud-init 

```   
 
Kiểm tra trạng thái cloud-init sau khi cài đặt

```    
systemctl status cloud-init

```

Để khởi động cloud-init trong trường hợp dịch vụ cloud-init chưa được bật

```
systemctl start cloud-init

```

* Trên Ubuntu

Update lại OS sau khi mới cài đặt xong

```
apt-get -y update

```
Cài đặt cloud-init

```

apt-get -y install cloud-init

```

** Khi cài IP không đổi thì remove và cài đặt lại**


```
apt remove cloud-init
apt purge cloud-init
rm -rf /var/lib/cloud/
rm -rf /etc/cloud/
apt install cloud-init

```

 2.  Cài đặt cloud-init  giao diện Proxmox

  Chọn máy ảo cần cài 
   +  Vào phần Hardware -> Add "Cloudlnit Drive"
   +  Chọn bus/device: scsi - giá trị: 1  
      "ssdzfs:vm-105-cloudinit,media=cdrom"
   +  Chọn tab Cloud-init -> điền thông tin -> Regenerate Image 
   +  Tắt máy -> khởi động lại -> gõ ip r -> ra đúng ip vừa đặt là thành công.




## 7. Cài đặt 1 máy ảo cluster-control và 2 máy ảo mariadb



Sau khi cài đặt máy ảo thành công.

Sử dụng 1 máy ảo CentOS 7 cài đặt cluster-control và 2 máy CentOS 7 cài đặt mariadb + galera


#### 1. Đặt hostname cho máy ảo



Cấu hình trên máy cluster control


```
hostnamectl set-hostname clusterctl
```

Cấu hình trên máy mariadb 1


```
hostnamectl set-hostname mariadb1
```

Cấu hình trên máy mariadb 2


```
hostnamectl set-hostname mariadb2
```

Sửa file host trên 3 máy ảo

```
vi /etc/hosts
```
       
```
10.20.20.168 mariadb1
10.20.20.167 mariadb2
10.20.20.170 clusterctl
```

ping kiểm tra

```
ping mariadb1
ping clusterctl
ping mariadb2
```

Cài đặt phần mềm phụ trợ

```
yum install wget net-tools
yum install epel-release
```


#### 2. Cài đặt mariadb trên 2 máy mariadb1 + mariadb2

Sửa file repo cài đặt mariadb

```
vi /etc/yum.repos.d/mariadb.repo
```

```
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.4/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1

```

Enable repo:

```
yum makecache --disablerepo='*' --enablerepo='mariadb'

```

Thành công sẽ báo output như sau

```
Output
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
mariadb                                                                                                                                                                              | 2.9 kB  00:00:00
(1/3): mariadb/primary_db                                                                                                                                                            |  43 kB  00:00:00
(2/3): mariadb/other_db                                                                                                                                                              | 8.3 kB  00:00:00
(3/3): mariadb/filelists_db                                                                                                                                                          | 238 kB  00:00:00
Metadata Cache Created
```

Cài đặt Mariadb

```
yum install MariaDB-server MariaDB-client MariaDB-backup
```

Backup lại file cấu hình

```
cp /etc/my.cnf.d/server.cnf /etc/my.cnf.d/server.cnf.bak
```

Mở port tường lửa

```
sudo firewall-cmd --permanent --zone=public --add-port=3306/tcp
sudo firewall-cmd --permanent --zone=public --add-port=4567/tcp
sudo firewall-cmd --permanent --zone=public --add-port=4568/tcp
sudo firewall-cmd --permanent --zone=public --add-port=4444/tcp
sudo firewall-cmd --permanent --zone=public --add-port=4567/udp

```
```
sudo firewall-cmd --permanent --zone=public --add-source=10.20.20.167/24
sudo firewall-cmd --permanent --zone=public --add-source=10.20.20.168/24

```
Reload lại firewall

```
sudo firewall-cmd --reload

```


Hoặc đơn giản là tắt tường lửa và SE policy và reboot máy. init 6: reboot


```
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
systemctl stop firewalld
systemctl disable firewalld
init 6

```


##### 2.1. Sửa file cấu hình tại máy mariadb1 10.20.20.168

**Lưu ý: wsrep_cluster_address : địa chỉ máy bên cạnh trước, địa chỉ máy đang sửa sau.**


Sửa file cấu hình như sau:

```
echo > /etc/my.cnf.d/server.cnf

vi /etc/my.cnf.d/server.cnf
```

Nội dung file cấu hình:

```
[server]

skip-character-set-client-handshake
character-set-server = utf8
collation-server = utf8_unicode_ci

#innodb_file_format=Barracuda
#innodb_file_per_table=ON
log-error = /var/log/mysql/error.log
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 5

[mysqld]
#bind-address=10.20.20.168
#log_error=/var/log/mariadb.log

#
# * Galera-related settings
#
[galera]
# Mandatory settings
wsrep_on=ON
wsrep_provider=/usr/lib64/galera-4/libgalera_smm.so
wsrep_cluster_address="gcomm://10.20.20.167,10.20.20.168"
binlog_format=row
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2

#Cluster name
wsrep_cluster_name="portal_cluster"

#
# Allow server to accept connections on all interfaces.
#
bind-address=10.20.20.168
#
# Optional setting
#wsrep_slave_threads=1
#innodb_flush_log_at_trx_commit=0

wsrep_node_name="node1"
wsrep_sst_method=rsync
```

Tạo file log 

```
mkdir /var/log/mysql
touch /var/log/mysql/slow.log
touch /var/log/mysql/error.log
chown mysql:mysql -R /var/log/mysql/
```

Kiểm tra lại file log


```
ls -l /var/log/mysql/
```


##### 2.2. Sửa file cấu hình tại máy mariadb2 10.20.20.167


Sửa file cấu hình như sau:

```
echo > /etc/my.cnf.d/server.cnf

vi /etc/my.cnf.d/server.cnf
```
Nội dung file cấu hình


**Lưu ý: wsrep_cluster_address : địa chỉ máy bên cạnh trước, địa chỉ máy đang sửa sau.**

```
[server]

skip-character-set-client-handshake
character-set-server = utf8
collation-server = utf8_unicode_ci

#innodb_file_format=Barracuda
#innodb_file_per_table=ON
log-error = /var/log/mysql/error.log
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 5

[mysqld]
#bind-address=10.20.20.167
#log_error=/var/log/mariadb.log

#
# * Galera-related settings
#
[galera]
# Mandatory settings
wsrep_on=ON
wsrep_provider=/usr/lib64/galera-4/libgalera_smm.so
wsrep_cluster_address="gcomm://10.20.20.168,10.20.20.167"
binlog_format=row
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2

#Cluster name
wsrep_cluster_name="portal_cluster"

#
# Allow server to accept connections on all interfaces.
#
bind-address=10.20.20.167
#
# Optional setting
#wsrep_slave_threads=1
#innodb_flush_log_at_trx_commit=0

wsrep_node_name="node2"
wsrep_sst_method=rsync

```

Tạo file log 

```
mkdir /var/log/mysql
touch /var/log/mysql/slow.log
touch /var/log/mysql/error.log
chown mysql:mysql -R /var/log/mysql/
```

Kiểm tra lại file log


```
ls -l /var/log/mysql/
```



##### 2.3 Thực hiện thao tác tạo cluster trên mariadb1

Tạo galera cluster

```
galera_new_cluster
```

Kiểm tra port sql

```
lsof -i:4567
lsof -i:3306
```

Start mariadb

```
systemctl start mariadb
systemctl enable mariadb

```

Kiểm tra số node trong cluster

```
mysql -u root -e "SHOW STATUS LIKE 'wsrep_cluster_size'"
```
```
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_cluster_size | 1     |
+--------------------+-------+
```


##### 2.4. Thực hiện thao tác tạo cluster trên mariadb2

Bật mariadb

```
systemctl start mariadb
systemctl enable mariadb
```
Kiểm tra số node trong cluster

```
mysql -u root -e "SHOW STATUS LIKE 'wsrep_cluster_size'"
```
```
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_cluster_size | 2     |
+--------------------+-------+
```


>> Sau khi tạo xong cluster muốn restart lại mysql cần thực hiện lần lượt trên từng máy


#### 3. Cài đặt cluster control

**Chèn SSH key sang máy clustercontrol**

Tạo SSH key không đặt passphase

```
ssh-keygen -b 3072
```
```
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:A6QwrdLioUPZhcyEPboxLuRx1M3H32kdh25cRhTXs04 root@mariadb1
The key's randomart image is:
+---[RSA 3072]----+
|  Ooo + .      =*|
| . X.= o o    .o*|
| .=.+ . . . .ooo=|
|oXoo   .   . ++E |
|BoB     S   ..o  |
|+=       .     . |
|..               |
|                 |
|                 |
+----[SHA256]-----+
```

Add key sang máy mariadb1 và mariadb2

```
ssh-copy-id 10.20.20.167
ssh-copy-id 10.20.20.168

```

ssh thử sang máy mariadb1 và mariadb2

```
ssh mariadb1
exit
```

Nếu ssh được thì thoát và tiếp tục cài đặt phần dưới đây.

** Cài đặt cluster control**

```
yum install epel-release
yum update
```

Tải về và cài đặt 

```
 wget --no-check-certificate https://severalnines.com/downloads/cmon/install-cc
 ls -la
 chmod +x install-cc
 ./install-cc
```

Trong lúc cài đặt hệ thống hỏi:

```
=> The Controller hostname will be set to 10.20.20.170. Do you want to change it? (y/N): y
=> Enter the hostname: clusterctl
```

```
=> Importing the Web Application DB schema and creating the cmon user.
=> Enter your MySQL root user's password: Enter password:

=> Importing /var/www/html/clustercontrol/sql/dc-schema.sql
Enter password:
=> Set a password for ClusterControl's MySQL user (cmon) [cmon]
=> Supported special characters: ~!@#$%^&*()_+{}<>?
=> Enter a CMON user password:
=> Enter the CMON user password again: => Creating the MySQL cmon user ...
Enter password:
Enter password:
```

Sau khi cài đặt xong hiển thị thông tin clustercontrol vừa cài đặt

```
#
# Configuration file for the Cmon Controller.
#

#
# The name or IP address of the Cmon Controller.
#
hostname=clusterctl

#
# Cmon Database credentials. The controller will use
# this database to store its own data structures.
#
mysql_hostname=127.0.0.1
mysql_port=3306
mysql_password='vvg2020'
cmon_user=cmon
cmon_db=cmon
controller_id=clustercontrol
rpc_key=9f1f34bbf5f7d25f2d77ba7bfb1a513795bedcee
```

Port và log cmon

```
Going to daemonize.. - cmon will write a log in /var/log/cmon.log from now
Starting cmon  --rpc-port=9500 --events-client='http://127.0.0.1:9510' --cloud-service='http://127.0.0.1:9518' : ok
```

Kiểm tra port đang hoạt động

```
netstat -ltpn
```

Mở port firewall


```
firewall-cmd --permanent --zone=public --add-port=443/tcp
firewall-cmd --permanent --zone=public --add-port=80/tcp
```


Reload lại firewall

```
sudo firewall-cmd --reload

```


Hoặc đơn giản là tắt tường lửa và SE policy và reboot máy. init 6: reboot


```
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
systemctl stop firewalld
systemctl disable firewalld
init 6

```

Truy cập website từ: https://10.20.20.170/clustercontrol/

Điền thông tin email password

-> Dasboard -> Import Exist Server/database -> Mysql Galera -> điền thông tin clustername: portal_cluster -> tich chon Mariadb version 10.4 -> Bo chon Automatic Node Discovery -> điền đầy đủ thông tin password -> Node maradb1, mariadb2

Sau khi kết nối thành công thực hiện trên giao diện website


Dashboard -> Cluster postal_cluster ->  Manage -> Schema and Users -> Create new user:

Username: backupuser
Password: vvg2020
Hostname: 10.20.20.%

```
Grant:  SELECT, RELOAD, LOCK TABLES, REPLICATION CLIENT, SHOW VIEW, EVENT, TRIGGER 
ON: *.*

```
-> Create

Sau khi tạo xong user, sửa file cấu hình trên giao diện như sau:

Dashboard -> Cluster postal_cluster -> Manage -> Configuration -> Change/Set Paramater -> Database: mariadb1 mariadb2 -> Group: galera -> Parameter: wsrep_sst_method -> New Value: mariabackup -> Proceed

Dashboard -> Cluster postal_cluster -> Manage -> Configuration -> Change/Set Paramater -> Database: mariadb1 mariadb2 -> Group: galera -> Parameter: wsrep_sst_auth -> New Value: backupuser:vvg2020 -> Proceed




## 8. Cài đặt GARBD + HAPROXY cho database cluster

Tạo thêm 1 máy CentOS cài garbd + haproxy


Đặt hostname

```
hostnamectl set-hostname grbd-ha
```

Sửa file host 4 máy ảo
```
vi /etc/hosts
```

```
10.20.20.168 mariadb1
10.20.20.167 mariadb2
10.20.20.170 clusterctl
10.20.20.174 grbd-ha
```


** Trên máy clusterctl **

Thực hiện copy ssh key cho phép cài đặt

```
ssh-copy-id grbd-ha

```

SSH sang máy grbd-ha

```
ssh grbd-ha
exit
```
** Trên máy grbd-ha**


Tắt firewalld và SE policy

```
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
systemctl stop firewalld
systemctl disable firewalld
init 6
```

** Trên giao diện cluster controll **

Deploy Haproxy như sau:

```
Search -> Add Load Balancer -> HA proxy -> Server Address: Nhập địa chỉ grbd-ha (10.20.20.174) -> Policy: leastconn -> Listen port: 3306 -> Include: mariadb1 Active -> Include: mariadb2 Backup -> Deploy

```
Deploy Garbd như sau:

```
Search -> Add Load Balancer -> Garbd -> Server Address: Nhập địa chỉ grbd-ha (10.20.20.174) -> Deploy
```



