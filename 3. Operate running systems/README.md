# 1. Boot, reboot, and shut down a system normally
For `RHEL 7+` we use the `systemd`. We can confirm this by running `ps 1`
```
[root@rhcsa-node-1 ~]# ps 1
    PID TTY      STAT   TIME COMMAND
      1 ?        Ss     0:03 /usr/lib/systemd/systemd --switched-root --system --deserialize 17
```
As we're using `systemd`:
```
systemctl reboot
systemctl reboot --force
systemctl reboot --force --force

systemctl poweroff
systemctl poweroff --force
systemctl poweroff --force --force
```
We can also schedule our reboots and shutdowns
```
shutdown 02:00 # shutdown at 2:00am
shutdown +15 # shutdown in 15 minutes

shutdown -r 02:00 # reboot at 2:00am
shutdown -r +15 # reboot in 15 minutes
```

# 2. Boot systems into different targets manually
To view our potential targets, we can run the below
```
[root@rhcsa-node-1 ~]# ls -l /usr/lib/systemd/system/runlevel*.target
lrwxrwxrwx. 1 root root 15 Sep 27 08:35 /usr/lib/systemd/system/runlevel0.target -> poweroff.target
lrwxrwxrwx. 1 root root 13 Sep 27 08:35 /usr/lib/systemd/system/runlevel1.target -> rescue.target
lrwxrwxrwx. 1 root root 17 Sep 27 08:35 /usr/lib/systemd/system/runlevel2.target -> multi-user.target
lrwxrwxrwx. 1 root root 17 Sep 27 08:35 /usr/lib/systemd/system/runlevel3.target -> multi-user.target
lrwxrwxrwx. 1 root root 17 Sep 27 08:35 /usr/lib/systemd/system/runlevel4.target -> multi-user.target
lrwxrwxrwx. 1 root root 16 Sep 27 08:35 /usr/lib/systemd/system/runlevel5.target -> graphical.target
lrwxrwxrwx. 1 root root 13 Sep 27 08:35 /usr/lib/systemd/system/runlevel6.target -> reboot.target
```

# 3. Interrupt the boot process in order to gain access to a system
This is __VERY IMPORTANT__ for the exam. If we cannot do this, we cannot proceed with the exam

We will need to boot up our linux node, and at the `GRUB` prompt, hit `e` to interrupt the process

On the line starting with `linux`, and the end, we need to append `rd.break`. This will bring us to the `initrd` environment

From here, we will not have access to our usual binaries, as the OS has not loaded yet. To fix this
```
mount -o rw,remount /sysroot
chroot /sysroot
passwd
touch /.autorelabel # it's important to relabel the entire filesystem, as selinux does not know the context as we've edited files outside of the OS
exit
exit
```

# 4. Identify CPU/memory intensive processes and kill processes

`top` - we can use `top` in order to display a list of running processes on the system. Here we can see the current system resource usage

`kill -l` - here we can see all of the possible kill commands available to us
```
[root@rhcsa-node-1 ~]# kill -l                                                 
 1) SIGHUP       2) SIGINT       3) SIGQUIT      4) SIGILL       5) SIGTRAP    
 6) SIGABRT      7) SIGBUS       8) SIGFPE       9) SIGKILL     10) SIGUSR1    
11) SIGSEGV     12) SIGUSR2     13) SIGPIPE     14) SIGALRM     15) SIGTERM    
16) SIGSTKFLT   17) SIGCHLD     18) SIGCONT     19) SIGSTOP     20) SIGTSTP    
21) SIGTTIN     22) SIGTTOU     23) SIGURG      24) SIGXCPU     25) SIGXFSZ    
26) SIGVTALRM   27) SIGPROF     28) SIGWINCH    29) SIGIO       30) SIGPWR     
31) SIGSYS      34) SIGRTMIN    35) SIGRTMIN+1  36) SIGRTMIN+2  37) SIGRTMIN+3 
38) SIGRTMIN+4  39) SIGRTMIN+5  40) SIGRTMIN+6  41) SIGRTMIN+7  42) SIGRTMIN+8 
43) SIGRTMIN+9  44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13
48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-13 52) SIGRTMAX-12
53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9  56) SIGRTMAX-8  57) SIGRTMAX-7 
58) SIGRTMAX-6  59) SIGRTMAX-5  60) SIGRTMAX-4  61) SIGRTMAX-3  62) SIGRTMAX-2 
63) SIGRTMAX-1  64) SIGRTMAX                                                   
```
For a process ID of 800, we could run `kill 800` in order to kill this process

We can also use `pkill` `<process_name>` if we want to kill via the process name, rather than the PID of the process

We can also list processes by a specific user
```
[root@rhcsa-node-1 ~]# pgrep -l -u user1
1660 bash       
[root@rhcsa-node-1 ~]# pstree -u user1
bash                                  
                        
[root@rhcsa-node-1 ~]# pkill -u user1   
[root@rhcsa-node-1 ~]# pgrep -l -u user1
[root@rhcsa-node-1 ~]#                  
```

# 5. Adjust process scheduling
`chrt` - manipulate the real-time attributes of a process

* `-f` - set schedule to `SCHED_FIFO`
* `-o` - set schedule to `SCHED_OTHER`
* -`r` - set schedule to `SCHED_RR`

if we want to see the max value range for each option, we can use `chrt --max`

We have another utility, `nice/renice` - this allows us to change the levels of `SCHED_OTHER`

