# RAID10 Software RAID Setup

-----------------------------------------------------------------------------------
Check if HW Raid Controller is present. If present Use HW RAID instead of SW RAID.
```
  lspci -knn | grep 'RAID bus controller'
```
-----------------------------------------------------------------------------------

## Prerequisite:
1. Hardware
  4 Disk Drives
2. Software
  ```
  sudo apt-get update && sudo apt-get install lsscsi mdadm xfsprogs
  ```
## Setup Steps:

1. Check if already any SW RAID setup exists in System.
  ```
  cat /proc/mdstat
  ```
  If exists you can remove it with following sample commands
  ```
  umount /dev/md0
  mdadm --stop /dev/md0
  mdadm --zero-superblock /dev/sda
  mdadm --zero-superblock /dev/sdb
  mdadm --zero-superblock /dev/sdc
  mdadm --zero-superblock /dev/sdd
  ```
2. Format all drives with xfs file system
  ```
  sudo mkfs.xfs /dev/sda
  sudo mkfs.xfs /dev/sdb
  sudo mkfs.xfs /dev/sdc
  sudo mkfs.xfs /dev/sdd
  ```
2. Create RAID10 Partition
  ```
  sudo mdadm -v --create /dev/md0 --level=raid10 --raid-devices=4 /dev/sda /dev/sdb /dev/sdc /dev/sdd
  ```
3. Format RAID10 
  ```
  sudo mkfs.xfs /dev/md0
  ```
4. Create path for mounting the RAID drive
  ```
  sudo mkdir /data
  sudo chown -R admin:admin /data
  sudo chmod ug+rwx /data
  ```
5. Get UUID for the md0 disc from the output of the following command.
  ```
  lsblk -o NAME,UUID
  ```
  Sample Responce will be similar to shown below
  ```
  NAME    UUID
sda     26928fde-2ae5-0870-1376-fd04df690ffd
└─md0   b5f32795-6aa0-4d9e-b08d-f653cf78abe8
sdb     26928fde-2ae5-0870-1376-fd04df690ffd
└─md0   b5f32795-6aa0-4d9e-b08d-f653cf78abe8
sdc     26928fde-2ae5-0870-1376-fd04df690ffd
└─md0   b5f32795-6aa0-4d9e-b08d-f653cf78abe8
sdd     26928fde-2ae5-0870-1376-fd04df690ffd
└─md0   b5f32795-6aa0-4d9e-b08d-f653cf78abe8
sde     eeac5e41-16d1-409d-bce5-603eee0b206b
sdf
├─sdf1  b7787947-9088-4fba-bad3-158c2f2efcde
├─sdf2  cecce18c-eea6-418d-96cb-23c7801e10fe
└─sdf3  5bdf4309-8ee9-4eec-a292-4de23e4603a6

  ```
6. Add the partition to /etc/fstab for mounting on boot
  Append following line to the file /etc/fastb where the unique value of UUID taken from lsblk should be used.
  ```
  UUID=b5f32795-6aa0-4d9e-b08d-f653cf78abe8 /data           xfs     rw,user,auto      0       0
  ```
7. Reboot System
