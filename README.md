ansible-raid
============
Creates a RAID (using mdraid or lvmraid) and optional VDO layer.

VDO
---
Optionally adds a VDO layer, tuned according to the resulting RAID device. In the case of an lvmraid, [lvmvdo](https://man7.org/linux/man-pages/man7/lvmvdo.7.html) is used. Tuning values are based on those recommended by [the RHEL 8 VDO documentation](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/deduplicating_and_compressing_storage/).

Please note that lvmvdo is still an experimental feature of LVM!

Options
-------
Set the following as vars for your role. Choices in bold indicate the default.

| Option | Choices | Comments |
| ------ | ------- | -------- |
| raid\_backend | **mdraid**, lvmraid | |
| raid\_level | raid0, raid1, raid5, raid6, **raid10** | |
| raid\_block\_devs | *No default* | A list of block devices (e.g. `[ sdb, sdc, sdd, sde ]` ) |
| raid\_name | *No default* | For mdraid, this will become `/dev/<raid_name>` while lvmraid will use `<raid_name>_lv`, `<raid_name>_vg`, etc.
| create\_vdo | yes, **no** | If yes, this creates a VDO layer on your RAID. |
| vdo\_purpose | **virtualization**, block\_storage | Choosing "virtualization" creates a logical layer 10x the size of the backing device, while "block\_storage" creates a logical layer 3x the size of the backing device. |
| create\_fs | **yes**, no | Formats the resulting device with the XFS file system. Chunk size and stripe width are automatically tuned to the backing storage and RAID-type you choose. However, if you use VDO, it will simply use the defaults. |
| add\_to\_fstab | **yes**, no | Mounts the resulting device and adds it to your `/etc/fstab` |
| mountpoint | `/mnt/<raid_name>` | Where the resulting device witll be mounted
 
Sample Playbook
---------------
```yaml
- hosts: raid-test.example.com
  roles:
    - role: raid
      vars:
        raid_backend: mdraid
        raid_level: raid10
        raid_name: my_cool_raid
        raid_block_devs: [ sdb, sdc, sdd, sde ]
        use_vdo: yes
        add_to_fstab: no
```
