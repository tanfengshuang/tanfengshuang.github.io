---
layout: post
title:  "(436-10)"
categories: Linux
tags: RHCA 436
---

### LVM Review

LVM uses the kernel Device Mapper subsystem to create its devices. Logical volumes are created in /dev/ as dm-* device nodes, but with symlinks in both /dev/ mapper and /dev/<vgname>/ using more administrator-friendly (and persistent) names.

###### Configuration files

The behavior of LVM can be configured in /etc/lvm/lvm.conf

*    dir: Which directory to scan for physical volume device nodes.
*    obtain_device_list_from_udev: If udev should be used to obtain a list of block devices eligible for scanning.
*    preferred_names: A list of regular expressions detailing which device names should have preference when displaying devices. If multiple device names (nodes) are found for the same PV, the LVM tools will give preference to those that appear earlier in this list.
*    filter: A list of regular expressions prefixed with a for Add or r for Remove. This list determines which device nodes will be scanned for the presence of a PV signature. The default is [ "a/.*/" ], which adds every single device. This option can be used to remove devices that should never be scanned.
*    backup: This setting determines if a text-based backup of volume group metadata should be stored after every change. This backup can be used to restore a volume group if the on-disk metadata gets corrupted.
*    backup_dir: This setting specifies where the backup of volume group metadate should be stored.
*    archive: This setting determines if old volume group configurations/layouts should be archived so they can later be used to revert changes.
*    archive_dir: This setting specifies where archives of old configurations should be stored.

### LVM Snapshots

*    LVM can be configured to keep archived copies of volume group metadata. If the archive option is set to 1 in /etc/lvm/lvm.conf, the LVM tools will create an archived copy of the current volume group metadata before making any changes on disk. 
*    In a default configuration, these archives can be found in /etc/lvm/archive. 
*    To view a list of all archived metadata for a volume group, examine the files in /etc/lvm/ archive, paying special attention to the description lines. Alternatively, the vgcfgrestore --list <vgname> command can be used.


```
# vgs
  VG                     #PV #LV #SN Attr   VSize  VFree 
  rhel_cloud-qe-16-vm-01   1   3   0 wz--n- 79.51g 64.00m

# vgcfgrestore --list rhel_cloud-qe-16-vm-01
  File:		/etc/lvm/archive/rhel_cloud-qe-16-vm-01_00000-974767884.vg
  Couldn't find device with uuid R6cSxM-lP59-HBoE-wBwX-120N-7fMi-VU2L6Z.
  VG name:    	rhel_cloud-qe-16-vm-01
  Description:	Created *before* executing 'pvscan --cache --activate ay 252:2'
  Backup Time:	Tue Feb 14 06:27:16 2017

   
  File:		/etc/lvm/backup/rhel_cloud-qe-16-vm-01
  VG name:    	rhel_cloud-qe-16-vm-01
  Description:	Created *after* executing 'pvscan --cache --activate ay 252:2'
  Backup Time:	Tue Feb 14 06:27:16 2017

# ls /etc/lvm/archive/
rhel_cloud-qe-16-vm-01_00000-974767884.vg

# ls /etc/lvm/backup/
rhel_cloud-qe-16-vm-01
```


