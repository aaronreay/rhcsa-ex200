# 1. Create, delete. and modify local user accounts

All `UIDs` `< 1000` are reserved for system users and groups

Here are some useful directories we should take note of:
* `/etc/password` - account information
* `/etc/shadow` - secure account information
* `/etc/group` - group account information
* `/etc/gshadow` - secure group account information
* `/etc/default/useradd` - default values for creating an acount
* `/etc/skel` - directory containing default files
* `/etc/login.defs` - shadow password suite configuration

Here we add a user, with a home directory `-m`, a comment `-c`, and add supplementary group `-G` of `wheel`
Also note, a home directory will normally be added, as defined in `/etc/default/useradd`
```
[root@rhcsa-node-1 ~]# useradd -m -c "Test User 1" -G wheel user1 
[root@rhcsa-node-1 ~]# id user1                                   
uid=1001(user1) gid=1001(user1) groups=1001(user1),10(wheel)      
```
To delete the user we can run: `userdel -r user`. This will remove `mail` and `home` directories as well

# 2. Change passwords and adjust password aging for local user accounts
