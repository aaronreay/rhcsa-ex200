# 1. Schedule tasks using at and cron

## at
`at` is useful if you're looking to schedule a single-run of a job in the future
```
[root@rhcsa-node-1 ~]# dnf install at -y
[root@rhcsa-node-1 ~]# systemctl enable atd --now
```
Let's say we have a simple script that lists current files and prints to our terminal
```
#!/bin/bash
ls -ltr > /dev/pts/0 # use ps -a to find the tty session
```
We can use the `at` command to schedule a job
```
[root@rhcsa-node-1 ~]# at -f ls_script.sh now # will run the job now
[root@rhcsa-node-1 ~]# at -f ls_script.sh now +7 days # will run the job in 7 days time
```
To check our queue, and to remove jobs
```
[root@rhcsa-node-1 ~]# atq
2Mon Mar  6 16:17:00 2023 a root
[root@rhcsa-node-1 ~]# atrm 2
[root@rhcsa-node-1 ~]# atq
[root@rhcsa-node-1 ~]# 
```
We can also limit who can & cannot schedule jobs:
* `/etc/at.allow` - explicitly allow users to schedule jobs
* `/etc/at.deny` - prevent users from scheduling jobs

## Cron
Cron allows us to schedule jobs to run at certain intervals, multiple times

![Crontab Schedule](crontab.png)

We can use `crontab -e` to edit the crontab of the current user. Let's put our script to run at 17:00 everyday
```
[root@rhcsa-node-1 ~]# crontab -e
00 17 * * * root /root/ls_script.sh
[root@rhcsa-node-1 ~]# crontab -l
00 17 * * * root /root/ls_script.sh
```
To delete the crontab we can use `crontab -d`

We can also create a cron file, with the same contents as above, as place it in the `/etc/cron.d/*` directory

`Cron` actually has four default directories:
* `/etc/cron.weekly`
* `/etc/cron.monthly`
* `/etc/cron.hourly`
* `/etc/cron.daily`

We can simply place our `ls_script.sh` file in one of these, as these directories have pre-configured run times (as evidenced in the name)

Similar to `at`, we can also allow or deny user scheduling with the two directories:
* `/etc/cron.allow`
* `/etc/cron.deny`

# 2. Start and stop services and configure services to start automatically at boot

We can list all of our possible services with:
```
[root@rhcsa-node-1 ~]# systemctl list-units --all
```
If we want to enable and start a service:
```
[root@rhcsa-node-1 ~]# systemctl unmask atd.service # not normally required, but useful to check if a service is masked (pointed to /dev/null)
[root@rhcsa-node-1 ~]# systemctl enable atd.service --now

[root@rhcsa-node-1 ~]# systemctl status atd.service
● atd.service - Job spooling tools
   Loaded: loaded (/usr/lib/systemd/system/atd.service; enabled; vendor preset: enabled)
   Active: active (running) since Mon 2023-02-27 16:06:48 GMT; 39min ago
 Main PID: 11696 (atd)
    Tasks: 1 (limit: 11214)
   Memory: 404.0K
   CGroup: /system.slice/atd.service
           └─11696 /usr/sbin/atd -f
```
If we would like to stop and disable a service, we can do the reverse
```
[root@rhcsa-node-1 ~]# systemctl disable atd.service --now
Removed /etc/systemd/system/multi-user.target.wants/atd.service.
[root@rhcsa-node-1 ~]# systemctl mask atd.service
Created symlink /etc/systemd/system/atd.service → /dev/null.
[root@rhcsa-node-1 ~]# systemctl status atd.service
● atd.service
   Loaded: masked (Reason: Unit atd.service is masked.)
   Active: inactive (dead)
```
If we have cockpit installed, we can navigate to http://localhost:9090 and go to the `Services` tab

# 3. Configure systems to boot into a specific target automatically

We can check our current default target with:
```
[root@rhcsa-node-1 ~]# systemctl get-defaul
multi-user.target
```
a `*.target` unit allows us to group units via dependencies

To list all possible targets
```
[root@rhcsa-node-1 ~]# systemctl list-units --type target # only lists active
[root@rhcsa-node-1 ~]# systemctl list-units --type target --all # lists both active and disabled
```

To set our default target
```
[root@rhcsa-node-1 ~]# systemctl set-default basic.target
Removed /etc/systemd/system/default.target.
Created symlink /etc/systemd/system/default.target → /usr/lib/systemd/system/basic.target.
```
As we can see, it's simply a `symlink` from our `target` file in `/usr/lib/systemd/system/*` to running config of `/etc/systemd/system/*`

# 4. Configure time service clients
## date
We can print out the current time and date with the `date` command
```
[root@rhcsa-node-1 ~]# date '+%Y-%m-%d %H:%M'
2023-02-27 17:11                             
```
This runs the `date` command with the following variables:
* `%Y` - year
* `%m` - month
* `%d` - day of month
* `%H` - hour
* `%M` - minute

## hwclock
set system clock to hardware clock `OR` set hardware clock to system clock
```
[root@rhcsa-node-1 ~]# hwclock -s
[root@rhcsa-node-1 ~]# hwclock -w
```

