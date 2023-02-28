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

