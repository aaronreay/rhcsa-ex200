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

