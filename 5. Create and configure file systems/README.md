# 1. Create, mount, umount, and use vdat, ext4, and xfs file systems

before we can create a file system, we will need a block device or logical volume
```
parted /dev/vdb mklabel gpt mkpart 1 ext4 2048s 100%
pvcreate /dev/vdb1
vgcreate vg_data /dev/vdb1
lvcreate -l 100%FREE vg_data -n lv_data
```

## ext4
```
mkfs.ext4 /dev/vg_data/lv_data
echo -e '/dev/mapper/vg_data-lv_data /mnt/tmp ext4 defaults 1 2' >> /etc/fstab
mount -a
```
## xfs
```
mkfs.xfs /dev/vg_data/lv_data
```
## vfat
```
mkfs.vfat /dev/vg_data/lv_data
```