## timedatectl
`timedatectl` shows us our current time, date, timezone, and whether we are synchronised or not
```
[root@rhcsa-node-1 ~]# timedatectl                     
               Local time: Mon 2023-02-27 17:16:39 GMT 
           Universal time: Mon 2023-02-27 17:16:39 UTC 
                 RTC time: Mon 2023-02-27 17:16:39     
                Time zone: Europe/London (GMT, +0000)  
System clock synchronized: no                          
              NTP service: n/a                         
          RTC in local TZ: no                          
```
Setting time manually with `timedatectl`
```
[root@rhcsa-node-1 ~]# timedatectl set-time 2023-02-27
[root@rhcsa-node-1 ~]# timedatectl set-time 17:00:00  
[root@rhcsa-node-1 ~]# timedatectl                    
               Local time: Mon 2023-02-27 17:00:01 GMT
           Universal time: Mon 2023-02-27 17:00:01 UTC
                 RTC time: Mon 2023-02-27 17:00:02    
                Time zone: Europe/London (GMT, +0000) 
System clock synchronized: no                         
              NTP service: n/a                        
          RTC in local TZ: no                         
```
setting timezones
```
[root@rhcsa-node-1 ~]# timedatectl list-timezones
[root@rhcsa-node-1 ~]# timedatectl set-timezone Europe/London 
```
Setup `NTP` sync with `timedatectl`
```
[root@rhcsa-node-1 ~]# timedatectl set-ntp true
[root@rhcsa-node-1 ~]# timedatectl | grep -E '(sync|NTP)'
System clock synchronized: yes                           
              NTP service: active                        
```
If the `System clock` is not set to `yes`, we need to ensure we have `chrony` installed
```
[root@rhcsa-node-1 ~]# dnf install chrony -y 
[root@rhcsa-node-1 ~]# systemctl enable chronyd --now
[root@rhcsa-node-1 ~]# systemctl status chronyd # ensure it's running
```
To see if we are synced, we can run the following:
```
[root@rhcsa-node-1 ~]# chronyc sources -v

  .-- Source mode  '^' = server, '=' = peer, '#' = local clock.
 / .- Source state '*' = current best, '+' = combined, '-' = not combined,
| /             'x' = may be in error, '~' = too variable, '?' = unusable.
||                                                 .- xxxx [ yyyy ] +/- zzzz
||      Reachability register (octal) -.           |  xxxx = adjusted offset,
||      Log2(Polling interval) --.      |          |  yyyy = measured offset,
||                                \     |          |  zzzz = estimated error.
||                                 |    |           \
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^- dot.kkursor.ru                2   6   273     4  +1091us[+1091us] +/-   66ms
^- ntp.copaco.com.py             2   6   377     5  -1813us[-1813us] +/-  188ms
^- www.kapos-net.hu              3   6   375     5  +1797us[+1797us] +/-   65ms
^* ntp3.lwlcom.net               1   6   377     6   -106us[ -172us] +/-   12ms
```
At the moment, we are using upstream NTP timeservers. If we want to use our own, we will need to edit
the `/etc/chrony.conf` file and append `server ntp.example.lab iburst`. Ensure we have the iburst
option, which provides a greater accuracy of time syncing

## firewalls for NTP
If we're unable to reach externally to an NTP server, we need to ensure we have `port 123` or services `ntp`
enabled in firewalld
```
[root@rhcsa-node-1 ~]# firewall-cmd --add-port=123/udp --permanent
success                                                           
[root@rhcsa-node-1 ~]# firewall-cmd --add-service=ntp --permanent 
Warning: ALREADY_ENABLED: ntp                                     
success                                                                   
[root@rhcsa-node-1 ~]# firewall-cmd --reload                      
success                                                           
```

# 5. Install and update software packages from Red Hat Network, a remote repository, or from the local file system

In order to gain access to RedHat Repositories, we must first register our system
```
subscription-manager register              
subscription-manager refresh               
subscription-manager list --available --all
subscription-manager attach --auto         
```
To install packages, we can use the `DNF` package manager
```
[root@rhcsa-node-1 ~]# dnf group list
[root@rhcsa-node-1 ~]# dnf group info <group_name> 
[root@rhcsa-node-1 ~]# dnf group install <group_name>
[root@rhcsa-node-1 ~]# dnf search
[root@rhcsa-node-1 ~]# dnf whatprovides
[root@rhcsa-node-1 ~]# dnf localinstall
[root@rhcsa-node-1 ~]# dnf install
```
We can also use `RPM` in order to install RPM packages
```
[root@rhcsa-node-1 ~]# rpm -i package.rpm # install a package
[root@rhcsa-node-1 ~]# rpm -U package.rpm # upgrade a package
```

# 6. Modify the system bootloader
The bootloader for a Linux system can be stored in two locations, depending on whether it's a `UEFI` or `BIOS` system:
* `UEFI` - `/boot/efi/EFI/redhat/grub.cfg`
* `BIOS` - `/boot/grub2/grub.cfg`

For any changes we want to make to the bootloader, we will always use the `/etc/default/grub` location

After changes have been made, we can re-generate out boot loader with
```
# For UEFI
[root@rhcsa-node-1 ~]# grub2-mkconfig -o /boot/efi/EFI/redhat/grub.cfg
Generating grub configuration file ...                                
Adding boot menu entry for EFI firmware configuration                 
done                                                                  

# For BIOS
[root@rhcsa-node-1 ~]# grub2-mkconfig -o /boot/grub2/grub.cfg
Generating grub configuration file ...                                
```
