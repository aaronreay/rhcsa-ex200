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
