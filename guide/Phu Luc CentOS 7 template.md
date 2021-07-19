cen77-base changelog

Thực hiện những việc sau tạo template
----------------------------
```
: v0.0.1
-* Update to latest
-* installed epel repo
-* sshd reconfig: disallow root login by password
-* disable firewall
-* disable ctrl-alt-del reboot (systemctl mask ctrl-alt-del.target)
-* install cloud-init
-* modify console size (ttyS0)
-* config zram swap
-* adjust vm.swapiness
-* installed some alias
-* installed basic monitor scripts: iftop iotop htop vim net-tools
-* adjust bootloader timeout
-* adjust default nx-user : groups trusted,wheel
-* pam require wheel group
-* installed lz4-1.7.5-3.el7
-* schedule TRIM weekly
-* schedule ntp update hourly
-* installed motd
* history clear
```
