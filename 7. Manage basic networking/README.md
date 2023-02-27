# 1. Configure IPv4 and IPv6 addresses

RHEL 7+ onwards no longer uses network-scripts (although supported).
`NetworkManager` is the go-to network configuration tool

> **&#9432; NOTE:** we can use nmtui to configure our network devices, which is a text-based tool

To show our current configuration
```
[root@rhcsa-node-1 ~]# nmcli con show                         
NAME    UUID                                  TYPE      DEVICE
enp1s0  e66ac25b-0ae4-4553-8105-66b3a9d45952  ethernet  enp1s0
```
If we want to add a new connection
```
[root@rhcsa-node-1 ~]# nmcli con add con-name net0 ifname eth0 type ethernet ipv4.address 192.168.122.12/24 ipv4.gateway 192.168.122.1
Connection 'net0' (a50f673b-c56c-4e11-bbc6-035c211f5c43) successfully added.
```
If we want to add DNS servers to our new connection
```
[root@rhcsa-node-1 ~]# nmcli con mod net0 ipv4.dns "8.8.8.8,1.1.1.1"
[root@rhcsa-node-1 ~]# nmcli con show net0 | grep ipv4.dns
ipv4.dns:                               8.8.8.8,1.1.1.1
```
We also have the option to change our `firewalld` zone interface with the following:
```
[root@rhcsa-node-1 ~]# firewall-cmd --get-active-zones
public                                                
  interfaces: enp1s0                                  

[root@rhcsa-node-1 ~]# firewall-cmd --permanent --zone=public --change-interface=net0
success
[root@rhcsa-node-1 ~]# firewall-cmd --reload
success
[root@rhcsa-node-1 ~]# firewall-cmd --get-active-zones
public
  interfaces: net0 enp1s0
```

# 2. Configure hostname resolution

in `RHEL 7+` we have the `systemd-hostnamed` daemon. We can change our hostname with `hostnamectl`
```
[root@rhcsa-node-1 ~]# hostnamectl set-hostname rhcsa-node-1.lab    
[root@rhcsa-node-1 ~]# systemctl restart systemd-hostnamed.service  
[root@rhcsa-node-1 ~]# hostnamectl --static                         
rhcsa-node-1.lab                                                    
```

If we want to add `DNS` to our host
```
[root@rhcsa-node-1 ~]# nmcli con show                          
NAME    UUID                                  TYPE      DEVICE 
enp1s0  e66ac25b-0ae4-4553-8105-66b3a9d45952  ethernet  enp1s0 

[root@rhcsa-node-1 ~]# nmcli con modify enp1s0 +ipv4.dns 8.8.8.8,1.1.1.1
```
If we want to add a `domain`
```
[root@rhcsa-node-1 ~]# nmcli con modify enp1s0 +ipv4.dns-search example.com
```
To ignore the default dns set with dhcp
```
[root@rhcsa-node-1 ~]# nmcli con modify enp1s0 ipv4.ignore-auto-dns yes
```

# 3. Configure network services to start automatically at boot
```
# Let's check the httpd service
[root@rhcsa-node-1 ~]# systemctl list-unit-files | grep httpd.service 
httpd.service                                    disabled

# Other types of units can be shown with
[root@rhcsa-node-1 ~]# systemctl -t help 
Available unit types:                    
service                                  
socket                                   
target                                   
device                                   
mount                                    
automount                                
swap                                     
timer                                    
path                                     
slice                                    
scope                                    

# To enable and start our new service
[root@rhcsa-node-1 ~]# systemctl enable httpd.service --now
Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → /usr/lib/systemd/system/httpd.service.
[root@rhcsa-node-1 ~]# systemctl status httpd.service
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset:>
   Active: active (running) since Mon 2023-02-27 18:18:36 GMT; 5s ago
```

# 4. Restrict network access using firewall-cmd/firewall


