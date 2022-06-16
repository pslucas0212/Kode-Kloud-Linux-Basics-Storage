# KodeKloud Linux Basics: Storage in Linux  

[KodeKloud Linux Basics Course Notes Table of Contents](https://github.com/pslucas0212/LinuxBasics)

## Storage in Linux

## Storage Basics

### Block Devices
Block device is a type of file that can be found under /dev directory representing a piece of hardware like a spinning disk or an SSD drive.  It is a called block storage cause data is written and read in blocks or "chunks" of data.

To see list of block devices in your system run:
To see block storage run:
```
$ lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
mmcblk0     179:0    0 29.8G  0 disk 
├─mmcblk0p1 179:1    0  2.2G  0 part 
├─mmcblk0p2 179:2    0    1K  0 part 
├─mmcblk0p5 179:5    0   32M  0 part 
├─mmcblk0p6 179:6    0  256M  0 part /boot
└─mmcblk0p7 179:7    0 27.3G  0 part /
```
Or run
```
$ ls -l /dev/ | grep "^b"
brw-rw----  1 root disk      7,   0 Jun  7 16:17 loop0
brw-rw----  1 root disk      7,   1 Jun  7 16:17 loop1
brw-rw----  1 root disk      7,   2 Jun  7 16:17 loop2
brw-rw----  1 root disk      7,   3 Jun  7 16:17 loop3
brw-rw----  1 root disk      7,   4 Jun  7 16:17 loop4
brw-rw----  1 root disk      7,   5 Jun  7 16:17 loop5
brw-rw----  1 root disk      7,   6 Jun  7 16:17 loop6
brw-rw----  1 root disk      7,   7 Jun  7 16:17 loop7
brw-rw----  1 root disk    179,   0 Jun  7 16:17 mmcblk0
brw-rw----  1 root disk    179,   1 Jun  7 16:17 mmcblk0p1
brw-rw----  1 root disk    179,   2 Jun  7 16:17 mmcblk0p2
brw-rw----  1 root disk    179,   5 Jun  7 16:17 mmcblk0p5
brw-rw----  1 root disk    179,   6 Jun  7 16:17 mmcblk0p6
brw-rw----  1 root disk    179,   7 Jun  7 16:17 mmcblk0p7
...
```
Note in this example the mcblk0 represents the entire disk as denoted by the disk in the type field, and the part in the type field rerpresents partitions.

Each block device has a major and minor number.  The first number reprsent block device type and the second number identifies the whole disk and partitions created  
  
Major Number | Device Type
------------ | -----------
1 | RAM
3 | Hard Disk or CD ROM
6 | Parallel Printers
8 | SCSI DISK - fixed naming starting with sdxxx

From a RHEL 8 VM when running lsblk
```
$ lsblk
NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda             8:0    0  100G  0 disk 
├─sda1          8:1    0  600M  0 part /boot/efi
├─sda2          8:2    0    1G  0 part /boot
└─sda3          8:3    0 98.4G  0 part 
  ├─rhel-root 253:0    0 63.5G  0 lvm  /
  ├─rhel-swap 253:1    0    4G  0 lvm  [SWAP]
  └─rhel-home 253:2    0   31G  0 lvm  /home
sr0            11:0    1 10.2G  0 rom  
```


The disk can be broken down into smaller parts of the disk called partions.  Partition segments space for use of a particular purpose.  You don't have to partition a disk.  You can use it as is.  But partitioning makes the disk more usable and more flexibiligy.  

Partition information is stored in a partition table.
  
Partition information can be found with the fdisk command and can be used to create and delete partitions
```
sudo fdisk -l /dev/sda
[sudo] password for pslucas: 
Disk /dev/sda: 100 GiB, 107374182400 bytes, 209715200 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: DA76B744-AE84-452C-917F-8A525C722EEF

Device       Start       End   Sectors  Size Type
/dev/sda1     2048   1230847   1228800  600M EFI System
/dev/sda2  1230848   3327999   2097152    1G Linux filesystem
/dev/sda3  3328000 209713151 206385152 98.4G Linux LVM
```  
There are three types of disk partitions
- Primary partition used to boot the system.  There is a limit 4 primary partition. 
- Extend partitions cannot be used on their own but can host logical partitions - 1 or more.  Extended partition is like a disk drive with its own partitions.  You can create and host logical partitions. It has a partiton table that points to one or more partions
- Logical partition created with an extended partition


Partition table or partitioning scheme defines how a disk is partioned.  We see the traditional MBR partition scheme. Master Boot Record which has been around for over 30 years.  Only 4 primary partitions in MBR and maximum size 2 TB.  If we want more than 4 partitions for a disk we would use a extended partition and carve out logical partions.   
  
GPT is another partition scheme and stands for GUID Partition Table.  A more recent partitioning scheme to address MBR limitations.  GPT allows unlimted number of partitions per disk and no max sizer per partiion.    Only limited by the OS.  RHEL allows 129 partitions and no limite on size a partition.

Creating partitions
```
$ lsblk
```
If you see a non-partitioned disk run gdisk to partition.  gdisk has a menu driven interface.  Use $ gdisk ? to see options.  gdisk is an improved version of fdisk that works with gpt partition table
  
Example gdisk
```
$ gdisk /dev/sdb
```
Once you are in the gdisk menu, use ? to see all available options.   
Next type the n command to. create a new partition.  Pick partition number and size of partition.  It will ask for a hex code for the partition type.  Stick Linux filed default 8300.  Type L key to see all available types.  Now type w command to write the disk.

  
Once your disk is partioned  check status of the partition run lsblk or fdisk:

Example of creating a partion with gdkisk
Before running gdkisk
```
$ lsblk
NAME                   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
vda                    252:0    0   10G  0 disk 
└─vda1                 252:1    0   10G  0 part 
  ├─vagrant--vg-root   253:0    0    9G  0 lvm  /
  └─vagrant--vg-swap_1 253:1    0  980M  0 lvm  [SWAP]
vdb                    252:16   0    1G  0 disk 
vdc                    252:32   0    1G  0 disk
```
Running gdisk with these options:
In the interactive prompt, enter n
Select parition number = 1 (for vdd1)
Select default first sector = 2048
Select +500M when asked for last sector
Use default hex code = 8300
Finally type w to write to the partition table

```
$ sudo gdisk /dev/vdb
GPT fdisk (gdisk) version 1.0.3

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: not present

Creating new GPT entries.

Command (? for help): GPT
b       back up GPT data to a file
c       change a partition's name
d       delete a partition
i       show detailed information on a partition
l       list known partition types
n       add a new partition
o       create a new empty GUID partition table (GPT)
p       print the partition table
q       quit without saving changes
r       recovery and transformation options (experts only)
s       sort partitions
t       change a partition's type code
v       verify disk
w       write table to disk and exit
x       extra functionality (experts only)
?       print this menu

Command (? for help): n
Partition number (1-128, default 1): 1
First sector (34-2097118, default = 2048) or {+-}size{KMGTP}: 
Last sector (2048-2097118, default = 2097118) or {+-}size{KMGTP}: +500M
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300): 8300
Changed type of partition to 'Linux filesystem'

Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): Y
OK; writing new GUID partition table (GPT) to /dev/vdb.
The operation has completed successfully.
```
After running gdisk:
```
$ lsblk
NAME                   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
vda                    252:0    0   10G  0 disk 
└─vda1                 252:1    0   10G  0 part 
  ├─vagrant--vg-root   253:0    0    9G  0 lvm  /
  └─vagrant--vg-swap_1 253:1    0  980M  0 lvm  [SWAP]
vdb                    252:16   0    1G  0 disk 
└─vdb1                 252:17   0  500M  0 part 
vdc                    252:32   0    1G  0 disk 
```
## File Systems in Linux  
Partitioning alone do not make the disks usable.  The disk and partitions are seen by the kernal as a raw disk.  To write to a disk or partion we must first create a file system which defines how data is written to the disk.  After we create a file system we mount the file system to a directoy and then we can read and write data to the disk.

Linux File Systems

EXT2 and EXT3 do a great job of reliably storing data.  In case of an unclean shutdown EXT2 since it doesn't use a journal, can take a long to reboot.  EXT3 implemented additional features to boot more quickly.  EXT 4 included additional features
  
Capabilities | EXT2 | EXT3 | EXT4
-------------|------|------|-----
Max file size | 2 TB File Size | 2 TB File Size | 16 TB File Size
Max volume size | 4 TB Volume size | 4 TB Volume size | 1 Exabyte
Other capabilites  | supports compression | Uses Journal | Uses Journal
Compatiblilty with EXT3/EXT2| Support Linux permissions | Backwards Compatible | Backwards Compatible
Crash recovery | Long crash Recovery | Faster Crash Recovery | Uses chsksum for Journal
  
Creating file systems.  In this example we will add a file system the /dev/dsb1 partition::
```
$ mkfs.ext4 /dev/sdb1
$ mkdir /mnt/ext4;
$ mount /dev/sdb1 /mnt/ext4
$ mount | grep /dev/sdb1
$ df -hP | grep /dev/sdb1
```
To make mount permanent after reboot add an entry to /etc/fstab:  

Add to file:
```
#<file system> <mount point>    <type>           <options>                              <dump>           <pass>
/dev/sda1      /                ext4             defaults,relatime,errors=panic            0               1 ~
```
  
Field | Purpose
----- | -------
Filesystem | Such as /dev/vdb1 to be mounted
Mountpoint | Directory to be mounted on
Type | Example ext2, ext3, ext4
Options | Such as RW or RO - mounts file to read-wrote or read-only
Dump | 0=ignore, 1=backup - 0 don't make a backup
Pass | 0=ignore, 1 or 2 = FSCK filesystem check enforced - order of checking filesystem after crash. 1 is the maximum is usually set for the root file system
  
or
```
echo "/dev/sdb1  /mnt/ext4   ext4 rw 0 0" >> /etc/fstab
```
In the follwoing lines we want to create and mount and ext4 filesystem on an available drive (or partition).  We will have to first determine the disks/partions available.    
See disks and partitons example:
```
$ lsblk
NAME                   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
vda                    252:0    0   10G  0 disk 
└─vda1                 252:1    0   10G  0 part 
  ├─vagrant--vg-root   253:0    0    9G  0 lvm  /
  └─vagrant--vg-swap_1 253:1    0  980M  0 lvm  [SWAP]
vdb                    252:16   0    1G  0 disk 
vdc                    252:32   0    1G  0 disk /mnt/backups
```
Check for present filesystem (mounted file systems).  Note only disks/partions that are mounted have fileystem:
```
$ df -h
Filesystem                    Size  Used Avail Use% Mounted on
udev                          461M     0  461M   0% /dev
tmpfs                          99M  5.4M   94M   6% /run
/dev/mapper/vagrant--vg-root  8.9G  1.6G  6.9G  19% /
tmpfs                         493M     0  493M   0% /dev/shm
tmpfs                         5.0M     0  5.0M   0% /run/lock
tmpfs                         493M     0  493M   0% /sys/fs/cgroup
tmpfs                          99M     0   99M   0% /run/user/1002
/dev/vdc                     1008M  1.3M  956M   1% /mnt/backups
```
Check for fileystem in use on a mounted filesystem:
```
$ sudo blkid /dev/vdc
/dev/vdc: UUID="6cc962a4-a0a7-4d8a-a124-339cc63ff740" TYPE="ext2"
```
Make an ext4 file system on /dev/vdb and mount it to /mnt/data
```
$ sudo mkfs.ext4 /dev/vdb
mke2fs 1.44.1 (24-Mar-2018)
Creating filesystem with 262144 4k blocks and 65536 inodes
Filesystem UUID: d62fe945-e483-4b6c-915e-bca8fadb4cdd
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done
```
```
$ sudo mkdir /mnt/data
```
```
$ sudo mount /dev/vdb /mnt/data
```
```
$ mount | grep /dev/vdb
/dev/vdb on /mnt/data type ext4 (rw,relatime,data=ordered)
```
```
$ df -hP | grep /dev/vdb
/dev/vdb                      976M  2.6M  907M   1% /mnt/data
```
Make the mount permanent by adding this line to /etc/fstab:
```
/dev/vdb        /mnt/data       ext4    rw                0        0
```
Updated /etc/fstab:
```
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
/dev/mapper/vagrant--vg-root /               ext4    errors=remount-ro 0       1
/dev/mapper/vagrant--vg-swap_1 none            swap    sw              0       0
/dev/fd0        /media/floppy0  auto    rw,user,noauto,exec,utf8 0       0
/dev/vdb        /mnt/data       ext4    rw                0        0
``` 

## External Storage DAS, NAS and SAN
Three types of external storage with High Aavilability - DAS -> Direct Attached Storage, NAS -> Network Attached Storage, SAN -> Storage Area Network (uses a fiber channel)  
DAS - connects directly to the host system. THe host system seed a directly connect block device.  No firewall and very fast.  Since DAS is directly attached it can't be shared. 
NAS - Located separately from the hosts that will use it.  The data traverses through the network.  NAS is similar to NFS.  Storage is provide to the hosts via a share and exported via NFS to the host.  Good for sharing storage among many different devices.  Not recommended for production databases or OS installations
SAN - Provides block storage for business critical applications with high performance/throughput and low latency.  Storage is allocated to hosts in the form of a LUN (Logical Unit Number).  A LUN is a range of blocks provisioned from a pool of shared storage and presented to the server as a logical disk.  The host systems sees the storage as a raw disk and create partions with file systems that are then mounted to the server.  FBCP - FIber channel protocoal use a fibre switch to a HBA (Host bust adapter) in a PCI slot.  Good for Databases, Hypervisors, etc.
  

DAS | NAS | SAN
--- | --- | ---
Directly Attached | Network attached NFS/CIFS | Storage Area Network FC (Fibre Channel) or iSCSI
Block Storage | File storage | Block Storage
Fast and Reliable | Reasonably Fast and Reliable | Fast, secure and reliable
Dedicated to single host | Shared Storage | Highly Available
Ideal for small Businesses | Mid/Large Business | Enterprise Storage
Suitable for OS | Not suitable for OS | Not suitable for OS
  
## NFS File System  
Unlock block devices, NFS saves data in the form of fiels and works on server client model.  As an exampoel the NFS Server has /software/repos is shared across the network via a mount to local host.    Directory share is called exporting.  
  
NFS server maintains exports file at /etc/exports that defines the clients that access the directories on the NFS server.  
/etc/exports has a list of cliesnts that can access a particular NFS folder.  You can use hostnames, ip addresses, ip range, or wilde card * for any server.  There maybe a firewall between the clients and NFS and NFS ports may need to be opened for the clients to access the NFS server.    
```
/software/repos hosta hostb hostc
```
Share the directory on the clients with exportfs command.  
Share all mounts in the /etc/exports run:
```
$ exportsfs -a
```
Allows manually to export a directly
```
$ exportsfs -o 10.16.13.201:/software/repos
```
Now you can mount directory to the client.
```
$ mount 10.16.13.201:/software/repos /mnt/software/repos
```
  
## Logical Volume Manager (LVM)

Logical Volume Manager allows grouping of multiple physcial drives (hard disks or partitions) into one volume group where you can then carve out logical volumes.   This allows logical volumes to be easily resized dynamically as long as there space in the volume group.  
   
 To use LVM, you first need to install LVM.
```
$ apt-get install lvm2
```
Next identify free disks or partitions and create physical volume objects (pv).  For example we have one free disk /dev/sdb1.  Creat pv:
```
$ pvcreate /dev/sdb
```
Now create a volume group or vg.  A volume group can have one or more physcial volumes
```
$ vgcreate caleston_vg /dev/sdb
$ pvdisplay
$ vgdisplay
```
Lets create some logical volumes with lvcreat command.  Create a linear volume of 1GB.  -L linear command allows you to use multiple physical volumes to make the logical volume
```
$ lvcreate -L 1G -n vol1 caleston_vg
$ lvdisplay
```
List volumes and create filesystem and mount
```
$ lvs
$ mkfs.ext4 /dev/caleston_vg/vol1
$ mount -t ext4 /dev/caleston_vg/vol1 /mnt/vol1
```
Resize volume on volume 1.  First use vgs command to see if there is enough room to increase the volume.  Then expand the voloume
```
$ vgs
$ lvresize -L +1G /dev/caleston_vg/vol1
$ dfh -hP /mnt/vol1
```
After resizing the volume, you will need to resize the filesytem to match the size of the volume
```
$ resize2fs /dev/caleston_vg/vol1
$ df -hP /mnt/vol1
```
Notice that we can do this without stopping the system or unmounting the drive

Logical Voluems are availabe at two places:
Logical Volume | Fileystem Path
---------------|---------------
vol1 | /dev/caleston_vg/vol1
vol1 | /dev/mapper/caleston_vg-vol1

Let's work through an example of seeting up a logical volueme.  First is LVM installed on the system.  Use the follwowing commands to check:
```
$ sudo pvdisplay
[sudo] password for bob: 
  --- Physical volume ---
  PV Name               /dev/vda1
  VG Name               vagrant-vg
  PV Size               <10.00 GiB / not usable 2.00 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              2559
  Free PE               0
  Allocated PE          2559
  PV UUID               ecXq0T-x7d2-Hma9-rS1w-Mi02-st3y-EqIoiW'''
```
or 
```
$ sudo lvs
  LV     VG         Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root   vagrant-vg -wi-ao----  <9.04g                                                    
  swap_1 vagrant-vg -wi-ao---- 980.00m  
```
See the name of the physical vloume that has been created with pvdisplay command:
```
$ sudo pvdisplay
  --- Physical volume ---
  PV Name               /dev/vda1
  VG Name               vagrant-vg
  PV Size               <10.00 GiB / not usable 2.00 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              2559
  Free PE               0
  Allocated PE          2559
  PV UUID               ecXq0T-x7d2-Hma9-rS1w-Mi02-st3y-EqIoiW
```
This size fo thsi pv 10GB.  The volume group name is vagrant-vg  

Let's take two disk /dev/vdb and /dev/vdc and create two pvs
```
$ sudo pvcreate /dev/vdb
  Physical volume "/dev/vdb" successfully created.
$ sudo pvcreate /dev/vdc
  Physical volume "/dev/vdc" successfully created.
```
Now create a volume group called caleston_vg from the newly create pvs.
```
$ sudo vgcreate caleston_vg /dev/vdb /dev/vdc
  Volume group "caleston_vg" successfully created
```
Create logical volume 1G in sized called data
```
$ sudo lvcreate -L 1G -n data caleston_vg
  Logical volume "data" created.
```
Now create an ext4 file system on the lv and mount it to /mnt/media
```
$ sudo mkfs.ext4 /dev/caleston_vg/data
mke2fs 1.44.1 (24-Mar-2018)
Creating filesystem with 262144 4k blocks and 65536 inodes
Filesystem UUID: 27594346-ed81-4102-81fc-2663782643b1
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done

$ sudo mkdir /mnt/media
$ sudo mount -t ext4 /dev/caleston_vg/data /mnt/media
```
Let's add 500m to the LV.  First check for available "disk" on the vg:
```
$ sudo vgs
  VG          #PV #LV #SN Attr   VSize   VFree   
  caleston_vg   2   1   0 wz--n-   1.99g 1016.00m
  vagrant-vg    1   2   0 wz--n- <10.00g       0 
```
Now add the 500M to the filesytem.
```
$ sudo lvresize -L +500M /dev/caleston_vg/data
  Size of logical volume caleston_vg/data changed from 1.00 GiB (256 extents) to <1.49 GiB (381 extents).
  Logical volume caleston_vg/data successfully resized.
$ sudo resize2fs /dev/caleston_vg/data
resize2fs 1.44.1 (24-Mar-2018)
Filesystem at /dev/caleston_vg/data is mounted on /mnt/media; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 1
The filesystem on /dev/caleston_vg/data is now 390144 (4k) blocks long.
```
Check the mount size
```
df -hP /mnt/media
Filesystem                    Size  Used Avail Use% Mounted on
/dev/mapper/caleston_vg-data  1.5G  3.0M  1.4G   1% /mnt/media
```


