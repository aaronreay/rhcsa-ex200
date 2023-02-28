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
This is *_VERY IMPORTANT_* for the exam. If we cannot do this, we cannot proceed with the exam

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

