# 1. List, create and delete partitions on MBR and GPT disks

## Master Boot Record partitioning scheme (MBR) (BIOS)
* Maximum of four primary partitions
* Can create arbitary amount of logical/extend partitions on one of these partitions

## GUID Partition Table (GPT) (UEFI)
* 128 partitions
* Uses a `GUID` to identify each disk and partition
* Has a backup copy, the secondary GPT, which is housed at the end of the disk

## Parted
`Parted` can allow us to manage partitions on both `MBR` and `GPT`.
```
parted /dev/vdb print # display partition table on vdb
parted /dev/vdb # open a session for vdb
```
To make a `MBR` label we can use `parted /dev/vdb mklabel msdos`
To make a `GPT` label we can use `parted /dev/vdb/mklabel gpt`

### Creating MBR Partitions
```
parted /dev/vdb
mklabel msdos
mkpart primary
ext4
Start? 2048s # 2048 sectors is a 1 MiB alignment
End? 1000MB
# Size = End - Start
quit
udevadm settle # Waits for system to detect new partition and created mapping in /dev*

# Interactive mode
parted /dev/vdb mklabel msdos mkpart primary ext4 2048s 1000MB
```

### Creating GPT Partitions
```
parted /dev/vdb
mklabel gpt
mkpart data # we specify a name for our partition, instead of primary/extended
ext4
Start? 2048s
End? 1000MB
# Size = End - Start
quit
udevadm settle

# Interactive mode
parted /dev/vdb mklabel gpt mkpart data ext4 2048s 1000MB
```

### Deleting Partitions
This is the same process for either `MBR` or `GPT`
```
parted /dev/vdb
print # this will show the list of available partitions
rm <partition_number>
quit
```
we can also use `wipefs -a` to remove all headers from a disk afterwards. This will remove the label of `MBR` or `GPT`

## fdisk
`fdisk` is a CLI tool used on `MBR` partitions
```
fdisk -l /dev/vdb # lists partitions of vdb
n # creates a new partition
p # primary partition (max of 4)
1
2048s # fdisk will default to 2048 sector if non-specified
+1000M # similar to parted, we specify our last sector. It will default to the entire disk otherwise
w # write and quick fdisk
```
To remove a partition with `fdisk`, simply enter back with `fdisk /dev/vdb` and run `d`

## gdisk
`gdisk` is a CLI tool used on `GPT` partitions
```
n
1 # we can create 1-128 partitions
2048s # gdisk will default to 2048 sector if non-specified
+1000MB
w
```
To remove a partition with `gdisk`, simply enter back with `gdisk /dev/vdb` and run `d`

# 2. Create and remove physical volumes

Physical volume is a block device node, which can be created on an entire hard disk `/dev/vdb`, or also `MBR` and `GPT` partitions, `/dev/vdb1`, `/dev/vdb2`

```
[root@rhcsa-node-1 ~]# pvcreate /dev/vdb1
  Physical volume "/dev/vdb1" successfully created.
[root@rhcsa-node-1 ~]# pvremove /dev/vdb1
  Labels on physical volume "/dev/vdb1" successfully wiped.
```

# 3. Assign physical volumes to volume groups
```
[root@rhcsa-node-1 ~]# vgcreate vg_data /dev/vdb1
  Physical volume "/dev/vdb1" successfully created.
  Volume group "vg_data" successfully created
```

# 4. Create and delete logical volumes
```
[root@rhcsa-node-1 ~]# lvcreate -L 1G vg_data -n lv_data
  Logical volume "lv_data" created.

# We can also use the below to use the entire volume group
lvcreate -l 100%FREE vg_data -n lv_data

# To remove
[root@rhcsa-node-1 ~]# lvremove /dev/vg_data/lv_data 
Do you really want to remove active logical volume vg_data/lv_data? [y/n]: y
  Logical volume "lv_data" successfully removed.
```

# 5. Configure systems to mount file systems at boot by UUID or label
we can use `blkid` to obtain the UUID of our block device
```
[root@rhcsa-node-1 ~]# blkid /dev/vdb1
/dev/vdb1: UUID="UwrxaS-FbQ5-jFDy-yn0C-cqxK-TNO3-PMwDgj" TYPE="LVM2_member" PARTLABEL="Linux filesystem" PARTUUID="f32ee919-4f78-4b23-b3a1-39d6d9a59dae"
```
we can use `e2label` to assign a label to our ext4 filesystem
```
[root@rhcsa-node-1 ~]# mkfs.ext4 /dev/vdb1
[root@rhcsa-node-1 ~]# e2label /dev/vdb1
[root@rhcsa-node-1 ~]# e2label /dev/vdb1 "new_disk"
[root@rhcsa-node-1 ~]# e2label /dev/vdb1
new_disk
```

## mount with label
```
echo -e 'LABEL="new_disk" /mnt/tmp ext4 defaults 1 2' >> /etc/fstab
```



