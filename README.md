# KodeKloud Linux Basics: Storage in Linux  

[KodeKloud Linux Basics Course Notes Table of Contents](https://github.com/pslucas0212/LinuxBasics)

## Storage in Linux

## Storage Basics

### Block Devices
Block device is a type of file that can be found under /dev directory representing a peice of hwardware like a spinning disk or an SSD drive.  It is a called block storage cause data is written and read in blocks or "chunks" of data.

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
mcblk0 represents the entire disk and the part rerpresents partitions

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
```
$ sudo fdisk -l /dev/sdb
```
#### File Systems in Linux  
Partitioning alone do not make the disks usable.  TO write disk you must create a file system and mount to a direct to read and write data to the partition.  
  
EXT2 | EXT3 | EXT4
---- | ---- | ----
2 TB File Size | 2 TB File Size | 16 TB File Size
4 TB Volume size | 4 TB Volume size | 1 Exabyte
supports compression | Uses Journal | Uses Journal
Support Linux permissions | Backwards Compatible | Backwards Compatible
Long crash Recovery | Faster Crash Recovery | Uses chsksum for Journal
  
Creating file systems:
```
$ mkfs.ext4 /dev/sdb1
$ mkdir /mnt/ext4;
$ mount /dev/sdb1 /mnt/ext4
$ mount | grep /dev/sdb1
$ df -hP | grep /dev/sdb1
```
To make permanent after reboot after entry to /etc/fstab:  
  
Field | Purpose
----- | -------
Filesystem | Such as /dev/vdb1 to be mounted
Mountpoint | Directory to be mounted on
Type | Example ext2, ext3, ext4
Options | Such as RW or RO
Dump | 0=ignore, 1=backup
Pass | 0=ignore, 1 or 2 = FSCK filesystem check enforced
  
Add to file:
```
#<file system> <mount point>    <type>           <options>                              <dump>           <pass>
/dev/sda1      /                ext4             defaults,relatime,errors=panic            0               1 ~
```
or
```
echo "/dev/sdb1  /mnt/ext4   ext4 rw 0 0" >> /etc/fstab
```
    
 
#### DAS, NAS and SAN
Three types of storage - DAS -> Direct Attached Storage, NAS -> Network Attached Storage, SAN -> Storage Area Network (fiber attached)  
DAS - connects directly to the host system s
  

DAS | NAS | SAN
--- | --- | ---
Directly Attached | Network attached NFS/CIFS | Storage Area Network FC (Fibre Channel) or iSCSI
Block Storage | File storage | Block Storage
Fast and Reliable | Reasonably Fast and Reliable | Fast, secure and reliable
Dedicated to single host | Shared Storage | Highly Available
Ideal for small Businesses | Mid/Large Business | Enterprise Storage
Suitable for OS | Not suitable for OS | Not suitable for OS
  
 #### NFS File System  
 Saves data in form of files in client/server model.  NFS Server has /software/repos is shared across the network via a mount.  Sharing disk in NFS is called exporting.  
  
NFS server maintains exports file at /etc/exprots that maintains a list of servers that can access the folder
```
/software/repos hosta hostb hostc
```
Specific ports may have to be opened for NFS to work
```
$ exportsfs -a
$ exportsfs -o 10.16.13.201:/software/repos
```
On the server mount the drive with:
```
$ mount 10.16.13.201:/software/repos /mnt/software/repos
```
  
#### LVM
Logical Volume Manager allows grouping of multiple physcial drives into one volume group where you can then carve out logical volumes.   This allows logical volumes to be easily resized.  
   
 
Install LVM
```
$ apt-get install lvm2
$ pvcreate /dev/sdb
$ vgcreate caleston_vg /dev/sdb
$ pvdisplay
$ vgdisplay
$ lvcreate -L 1G -n vol1 caleston_vg
$ lvdisplay
$ lvs
$ mkfs.ext4 /dev/caleston_vg/vol1
$ mount -t ext4 /dev/caleston_vg/vol1 /mnt/vol1
```
Resize volumen - first see if there is enough size
```
$ vgs
$ lvresize -L +1G /dev/caleston_vg/vol1
$ dfh -hP /mnt/vol1
```
Resize file system size
```
$ resize2fs /dev/caleston_vg/vol1
$ df -hP /mnt/vol1
```
