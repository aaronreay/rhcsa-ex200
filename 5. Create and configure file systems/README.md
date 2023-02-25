# 1. Create, mount, umount, and use vdat, ext4, and xfs file systems

before we can create a file system, we will need a block device or logical volume
`parted /dev/vdb mklabel gpt mkpart 1 ext4 2048s 100%`