```
# pvs
  PV         VG                     Fmt  Attr PSize  PFree 
  /dev/vda2  rhel_cloud-qe-16-vm-01 lvm2 a--  79.51g 64.00m
# vgs
  VG                     #PV #LV #SN Attr   VSize  VFree 
  rhel_cloud-qe-16-vm-01   1   3   0 wz--n- 79.51g 64.00m
# lvs
  LV   VG                     Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  home rhel_cloud-qe-16-vm-01 -wi-ao---- 27.45g                                                    
  root rhel_cloud-qe-16-vm-01 -wi-ao---- 50.00g                                                    
  swap rhel_cloud-qe-16-vm-01 -wi-ao----  2.00g            
# mount | grep home
/dev/mapper/rhel_cloud--qe--16--vm--01-home on /home type xfs (rw,relatime,seclabel,attr2,inode64,noquota)
# umount /home

# lvremove /dev/rhel_cloud-qe-16-vm-01/home 
Do you really want to remove active logical volume home? [y/n]: y
  Logical volume "home" successfully removed
# ls /etc/lvm/archive/
rhel_cloud-qe-16-vm-01_00000-974767884.vg  rhel_cloud-qe-16-vm-01_00001-1585223078.vg
# vgcfgrestore --list rhel_cloud-qe-16-vm-01
  File:		/etc/lvm/archive/rhel_cloud-qe-16-vm-01_00000-974767884.vg
  Couldn't find device with uuid R6cSxM-lP59-HBoE-wBwX-120N-7fMi-VU2L6Z.
  VG name:    	rhel_cloud-qe-16-vm-01
  Description:	Created *before* executing 'pvscan --cache --activate ay 252:2'
  Backup Time:	Tue Feb 14 06:27:16 2017

   
  File:		/etc/lvm/archive/rhel_cloud-qe-16-vm-01_00001-1585223078.vg
  VG name:    	rhel_cloud-qe-16-vm-01
  Description:	Created *before* executing 'lvremove /dev/rhel_cloud-qe-16-vm-01/home'
  Backup Time:	Thu Feb 16 20:05:55 2017

   
  File:		/etc/lvm/backup/rhel_cloud-qe-16-vm-01
  VG name:    	rhel_cloud-qe-16-vm-01
  Description:	Created *after* executing 'lvremove /dev/rhel_cloud-qe-16-vm-01/home'
  Backup Time:	Thu Feb 16 20:05:55 2017

# lvextend -L+10G /dev/rhel_cloud-qe-16-vm-01/root 
  Size of logical volume rhel_cloud-qe-16-vm-01/root changed from 50.00 GiB (12800 extents) to 60.00 GiB (15360 extents).
  Logical volume root successfully resized.
# vgcfgrestore --list rhel_cloud-qe-16-vm-01  
  File:		/etc/lvm/archive/rhel_cloud-qe-16-vm-01_00000-974767884.vg
  Couldn't find device with uuid R6cSxM-lP59-HBoE-wBwX-120N-7fMi-VU2L6Z.
  VG name:    	rhel_cloud-qe-16-vm-01
  Description:	Created *before* executing 'pvscan --cache --activate ay 252:2'
  Backup Time:	Tue Feb 14 06:27:16 2017

   
  File:		/etc/lvm/archive/rhel_cloud-qe-16-vm-01_00001-1585223078.vg
  VG name:    	rhel_cloud-qe-16-vm-01
  Description:	Created *before* executing 'lvremove /dev/rhel_cloud-qe-16-vm-01/home'
  Backup Time:	Thu Feb 16 20:05:55 2017

   
  File:		/etc/lvm/archive/rhel_cloud-qe-16-vm-01_00002-761482099.vg
  VG name:    	rhel_cloud-qe-16-vm-01
  Description:	Created *before* executing 'lvextend -L+10G /dev/rhel_cloud-qe-16-vm-01/root'
  Backup Time:	Thu Feb 16 20:19:20 2017

   
  File:		/etc/lvm/backup/rhel_cloud-qe-16-vm-01
  VG name:    	rhel_cloud-qe-16-vm-01
  Description:	Created *after* executing 'lvextend -L+10G /dev/rhel_cloud-qe-16-vm-01/root'
  Backup Time:	Thu Feb 16 20:19:20 2017
# ls /etc/lvm/backup/
rhel_cloud-qe-16-vm-01
# ls /etc/lvm/archive/
rhel_cloud-qe-16-vm-01_00000-974767884.vg  rhel_cloud-qe-16-vm-01_00001-1585223078.vg  rhel_cloud-qe-16-vm-01_00002-761482099.vg

# xfs_growfs /dev/rhel_cloud-qe-16-vm-01/root
meta-data=/dev/mapper/rhel_cloud--qe--16--vm--01-root isize=256    agcount=4, agsize=3276800 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=0        finobt=0
data     =                       bsize=4096   blocks=13107200, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=0
log      =internal               bsize=4096   blocks=6400, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 13107200 to 15728640

# vgcfgrestore --list rhel_cloud-qe-16-vm-01
   
  File:		/etc/lvm/archive/rhel_cloud-qe-16-vm-01_00000-974767884.vg
  Couldn't find device with uuid R6cSxM-lP59-HBoE-wBwX-120N-7fMi-VU2L6Z.
  VG name:    	rhel_cloud-qe-16-vm-01
  Description:	Created *before* executing 'pvscan --cache --activate ay 252:2'
  Backup Time:	Tue Feb 14 06:27:16 2017

   
  File:		/etc/lvm/archive/rhel_cloud-qe-16-vm-01_00001-1585223078.vg
  VG name:    	rhel_cloud-qe-16-vm-01
  Description:	Created *before* executing 'lvremove /dev/rhel_cloud-qe-16-vm-01/home'
  Backup Time:	Thu Feb 16 20:05:55 2017

   
  File:		/etc/lvm/archive/rhel_cloud-qe-16-vm-01_00002-761482099.vg
  VG name:    	rhel_cloud-qe-16-vm-01
  Description:	Created *before* executing 'lvextend -L+10G /dev/rhel_cloud-qe-16-vm-01/root'
  Backup Time:	Thu Feb 16 20:19:20 2017

   
  File:		/etc/lvm/backup/rhel_cloud-qe-16-vm-01
  VG name:    	rhel_cloud-qe-16-vm-01
  Description:	Created *after* executing 'lvextend -L+10G /dev/rhel_cloud-qe-16-vm-01/root'
  Backup Time:	Thu Feb 16 20:19:20 2017
```

###### Reverting LVM changes

To undo a change, umount first, then use vgcfgrestore -f

