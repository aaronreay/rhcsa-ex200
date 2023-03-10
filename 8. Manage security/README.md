# 1. Configure firewall settings using firewall-cmd/firewalld
> **&#9432; NOTE:** We can use cockpit in order to manage `firewalld` settings

## starting
ensure `firewalld` is started, and our configuration is sane
```
[root@rhcsa-node-1 ~]# systemctl unmask firewalld      
[root@rhcsa-node-1 ~]# systemctl enable firewalld --now
[root@rhcsa-node-1 ~]# firewall-cmd --check-config     
success                                                
[root@rhcsa-node-1 ~]# firewall-cmd --list-all         
public (active)                                        
  target: default                                      
  icmp-block-inversion: no                             
  interfaces: enp1s0                                   
  sources: 192.168.122.0/24                            
  services: cockpit dhcpv6-client ntp ssh              
  ports: 123/udp 22/tcp                                
  protocols:                                           
  forward: no                                          
  masquerade: no                                       
  forward-ports:                                       
  source-ports:                                        
  icmp-blocks:                                         
  rich rules:                                          
```

## Services
All of `firewalld` pre-defined services reside within `/usr/lib/firewalld/services`
We can also create our own within the `/etc/firewalld/services` directory

```
[root@rhcsa-node-1 ~]# firewall-cmd --list-services
cockpit dhcpv6-client ntp ssh                      

[root@rhcsa-node-1 ~]# firewall-cmd --get-services # list all available pre-defined services

[root@rhcsa-node-1 ~]# firewall-cmd --add-service=cockpit --permanent
success                                                              
[root@rhcsa-node-1 ~]# firewall-cmd --reload                         
success                                                              
```
removal is done via `--remove-service`

we can also create service files via the following
```
firewall-cmd --new-service-from-file=<service.xml> --name <service_name> --permanent
```
we can copy a service template from one of the existing services already

## ports
```
[root@rhcsa-node-1 ~]# firewall-cmd --add-port=22/tcp --permanent
success                                                          
[root@rhcsa-node-1 ~]# firewall-cmd --reload                     
success                                                          
[root@rhcsa-node-1 ~]# firewall-cmd --list-ports                 
22/tcp 123/udp                                                   
```
removal is done via `--remove-port`

## zones
`firewalld` can use multiple different `zones`, each with their own set of rules

to get out default zone
```
[root@rhcsa-node-1 ~]# firewall-cmd --get-active-zone
public                                               
  interfaces: enp1s0                                 
  sources: 192.168.122.0/24                          
```
to list available zones and change our default
```
[root@rhcsa-node-1 ~]# firewall-cmd --get-zones                    
block dmz drop external home internal nm-shared public trusted work
[root@rhcsa-node-1 ~]# firewall-cmd --set-default-zone=work        
success                                                            
[root@rhcsa-node-1 ~]# firewall-cmd --get-active-zone              
public                                                             
  sources: 192.168.122.0/24                                        
work                                                               
  interfaces: enp1s0                                               
```
we can also change the interface set to a specific `zone`
```
firewall-cmd --change-interface=eth0 --zone=work
```
this will set our `work` zone to use an interface of `eth0`


# 2. Manage default file permissions
`stat` is a command which can give you more information on details of a file
```
[reaya@rhcsa-node-1 ~]$ stat file1.txt                                        
  File: file1.txt                                                             
  Size: 0               Blocks: 0          IO Block: 4096   regular empty file
Device: fd00h/64768d    Inode: 865796      Links: 1                           
Access: (0664/-rw-rw-r--)  Uid: ( 1000/   reaya)   Gid: ( 1000/   reaya)      
Context: unconfined_u:object_r:user_home_t:s0                                 
Access: 2023-02-27 21:27:13.973510855 +0000                                   
Modify: 2023-02-27 21:27:13.973510855 +0000                                   
Change: 2023-02-27 21:27:13.973510855 +0000                                   
 Birth: 2023-02-27 21:27:13.973510855 +0000                                   
```
As we know, linux has three types of permissions on a file:
* `4 (r)` - read
* `2 (w)` - write
* `1 (x)` - execute