Let's have a look at the following
```
[root@rhcsa-node-1 ~]# ps -l                                              
F S   UID     PID    PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD 
4 S     0    1544     911  0  80   0 -  6562 -      ttyS0    00:00:00 bash
0 R     0    1698    1544  0  80   0 - 11377 -      ttyS0    00:00:00 ps  
```
To set the priority of a process, we can run `nice -n 10 top`. This will set the nice level of top to 10

We can view this in the top output

![nice](nice.png)

to increase the priority of a process, the syntax is as follows:

`nice -n <nice_value> command`

The higher the value, the lower the priority. So `-` is good, `+` is bad (in a sense)

We also have the `renice` command. This allows us to change the process of an existing process

`renice -n <nice_value> -p/--pid <process_id` - change priority of existing process

`renice -n <nice_value> -g <group_id>` - change priority of all processes of a specific group

`renice -n <nice_value> -u <name>|<id>` - change priority of all programs of a specific user

# 6. Manage tuning profiles
`tuned` is the package responsible for tuning profiles. It is managed with `tuned-adm`. We can also get additional profiles with `dnf search tuned-profiles`
```
[root@rhcsa-node-1 ~]# tuned-adm recommend
virtual-guest
[root@rhcsa-node-1 ~]# tuned-adm active
Current active profile: virtual-guest
[root@rhcsa-node-1 ~]# tuned-adm list
Available profiles:
- accelerator-performance     - Throughput performance based tuning with disabled higher latency STOP states
- balanced                    - General non-specialized tuned profile
- desktop                     - Optimize for the desktop use-case
- hpc-compute                 - Optimize for HPC compute workloads
- intel-sst                   - Configure for Intel Speed Select Base Frequency
- latency-performance         - Optimize for deterministic performance at the cost of increased power consumption
- network-latency             - Optimize for deterministic performance at the cost of increased power consumption, focused on low latency network performance
- network-throughput          - Optimize for streaming network throughput, generally only necessary on older CPUs or 40G+ networks
- optimize-serial-console     - Optimize for serial console use.
- powersave                   - Optimize for low power consumption
- throughput-performance      - Broadly applicable tuning that provides excellent performance across a variety of common server workloads
- virtual-guest               - Optimize for running inside a virtual guest
- virtual-host                - Optimize for running KVM guests
Current active profile: virtual-guest
```
* to change our current profile we can run - `tuned-adm profile <profile_name>`
* to verify everything is working correctly - `tuned-adm verify` 

# 7. Locate and interpret system log files and journals
`rsyslog` is our `syslog daemon` of choice, for persistent storage
It will store all logs in directories under `/var/log/*`
* `/var/log/messages` - contents of boot and kernel messages
* `/var/log/secure` - contents of users and their activities

with `RHEL 7+` we also have a new utility, `journald/journalctl`
* `journalctl` - show all event messages, similar to `/var/log/messages`
* `journalctl -k` - show all kernel messages, similar to `dmesg`
* `journalctl -p crit` - show all journal entires based on critical priority
* `journalctl _UID=`user_uid` - display logs from a certain user/uid
* `journalctl --since today` - display all logs from today
* `journalctl -u sshd` - display all logs from a particular servce, in this case, `sshd`

However, `journald` does not provide persistent storage by default, unlike `rsyslog`. We will discuss this more in depth below

# 8. Preserve system journals
Before we persist journals, we will need to create a directory for which these will be stored
```
mkdir /var/log/journal

# Then we need to set journald to persist logging
# in /etc/journald.conf, set Storage from `auto` to `persistent`
reboot

# upon logging back on, we should see data in the /var/log/journal directory
[root@rhcsa-node-1 ~]# ls -l /var/log/journal/                                     
total 0                                                                            
drwxr-sr-x+ 2 root systemd-journal 53 Mar  1 17:46 c2687f51f97242b4ad7ee0bfbe57b450
```

# 9. Start, top and check status of network services
* `systemctl start sshd.service` - start the `ssh` daemon
* `systemctl status sshd.service` - check the status of the `ssh` daemon

If a serivce is `masked`, it will redirect to `/dev/null`. To `unmask` a server, we can run the following:
* `systemctl umask sshd.service`

If we want to permanently disable a service, simply `mask` the service
* `systemctl mask sshd.service`

* `systemctl enable sshd.service --now` - this will both enable and start the service on one line
* `systemctl is-enabled sshd.service` - check if the daemon is enabled

# 10. Securely transfer files between files
## scp
* `scp file1 reaya:rhcsa-node-2:/tmp/` - will copy `file` to `rhcsa-node-2`, as `reaya` user, and store in the `/tmp` directory

by default, `scp` uses `port 22`; but what if we want to use another?
* `scp -P 2525 file` reaya@rhcsa-node-2:/tmp` - the `-P` flag allows us to change the port

we can also send a file using `ssh keys`
* `scp -i ~/.ssh/id_rsa file1 reaya@rhcsa-node-2:/tmp` - this will use ssh keys, assuming the receiving host has a copy of the `.pub` key

## sftp
`sftp` is an interactive secure transfer tool
```
touch ajr
sftp reaya@rhcsa-node-2
cd /tmp
put ajr
```
With the code above, we create a simple file, then create a interactive session on rhcsa-node-2 with sftp. We change into the `/tmp` directory, and use
the `put` command to `upload` our file onto this node. sftp works from the current directory it was spawned from, hence it could see the file

As with `scp`, we can also change the port and use an ssh key:
* `sftp -o Port=2525 -o IdentitiyFile=~/.ssh/id_rsa reaya@rhcsa-node-2:/tmp`

We can also upload and download directories:
* `put -r directory`
* `get -r direct_from_remote`