```
# pvs
  PV         VG                     Fmt  Attr PSize  PFree 
  /dev/vda2  rhel_cloud-qe-16-vm-01 lvm2 a--  79.51g 17.51g
# vgs
  VG                     #PV #LV #SN Attr   VSize  VFree 
  rhel_cloud-qe-16-vm-01   1   2   0 wz--n- 79.51g 17.51g
# lvs
  LV   VG                     Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root rhel_cloud-qe-16-vm-01 -wi-ao---- 60.00g                                                    
  swap rhel_cloud-qe-16-vm-01 -wi-ao----  2.00g                                                    

# vgcfgrestore -f /etc/lvm/archive/rhel_cloud-qe-16-vm-01_00002-761482099.vg rhel_cloud-qe-16-vm-01
  Restored volume group rhel_cloud-qe-16-vm-01

# pvs
  PV         VG                     Fmt  Attr PSize  PFree 
  /dev/vda2  rhel_cloud-qe-16-vm-01 lvm2 a--  79.51g 27.51g
# vgs
  VG                     #PV #LV #SN Attr   VSize  VFree 
  rhel_cloud-qe-16-vm-01   1   2   0 wz--n- 79.51g 27.51g
# lvs
  LV   VG                     Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root rhel_cloud-qe-16-vm-01 -wi-ao---- 50.00g                                                    
  swap rhel_cloud-qe-16-vm-01 -wi-ao----  2.00g                                                    
# ls /etc/lvm/archive/
rhel_cloud-qe-16-vm-01_00000-974767884.vg  rhel_cloud-qe-16-vm-01_00001-1585223078.vg  rhel_cloud-qe-16-vm-01_00002-761482099.vg

# vgcfgrestore -f /etc/lvm/archive/rhel_cloud-qe-16-vm-01_00001-1585223078.vg rhel_cloud-qe-16-vm-01
  Restored volume group rhel_cloud-qe-16-vm-01
# pvs
  PV         VG                     Fmt  Attr PSize  PFree 
  /dev/vda2  rhel_cloud-qe-16-vm-01 lvm2 a--  79.51g 64.00m
# vgs
  VG                     #PV #LV #SN Attr   VSize  VFree 
  rhel_cloud-qe-16-vm-01   1   3   0 wz--n- 79.51g 64.00m
# lvs
  LV   VG                     Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  home rhel_cloud-qe-16-vm-01 -wi------- 27.45g                                                    
  root rhel_cloud-qe-16-vm-01 -wi-ao---- 50.00g                                                    
  swap rhel_cloud-qe-16-vm-01 -wi-ao----  2.00g        
```

In some cases, it might be necessary to deactivate, then reactivate the logical volume to make sure that all changes are committed in momery.

```
# lvchange -an /dev/vg_example/lv_example
# lvchange -ay /dev/vg_example/lv_example
```


### HA-LVM

There are two ways to use LVM on shared storage in a cluster

*    With Clustered LVM, all volume groups and logical volumes on shared storage are available to all cluster nodes all of the time. Clustered LVM is a good choice when working with a shared file system, like GFS2
*    With HA-LVM, a volume group and its logical volumes can only be accessed by one node at a time. HA-LVM is a good choice when working with a more traditional file system like ext4 or XFS, and restricted access to just one node at a time is desired to prevent file system and/or data corruption.

###### Creating an HA-LVM volume

```
# pvcreate /dev/sdd1
# vgcreate shared_vg /dev/sdd1
# lvcreate -L 1G -n ha_lv shared_vg
# mkfs -t xfs /dev/shared_vg/ha_lv
```

###### Configuring HA-LVM

There are two ways to configure HA-LVM

*    use LVM tages
*    use the Clustered LVM deamon - clvmd

The use of clvmd in conjuction with the LVM resource agent used by Pacemaker is not supported for active/passive LVM configurations. Therefore, in a cluster managed with Pacemaker, HA-LVM must use the tagging method. Since this course uses Pacemaker clusters, it will use the tagging method in its examples and exercises of HA-LVM configuration.

LVM performs file locking to prevent conflicting LVM commands from running concurrently, either on a single machine or across a cluster. To implement HA-LVM, it is necessary to ensure that LVM is configured to use local file-based locking.


### Clustered LVM


Clustered LVM allows the use of regular LVM volume groups and logical volumes on shared storage. In a cluster configured with clustered LVM, a volume group and its logical volumes are accessible to all cluster nodes at the same time. With clustered LVM, administrators can use the management benefits of LVM in conjunction with a shared file system like GFS2, for scenarios such as making virtual machine images inside logical volumes available to all cluster nodes.

The active/active configuration of logical volumes in a cluster using clustered LVM is accomplished by using a daemon called clvmd to propagate metadata changes to all cluster nodes. The clvmd daemon manages clustered volume groups and communicates their metadata changes made on one cluster node to all the remaining nodes in the cluster.

Without the clvmd daemon, LVM metadata changes made on one cluster node would be unknown to other cluster nodes. Since these metadata define which storage addresses are available for data and file system information, metadata changes not propagated to all cluster nodes can lead to corruption of LVM metadata as well as data residing on LVM physical volumes.

In order to prevent multiple nodes from changing LVM metadata simultaneously, clustered LVM uses Distributed Lock Manager (DLM) for lock management. The clvmd daemon and the DLM lock manager must be installed prior to configuring clustered LVM. They can be obtained by installing the lvm2-cluster and dlm packages, respectively. Both packages are available from the Resilient Storage repository.


###### Configuring clustered LVM

