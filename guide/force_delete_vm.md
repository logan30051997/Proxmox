Proxmox KVM virtual machine: Cannot delete due to missing storage
=================================================================

Today we encountered a situation where a Proxmox system’s KVM virtual machine refused to delete after the storage volume that it’s virtual HDD resided on was lost; trying to delete the KVM from the web GUI resulted in the following error:  

> TASK ERROR: storage ‘proxmoxHDD’ does not exists

  Attempting to delete it from the command line using:  

```shell
qm destroy [VM ID]
```
  …resulted in:  

> storage ‘proxmoxHDD’ does not exists

  Fortunately, there’s a way around this. The KVM config files live in `/etc/pve/qemu-server`:

  Move or erase the `VM ID.conf` file and when you refresh your web GUI the VM should be gone.
```shell
cd /etc/pve/qemu-server
rm 101.conf
```