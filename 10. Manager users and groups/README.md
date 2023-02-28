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
Passwords are normally defaulted to `99,999 days`

The default for `new` user creation are stored in `/etc/login.defs`
```
[root@rhcsa-node-1 ~]# cat /etc/login.defs | grep PASS_                         
#       PASS_MAX_DAYS   Maximum number of days a password may be used.          
#       PASS_MIN_DAYS   Minimum number of days allowed between password changes.
#       PASS_MIN_LEN    Minimum acceptable password length.                     
#       PASS_WARN_AGE   Number of days warning given before a password expires. 
PASS_MAX_DAYS   99999                                                           
PASS_MIN_DAYS   0                                                               
PASS_MIN_LEN    5                                                               
PASS_WARN_AGE   7                                                               
```
If we would like to change the password policy for existing uses, we will want to use the `chage` command
```
[root@rhcsa-node-1 ~]# chage --mindays 7 --maxdays 90 --warndays 5 user1
[root@rhcsa-node-1 ~]# chage --list user1                               
Last password change                                    : Feb 28, 2023  
Password expires                                        : May 29, 2023  
Password inactive                                       : never         
Account expires                                         : never         
Minimum number of days between password change          : 7             
Maximum number of days between password change          : 90            
Number of days of warning before password expires       : 5             
```
If we would like to expire an account, we can user the `-E | --expiredate` flag
```
chage -E "2030-01-20" user1 # expire date format is YYY-MM-DD
chage -E "-1" # account will never expire
```

# 3. Create, delete, and modify local groups and group memberships

To create a new group
```
groupadd group1
```
To override users primary group
```
usermod -g group2 user1 # this will override user1 group from group1 --> group2
```
To override users supplementary group
```
usermod -G wheel user1 # overwrite existing
usermod -aG wheel user1 # append to existing
```
```
