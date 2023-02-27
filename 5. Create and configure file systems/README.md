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
* `/etc/auto.master.d` - template files can be stored here, which `/etc/auto.master` will bring in

For our case, we can create the file `/etc/auto.master.d/data.autofs`, with the following contents:
```
/- /etc/auto.data
```
From here we can see that our template file will mount our data to a specific folder, with our mapping details located in `/etc/auto.data`

The contents of `/etc/auto.data` can be shown below:
```
/data/nfshare rhcsa-node-2:/data/nfshare
```

Now we can start autofs, check if the folder will be mounted when we attempt to access it
```
[root@rhcsa-node-1 /]# systemctl start autofs
[root@rhcsa-node-1 /]# df -kh | grep -i nfshare
[root@rhcsa-node-1 /]# cd /data/nfshare
[root@rhcsa-node-1 nfshare]# df -kh | grep -i nfshare
rhcsa-node-2:/data/nfshare     9.8G  256K  9.3G   1% /data/nfshare
```
Above we can see that the folder is only mounted on the filesystem when we actually attempt to access it, after starting the `autofs.service` daemon.

# 4. Extend existing logical volumes
For our case, let's assume we have `/dev/vg_data/lv_data` of `5G`, and we want to extend this LVM to 10G.
```
[root@rhcsa-node-1 /]# pvcreate /dev/vdb2
  Physical volume "/dev/vdb2" successfully created.
[root@rhcsa-node-1 /]# vgextend vg_data /dev/vdb2
  Volume group "vg_data" successfully extended
[root@rhcsa-node-1 /]# lvextend -l +100%FREE /dev/vg_data/lv_data
  Size of logical volume vg_data/lv_data changed from <5.00 GiB (1279 extents) to 9.99 GiB (2558 extents).
  Logical volume vg_data/lv_data successfully resized.
[root@rhcsa-node-1 /]# resize2fs /dev/vg_data/lv_data
```
If we had an `xfs` filesystem on our `LVM`, then we can use `xfs_grow /dev/vg_data/lv_data`.

If we want to resize an `ext4` filesystem in one command, we can run `lvextend -r -l +100%FREE /dev/vg_data/lv_data.
* `-r | --resize2fs` - resize an `ext4` filesystem

# 5. Create and configure set-GID directories for collaboration
* `setgid` - special permission. If used on a directory, all new files will inherit the group ownership of the group directory owner, not the user who created them. Useful when sharing files

For our case, let's say we have two users; `john` and `sarah`, and a common group `rocketengineers`
```
[root@rhcsa-node-1 /]# useradd john
[root@rhcsa-node-1 /]# useradd sarah
[root@rhcsa-node-1 /]# groupadd rocketengineers                                  
[root@rhcsa-node-1 /]# chown -R :rocketengineers /student_projects/rocket_science
[root@rhcsa-node-1 /]# usermod -aG rocketengineers sarah                    
[root@rhcsa-node-1 /]# usermod -aG rocketengineers john     
[root@rhcsa-node-1 /]# chmod 770 /student_projects/rocket_science/          
[root@rhcsa-node-1 /]# stat /student_projects/rocket_science/                    
  File: /student_projects/rocket_science/                                        
  Size: 6               Blocks: 0          IO Block: 4096   directory            
Device: fd00h/64768d    Inode: 34819463    Links: 2                              
Access: (0770/drwxrwx---)  Uid: (    0/    root)   Gid: ( 1006/rocketengineers)  
```
With these groups created, `john` and `sarah` can both copy documents over to this directory, and can both read them, but not write, as the permission of their file is `0664`, meaning only world readable by users not in their group

`john_doc.txt` in `/student_projects/rocket_science`
```
[john@rhcsa-node-1 rocket_science]$ stat john_doc.txt 
  File: john_doc.txt
  Size: 5               Blocks: 8          IO Block: 4096   regular file
Device: fd00h/64768d    Inode: 35490622    Links: 1
Access: (0664/-rw-rw-r--)  Uid: ( 1002/    john)   Gid: ( 1004/    john)
```
`sarah_doc.txt` in `/student_projects/rocket_science`
```
[sarah@rhcsa-node-1 ~]$ stat sarah_doc.txt                              
  File: sarah_doc.txt                                                   
  Size: 5               Blocks: 8          IO Block: 4096   regular file
Device: fd00h/64768d    Inode: 17468119    Links: 1                     
Access: (0664/-rw-rw-r--)  Uid: ( 1003/   sarah)   Gid: ( 1005/   sarah)
```
As the permissions of these files are owned wholly by `john` and `sarah`, they can only read each others document, but not write to them

To fix this, we can use the `setgid flag (s/2)`
```
[root@rhcsa-node-1 ~]# chmod g+s /student_projects/rocket_science/
[root@rhcsa-node-1 ~]# ls -ld /student_projects/rocket_science/
drwxrws---. 2 root rocketengineers 47 Feb 27 15:00 /student_projects/rocket_science/
```
Here we can see the addition of the `s` value. This is our `setgid` option. Now anytime a new document is copied into
this directory, it will inherit the group permission

```
[sarah@rhcsa-node-1 ~]$ cp sarah_doc_new.txt /student_projects/rocket_science/[s
[sarah@rhcsa-node-1 rocket_science]$ stat  sarah_doc_new.txt 
  File: sarah_doc_new.txt
  Size: 5               Blocks: 8          IO Block: 4096   regular file
Device: fd00h/64768d    Inode: 35490623    Links: 1
Access: (0664/-rw-rw-r--)  Uid: ( 1003/   sarah)   Gid: ( 1006/rocketengineers)

# we can see that the Gid is now set to rocketengineers, meaning john can now write to this file
```

# 6. Diagnose and correct file permission problems

For this topic, we will use the available script - https://gitlab.com/EddieJennings/rhcsa-practice.git
