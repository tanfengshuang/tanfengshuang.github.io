layout: post
title:  "How to extend root partition size"
categories: Linux
tags: LVM
---

### RHEL5 and RHEL6

For RHEL 6 and RHEL 5 this action is super easy because of EXT filesystem. It can be done by shrinking the /home lvm partition. Please follow these steps:

```
1. Check the free space and list the lvm volumes
# df -h
# lvs

2. Unmount the home partition and activate LVM
# umount /home
# lvm vgchange -a y

3. Shrink the home partition (for example to 10GB)
# resize2fs -f /dev/mapper/LVM_HOME_GOES_HERE 10G
# lvreduce -L10G /dev/mapper/LVM_HOME_COMES_HERE

4. Extend the root partition (for example with 80GB)
# lvextend -L+80G  /dev/mapper/LVM_ROOT_GOES_HERE
# resize2fs /dev/mapper/LVM_ROOT_GOES_HERE

5. Remount the home partition
# mount /home
```

### RHEL7

 On RHEL 7 is the situation far more complicated. By default RHEL 7 uses XFS filesystem. This filesystem does not support shrinking - that means the only way how to extend root is deleting the /home partition.

```
1. Check your filesystem (look for xfs), free space and list the lvm volumes
# cat /etc/fstab
# df -h
# lvs

2. Unmount the home partition and activate LVM
# umount /home
# lvm vgchange -a y

3. Delete home partition
# lvremove /dev/mapper/LVM_HOME_GOES_HERE

4. Extend root partition
# lvextend -L+80G /dev/mapper/LVM_ROOT_GOES_HERE
# xfs_growfs /
```
