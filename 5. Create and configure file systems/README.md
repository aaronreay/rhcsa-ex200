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

# 2. Mount and unmount network file systems using NFS

we will not have to configure an NFS server in the exam, but we will create one for our lab, on `rhcsa-node-2`
```
dnf install nfs-utils -y
systemctl enable --now nfs-server rpcbind
parted /dev/vdb mklabel gpt mkpart 1 ext4 2048s 100%
pvcreate /dev/vdb1
vgcreate vg_data /dev/vdb1
lvcreate -l +100%FREE vg_data -n lv_data
mkfs.ext4 /dev/vg_data/lv_data
mkdir /data
echo -e '/dev/mapper/vg_data-lv_data /data ext4 defaults 1 2' >> /etc/fstab
mount -a
mkdir -p /data/nfshare
echo -e '/data/nfshare 192.168.122.0/24(rw,no_root_squash)' > /etc/exports
exportfs -rav
firewall-cmd --add-service=nfs --permanent
firewall-cmd --add-service={nfs3,mountd,rpc-bind} --permanent
firewall-cmd --reload
```

## mounting our nfshare
```
dnf install nfs-utils -y
[root@rhcsa-node-1 ~]# showmount -e rhcsa-node-2
Export list for rhcsa-node-2:
/data/nfshare 192.168.122.0/24
[root@rhcsa-node-1 /]# mount rhcsa-node-2:/data/nfshare /data
# we can also specify to mount as type nfs
[root@rhcsa-node-1 /]# mount -t nfs rhcsa-node-2:/data/nfshare /data
echo -e 'rhcsa-node-2:/data/nfshare /data nfs defaults 0 0' >> /etc/fstab # we do not do fsck checking or dump on nfs share

```

# 3. Configure autofs
`AutoFS` provides automounting of network shares when they are accessed
* `/etc/auto.master` - main template file which can point to one or more other templates
* `/etc/auto.master.d` - includes all files with `*.autofs` appended - 

```mermaid
graph TD;
a-->b;
a-->c;
```