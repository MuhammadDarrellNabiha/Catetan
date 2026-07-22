# Linux Storage Administration Commands

## Device Discovery

### lsblk

``` bash
lsblk
lsblk -f
lsblk -o NAME,SIZE,FSTYPE,MOUNTPOINTS
lsblk -b
lsblk -a
lsblk -p
lsblk -J
lsblk -D
```

Opsi: `-f -o -p -b -a -J -D`

### blkid

``` bash
blkid
blkid /dev/nvme0n1p1
blkid -o full
blkid -o export
```

### findmnt

``` bash
findmnt
findmnt /
findmnt /dev/nvme0n1p1
findmnt -o TARGET,SOURCE,FSTYPE,OPTIONS
findmnt -t ext4
findmnt -rn
```

### df

``` bash
df
df -h
df -Th
df -i
df -x tmpfs
```

### du

``` bash
du -sh
du -ah
du -xh
du --max-depth=1
```

## Partitioning

### fdisk

``` bash
fdisk /dev/nvme0n1
```

Interactive: - m help - g GPT - o DOS/MBR - n new partition - d delete -
t type - p print - l list type - w write - q quit

### gdisk

``` bash
gdisk /dev/nvme0n1
```

### parted

``` bash
parted /dev/nvme0n1
parted -l
parted -s
```

Perintah: - print - mklabel gpt - mkpart - resizepart - rm - name -
set - align-check

### sfdisk

``` bash
sfdisk
```

## Formatting

### ext4

``` bash
mkfs.ext4
mkfs.ext4 -L DATA /dev/sdb1
mkfs.ext4 -m 1 /dev/sdb1
mkfs.ext4 -b 4096 /dev/sdb1
```

Opsi: `-F -L -U -m -b -O -E`

### XFS

``` bash
mkfs.xfs
mkfs.xfs -f
mkfs.xfs -L STORAGE
```

Opsi: `-f -L -m -d -l -s`

### FAT32

``` bash
mkfs.vfat -F 32 -n USB /dev/sdb1
```

## Label

``` bash
e2label /dev/sdb1 DATA
xfs_admin -L DATA /dev/sdb1
xfs_admin -U generate /dev/sdb1
```

## Mount

``` bash
mount /dev/sdb1 /mnt
mount UUID=<uuid> /mnt
mount LABEL=DATA /mnt
mount -a
mount -o remount,rw /
mount --bind /home /mnt/test
```

Mount options: - defaults - rw - ro - noexec - nosuid - nodev -
relatime - noatime - discard - sync - async - remount

## Umount

``` bash
umount /mnt
umount /dev/sdb1
umount -l
umount -f
umount -R
```

## Filesystem Check

``` bash
fsck
fsck.ext4
xfs_repair
```

## Metadata

``` bash
dumpe2fs -h /dev/sdb1
dumpe2fs -g /dev/sdb1
tune2fs -l /dev/sdb1
tune2fs -L DATA /dev/sdb1
tune2fs -U random /dev/sdb1
xfs_info /dev/sdb1
xfs_admin /dev/sdb1
```

## Kernel Information

``` bash
cat /proc/mounts
cat /proc/self/mountinfo
cat /proc/partitions
```

## Device Information

``` bash
blockdev --getss /dev/sdb
blockdev --getsize64 /dev/sdb
smartctl -a /dev/sda
smartctl -H /dev/sda
nvme list
nvme smart-log /dev/nvme0
nvme id-ctrl /dev/nvme0
fstrim -av
file -s /dev/sdb1
iostat
iotop
dstat
```

## Workflow

``` text
lsblk
↓
fdisk / parted / gdisk
↓
mkfs
↓
blkid / lsblk -f
↓
mkdir
↓
mount
↓
findmnt
↓
df
↓
/etc/fstab
↓
mount -a
```