However, we also have three `special` permissions:
* `setuid` `(u+s)` - set user id bit. This only works on a file, and only a file which is `executable`. When the executable is run
it will set permissions to that of the owner of the file, rather than the user who launched it. When you think about
it, this is very **_DANGEROUS_**.
* `setgid` `(g+s)`- set group id bit. Works on both `files` and `directories`. When used on a file, it acts in the same sense as `setuid`,
executing as the group of the user who owns it, rather than the user who executed it. When the bit is set for a directory, the files within the directory
will have the same group as the group of the parent directory. This is very useful for file sharing.
* `sticky bit` `(+t)` - When set on a directory, files can only be deleted or renamed by the file owner, directory owner, and root user, even with `777` permissions. Think of `/tmp` as a good example. we have the sticky bit `t` option if you run `ls -ld /tmp`
```
[reaya@rhcsa-node-1 tmp]$ touch ajr2          
[reaya@rhcsa-node-1 tmp]$ chmod u+s ajr2      
[reaya@rhcsa-node-1 tmp]$ ls -l ajr2          
-rwSrw-r--. 1 reaya reaya 0 Feb 27 21:53 ajr2 
[reaya@rhcsa-node-1 tmp]$ chmod u-s ajr2      
[reaya@rhcsa-node-1 tmp]$ chmod g+s ajr2      
[reaya@rhcsa-node-1 tmp]$ ls -l ajr2          
-rw-rwSr--. 1 reaya reaya 0 Feb 27 21:53 ajr2 
[reaya@rhcsa-node-1 tmp]$ chmod g-s ajr2      
[reaya@rhcsa-node-1 tmp]$ chmod +t ajr2       
[reaya@rhcsa-node-1 tmp]$ ls -l ajr2          
-rw-rw-r-T. 1 reaya reaya 0 Feb 27 21:53 ajr2 
```
The reason all of the special permissions above are in `UPPERCASE` is due to the fact that the underlying `execute` bit is not set.

We can also represent these in `octal` format
```
[reaya@rhcsa-node-1 tmp]$ chmod 4744 ajr2 ( SETUID )
[reaya@rhcsa-node-1 tmp]$ chmod 2644 ajr2 ( SGID )
[reaya@rhcsa-node-1 tmp]$ chmod 1755 ajr2 ( STICKY )
```

# 3. Configure key-based authentication for SSH
to generate a new ssh-key, we can run `ssh-keygen`
These will generate the files in `~/.ssh/`:
* `id_rsa` - this is your `private` ssh key. This should never leave the host machine
* `id_rsa.pub` - this is your `public` ssh key. This is stored in another hosts `~/.ssh/authorized_keys` directory.
This can allow passwordless logins

we can user `ssh-copy-id` to copy our public key over
```
[reaya@rhcsa-node-1 .ssh]$ ssh-copy-id rhcsa-node-2                                                                 
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/reaya/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
reaya@rhcsa-node-2's password:                                                      
                                          
Number of key(s) added: 1                                                           
                                          
Now try logging into the machine, with:   "ssh 'rhcsa-node-2'"                                                      
and check to make sure that only the key(s) you wanted were added.                                                  
```

If we want to disable `PasswordAuthentication` we can run
```
[root@rhcsa-node-1 ~]# sed -i 's/PasswordAuthentication yes/PasswordAuthentication no/g' /etc/ssh/sshd_config
[root@rhcsa-node-1 ~]# cat /etc/ssh/sshd_config | grep -i passwordauthentication
#PasswordAuthentication no
PasswordAuthentication no
# PasswordAuthentication.  Depending on your PAM configuration,
# PAM authentication, then enable this but set PasswordAuthentication
```

# 4. Set enforcing and permissive modes for SELinux
In order to check the current status of `selinux` we can run
```
[root@rhcsa-node-1 ~]# getenforce              
Enforcing                                      

[root@rhcsa-node-1 ~]# sestatus                
SELinux status:                 enabled        
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux   
Loaded policy name:             targeted       
Current mode:                   enforcing      
Mode from config file:          enforcing      
Policy MLS status:              enabled        
Policy deny_unknown status:     allowed        
Memory protection checking:     actual (secure)
Max kernel policy version:      33             
```
We have two modes to SELinux:
* `enforcing (1)` - enabled and enforcing
* `permissive (0)` - enabled and non-enforcing (will only alert)

