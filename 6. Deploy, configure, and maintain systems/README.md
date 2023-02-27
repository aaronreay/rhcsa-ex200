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
