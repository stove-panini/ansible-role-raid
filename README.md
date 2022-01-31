ansible-raid
============
Creates a RAID (using mdraid or lvmraid) and optional VDO layer.

VDO
---
Optionally adds a VDO layer, tuned according to the resulting RAID device. In the case of an lvmraid, [lvmvdo](https://man7.org/linux/man-pages/man7/lvmvdo.7.html) is used.

Please note that lvmvdo is still an experimental feature of LVM!

Tuning values are based on those recommended by [the RHEL 8 VDO documentation](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/deduplicating_and_compressing_storage/). The values for the "raid\_vdo\_purpose" configuration variable come from Section 1.8 of the guide:

>Replace logical-size with the amount of logical storage that the VDO volume should present:
>- For active VMs or container storage, use logical size that is ten times the physical size of your block device. For example, if your block device is 1TB in size, use 10T here.
>- For object storage, use logical size that is three times the physical size of your block device. For example, if your block device is 1TB in size, use 3T here.

Options
-------
Set the following as vars for your role. Options in bold are required. Choices in bold indicate the default.

| Option | Choices | Comments |
| ------ | ------- | -------- |
| **raid_level** | raid0, raid1, raid5, raid6, raid10 | |
| **raid_block_devs** | *No default* | A list of block devices (e.g. `[ sdb, sdc, sdd, sde ]` ) |
| **raid_name** | *No default* | For mdraid, this will become `/dev/<raid_name>` while lvmraid will use `<raid_name>_lv`, `<raid_name>_vg`, etc.
| raid\_backend | **mdraid**, lvmraid | |
| raid\_create\_vdo | yes, **no** | If yes, this creates a VDO layer on your RAID. |
| raid\_vdo\_purpose | **virtualization**, object\_storage | Choosing "virtualization" creates a logical layer 10x the size of the backing device, while "object\_storage" creates a logical layer 3x the size of the backing device. |
| raid\_vdo\_custom\_size | *No default* | Instead of using the multipliers predefined in vdo\_purpose, you may specify a custom size. (e.g. `6 TB`, `40 GB`, etc.) |
| raid\_create\_fs | **yes**, no | Formats the resulting device with the XFS file system. Chunk size and stripe width are automatically tuned to the backing storage and RAID-type you choose. However, if you use VDO, it will simply use the defaults. |
| raid\_add\_to\_fstab | **yes**, no | Mounts the resulting device and adds it to your `/etc/fstab` |
| raid\_mountpoint | **`/mnt/<raid_name>`** | Where the resulting device will be mounted

Sample Playbook
---------------
```yaml
- hosts: raid-test.example.com
  roles:
    - role: raid
      vars:
        raid_level: raid10
        raid_name: my_cool_raid
        raid_block_devs: [ sdb, sdc, sdd, sde ]
        raid_backend: lvmraid
        raid_use_vdo: yes
        raid_add_to_fstab: no
```