To change our modes we can run
```
[root@rhcsa-node-1 ~]# setenforce 1
[root@rhcsa-node-1 ~]# getenforce  
Enforcing                          
```
We can also edit the config file `/etc/selinux/config`

# 5. List and identify SELinux file and process context

## Process context
to view the process context we can run
```
[root@rhcsa-node-1 ~]# ps -eZ | grep passwd # `passwd` run in a seperate terminal
unconfined_u:unconfined_r:passwd_t:s0-s0:c0.c1023 15400 pts/3 00:00:00 passwd 
```
Here we can see that we have a type context of `passwd_t`, even though `passwd` normally has a type context
of `password_exec_t`. This is due to the fact that types define a domain for processes, and a type for files

## File context
to view the file context we can run
```
[reaya@rhcsa-node-1 ~]$ touch ajr       
[reaya@rhcsa-node-1 ~]$ ls -Z ajr       
unconfined_u:object_r:user_home_t:s0 ajr
```
* user - `unconfined_u`
* role - `object_r`
* type - `user_home_t`
* level - `s0`

`SELinux` policy rules are only checked after Discretionary Access Controls (DAC) rules

to check a directory we canr un `ls -dZ /tmp`

# 6. Restore default file contexts
to restore default file contexts we can use the `restorecon` command
```
[reaya@rhcsa-node-1 ~]$ chcon -t httpd_sys_content_t ajr
[reaya@rhcsa-node-1 ~]$ ls -lZ ajr
-rw-rw-r--. 1 reaya reaya unconfined_u:object_r:httpd_sys_content_t:s0 0 Feb 27 22:53 ajr
```
Here we can see we've changed the `type` context from `user_home_t` to `httpd_sys_content_t`. This is 
incorrect, so we want to change it back
```
[reaya@rhcsa-node-1 ~]$ restorecon -v ajr
Relabeled /home/reaya/ajr from unconfined_u:object_r:httpd_sys_content_t:s0 to unconfined_u:object_r:user_home_t:s0
```
If we want to run this on an entire directory, we can specify the `-R` flag

# 7. Manage SELinux port labels
Each `port` within Linux has a defined `type` context assigned to it
```
[root@rhcsa-node-1 ~]# semanage port -l | grep ssh
ssh_port_t                     tcp      22        

[root@rhcsa-node-1 ~]# semanage port -l  | grep -i unreserved  
unreserved_port_t              sctp     1024-65535             
unreserved_port_t              tcp      61000-65535, 1024-32767
unreserved_port_t              udp      61000-65535, 1024-32767
```
We can see that `ssh` on port `22` has a default type context of `ssh_port_t`
However, we can also see that ports in the range of `1024-32767` for both `udp/tcp` are unreserved.
This does not count for ports that fall within that range, that do have a specific mapping to them

For example, if you attempted to spool `sshd` on port `1300`, it would fail, as this is then assigned
a context of `unreserved_port_t`. To overcome this, we can change the actual port type of `ssh_port_t`
```
[root@rhcsa-node-1 ~]# semanage port -a -t ssh_port_t -p tcp 1300
[root@rhcsa-node-1 ~]# semanage port -l | grep ssh_port_t        
ssh_port_t                     tcp      1300, 22                 
```
to remove this, we can specify the `-d | --delete` flag, rather than the `-a | --add` flag.

# 8. Use boolean settings to modify system SELinux settings
`SELinux` booleans allow for policy changes to be made during runtime, without the need of understanding how to write policies

To view the list of SELinux booleans we have available we can run:
```
[root@rhcsa-node-1 ~]# semanage boolean -l # list all booleans
[root@rhcsa-node-1 ~]# semanage boolean -l -C # list booleans with local customisation
```

