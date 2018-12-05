# RAID10 Software RAID Setup

-----------------------------------------------------------------------------------
Check if HW Raid Controller is present. If present Use HW RAID instead of SW RAID.
```
  lspci -knn | grep 'RAID bus controller'
```
-----------------------------------------------------------------------------------

## Prerequisite:
 Hardware
  4 Disk Drives
 Software
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
  mdadm --zero-superblock /dev/sde
  mdadm --zero-superblock /dev/sdf
  ```

2. Format all drives with xfs file system
  ```
  sudo mkfs.xfs /dev/sda
  sudo mkfs.xfs /dev/sdb
  sudo mkfs.xfs /dev/sde
  sudo mkfs.xfs /dev/sdf
  ```
2. Create RAID10 Partition
  ```
  sudo mdadm -v --create /dev/md0 --level=raid10 --raid-devices=4 /dev/sda /dev/sdb /dev/sde /dev/sdf
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
6. Add the partition to /etc/fstab for mounting on boot
  Append following line to the file /etc/fastb where the unique value of UUID taken from lsblk should be used.
  ```
  UUID=b5f32795-6aa0-4d9e-b08d-f653cf78abe8 /data           xfs     rw,user,auto      0       0
  ```
7. Reboot System
