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
* `setuid` - set user id bit. This only works on a file, and only a file which is `executable`. When the executable is run
it will set permissions to that of the owner of the file, rather than the user who launched it. When you think about
it, this is very **_DANGEROUS_**.
