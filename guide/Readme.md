# Training docs
[1. Hướng dẫn cài đặt os Proxmox](https://git.linex.vn/LiNEX/vvg-bot/src/branch/master/guide/1.Os-proxmox-install.md)

[2. Cấu hình bond network trên 3 nodes proxmox](https://git.linex.vn/LiNEX/vvg-bot/src/branch/master/guide/2.%08Network-bonding.md)

[3. Tạo phân vùng LVM cho ổ đĩa /dev/sdb](https://git.linex.vn/LiNEX/vvg-bot/src/branch/master/guide/3.LVM-for-SSD-disk.md)

[4. Tối ưu ZFS cho OS proxmox](https://git.linex.vn/LiNEX/vvg-bot/src/branch/master/guide/4.tuning-zfs.md)

[5. Cài đặt linstor Storage ](https://git.linex.vn/LiNEX/vvg-bot/src/branch/master/guide/5.Linstor%20Storage.md)

[6. Cài đặt máy ảo](https://git.linex.vn/LiNEX/vvg-bot/src/branch/master/guide/5.Create-Vms.md)

[7. Cài đặt Cloud-init trên máy ảo ubuntu và centos](https://git.linex.vn/LiNEX/vvg-bot/src/branch/master/guide/7.Cloud-init.md)

[8. Cài đặt 1 máy ảo cluster-control và 2 máy ảo mariadb](https://git.linex.vn/LiNEX/vvg-bot/src/branch/master/guide/8.clustercontrol-mariadb.md)

[9. Cài đặt GARBD + HAPROXY cho database cluster](https://git.linex.vn/LiNEX/vvg-bot/src/branch/master/guide/9.garbd-haproxy.md)


[10. Cài đặt HAPROXY (Loadbalancer)](https://git.linex.vn/LiNEX/vvg-bot/src/branch/master/guide/10.%20Haproxy_LB_notes.md)


[11. Backup và restore máy ảo.](https://git.linex.vn/LiNEX/vvg-bot/src/branch/master/guide/11.%20Backup%20v%C3%A0%20restore%20template.md)

[12. Kiểm tra dịch vụ.](https://git.linex.vn/LiNEX/vvg-bot/src/branch/master/guide/12.%20Ki%E1%BB%83m%20tra%20d%E1%BB%8Bch%20v%E1%BB%A5.md)

[13. Fortigate notes](https://git.linex.vn/LiNEX/vvg-bot/src/branch/master/guide/13.%20Fortigate%20Notes.md)

[14. Softdog](https://git.linex.vn/LiNEX/vvg-bot/src/branch/master/guide/14.%20Softdog.md)


[15. SMB notes](https://git.linex.vn/LiNEX/vvg-bot/src/branch/master/guide/15.%20Smb%20Notes.md)

## 1.Thực hiện deploy trên các trạm

Thực hiện cài đặt trên 4 máy pve lần lượt theo hướng dẫn từ 1 tới 4 sau đó thực hiện tạo ZFS trên các ổ cứng SATA (xem phụ lục [ZFS](https://git.linex.vn/LiNEX/vvg-bot/src/branch/master/guide/Ph%e1%bb%a5%20L%e1%bb%a5c%20ZFS%20Create%20pool.md) ) Sau đó tiếp tục thực hiện 11 và 12.

Đọc thêm 6 7 8 9 để hiểu thêm về dịch vụ.

Tiến hành upload lên bản backup có sẵn của các máy (db, clustercontrol, ha...) cần thiết cho hệ thống.

Restore lại các máy đã backup.

Truy cập vào các máy kiểm tra dịch vụ.

Một số lưu ý khi triển khai:

## 2.Fortigate

## 3. Kiểm tra kết nối mạng giữa các thiết bị

## 4. Mô hình kết nối app -> lb

Thay đổi so với dự kiến ban đầu, các app sẽ kết nối trực tiếp tới db và lb

app1 -> lb1 -> db1
app2 -> lb2 -> db2


## 5. Softdog trên pve