To enable booleans, we have two options, either through `semanage boolean` OR `setsebool`
```
[root@rhcsa-node-1 ~]# semanage boolean --modify --on httpd_manage_ipa   
[root@rhcsa-node-1 ~]# semanage boolean -l -C | grep -i httpd_manage_ipa 
httpd_manage_ipa               (on   ,   on)  Allow httpd to manage ipa  

[root@rhcsa-node-1 ~]# setsebool -P httpd_manage_ipa 1                  
[root@rhcsa-node-1 ~]# semanage boolean -l -C | grep -i httpd_manage_ipa
httpd_manage_ipa               (on   ,   on)  Allow httpd to manage ipa 
```

# 9. Diagnose and address routine SELinux policy violations
One of the most useful tools for diagnosing SELinux issues is `setroubleshoot`
The package can be used in the `GUI` OR `CLI`

Install it with `dnf install setroubleshoot -y`

For our scenario, we will attempt to boot `httpd` on port `1234`. When the service fails to start, we can look in 
a couple different places

```
[root@rhcsa-node-1 ~]# journalctl -xef
Feb 27 23:32:54 rhcsa-node-1.lab setroubleshoot[15785]: SELinux is preventing /usr/sbin/httpd from name_bind access on the tcp_socket port 1234. For complete SELinux messages run: sealert -l a0cc650b-3565-4726-a383-2abb00a85ace
```
Here we can see our `setroubleshootd` daemon has found a policy violation with `httpd`. Lets view this message
```
[root@rhcsa-node-1 ~]# sealert -l a0cc650b-3565-4726-a383-2abb00a85ace
SELinux is preventing /usr/sbin/httpd from name_bind access on the tcp_socket port 1234.

*****  Plugin catchall (100. confidence) suggests   **************************

If you believe that httpd should be allowed name_bind access on the port 1234 tcp_socket by default.
Then you should report this as a bug.
You can generate a local policy module to allow this access.
Do
allow this access for now by executing:
# ausearch -c 'httpd' --raw | audit2allow -M my-httpd
# semodule -X 300 -i my-httpd.pp
```
Here we can see it shows us that we are failing due to trying to bind on port `1234`. We get a couple of options to fix this, but what do they do?

`ausearch` is a way to parse audit daemon logs. We can use this with the `-c 'httpd'` flag in order to search for any event with this name
```
[root@rhcsa-node-1 ~]# ausearch -c 'httpd'
type=PROCTITLE msg=audit(1677599954.859:217): proctitle=2F7573722F7362696E2F6874747064002D44464F524547524F554E44
type=SYSCALL msg=audit(1677599954.859:217): arch=c000003e syscall=49 success=no exit=-13 a0=3 a1=563f199878c0 a2=10 a3=7fffc751fc0c items=0 ppid=1 pid=3209 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="httpd" exe="/usr/sbin/httpd" subj=system_u:system_r:httpd_t:s0 key=(null)type=AVC msg=audit(1677599954.859:217): avc:  denied  { name_bind } for  pid=3209 comm="httpd" src=1234 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:monopd_port_t:s0 tclass=tcp_socket permissive=0
```
With `ausearch`, we can pipe the output to `audit2allow`, which will create us a custom module
```
[root@rhcsa-node-1 ~]# ausearch -c 'httpd' --raw | audit2allow -M my-httpd 
******************** IMPORTANT ***********************                     
To make this policy package active, execute:                               
                                                                           
semodule -i my-httpd.pp                                                    
```
Here we can see that if we run `semodule -i my-httpd.pp`, we can enable this policy
```
[root@rhcsa-node-1 ~]# semodule -i my-httpd.pp
[root@rhcsa-node-1 ~]# systemctl restart httpd                                              
[root@rhcsa-node-1 ~]# systemctl status httpd                                               
??? httpd.service - The Apache HTTP Server                                                    
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
   Active: active (running) since Tue 2023-02-28 16:16:15 GMT; 3s ago                       
```
We can also have a look at the contents of what this module actually does - `my-httpd.pe`
```
module my-httpd 1.0;                             
                                                 
require {                                        
        type httpd_t;                            
        type monopd_port_t;                      
        class tcp_socket name_bind;              
}                                                
                                                 
#============= httpd_t ==============            
allow httpd_t monopd_port_t:tcp_socket name_bind;
```
To remove this custom module, we can run `semodule -r my-httpd`
