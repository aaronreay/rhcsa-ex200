# 1. Find and retrieve container images from a remote registry

Before we continue, we will need to install the following packages:
* `dnf install podman -y`
* `dnf install skopeo -y`

We will also need to login to the podman registry to access images
```
[root@rhcsa-node-1 ~]# podman login registry.redhat.io # redhat developer credentials
Login Succeeded!                                                  
```
If we want to `pull` an image, we can search the repository
```
[root@rhcsa--node-1 ~]# podman search 'Hello'
NAME                                                     DESCRIPTION
docker.io/library/hello-world                            Hello World! (an example of minimal Dockeriz...

[root@rhcsa-node-1 ~]# podman pull docker.io/library/hello-world
Trying to pull docker.io/library/hello-world:latest...
```

# 1. Inspect container images
We can use `Skopeo` to inspect remote container images
```
[root@rhcsa-node-1 ~]# skopeo inspect docker://docker.io/library/hello-world 
{                                                                            
    "Name": "docker.io/library/hello-world",                                 
```

We can use `podman inspect` on *local* images
```
[root@rhcsa-node-1 ~]# podman inspect feb5d9fea6a5 # image ID
[                                                                                  
     {                                                                             
          "Id": "feb5d9fea6a5e9606aa995e879d862b825965ba48de054caab5ef356dc6b3412",
```

# 3. Performing container management using commands such as podman and skopeo

## tagging containers
we can use the `podman tag` to add an additional name to a locally-stored image
```
[root@rhcsa-node-1 ~]# podman tag feb5d9fea6a5 hello-world:latest              
[root@rhcsa-node-1 ~]# podman image list                                       
REPOSITORY                     TAG         IMAGE ID      CREATED        SIZE   
docker.io/library/hello-world  latest      feb5d9fea6a5  17 months ago  19.9 kB
localhost/hello-world          latest      feb5d9fea6a5  17 months ago  19.9 kB # our new tagged local image
```

# 4. Build a container from a ContainerFile
To build an image from a `Container|DockerFile` we can use the `podman build` command, which uses `buildah` scripts in order to build
```
# Create a DockerFIle with the contents
[root@rhcsa-node-1 ~]# cat Dockerfile          
FROM registry.access.redhat.com/rhel-minimal   
ENTRYPOINT "echo "Podman build this container."

# run the podman build command
[root@rhcsa-node-1 ~]# podman build -t podbuilt . # podbuilt will be the name of our container
STEP 1/2: FROM registry.access.redhat.com/rhel-minimal

# run the new container
podman run podbuilt
```

# 5. Perform basic container management such as running, starting, stopping, and listing running containers

To `run` a container
```
[root@rhcsa-node-1 ~]# podman run -d rhel-minimal #  we use '-d' to detatch, and run in the background
c0a5f89e819b9d9b28ccffe3d98cf9f1af0628ae5ccfaf1332c40e20c89e32b0
```
To `list` container status
```
[root@rhcsa-node-1 ~]# podman container ls
CONTAINER ID  IMAGE                           COMMAND               CREATED             STATUS                 PORTS       NAMES
a01b9b29ee22  docker.io/library/caddy:latest  caddy run --confi...  About a minute ago  Up About a minute ago              amazing_matsumoto

podman ps -a # this will show both running and stopped containers
```
To `stop` and `start` a container
```
[root@rhcsa-node-1 ~]# podman stop amazing_matsumoto
amazing_matsumoto
[root@rhcsa-node-1 ~]# podman container ls
CONTAINER ID  IMAGE       COMMAND     CREATED     STATUS      PORTS       NAMES
[root@rhcsa-node-1 ~]# podman start amazing_matsumoto
amazing_matsumoto
[root@rhcsa-node-1 ~]# podman container ls
CONTAINER ID  IMAGE                           COMMAND               CREATED        STATUS             PORTS       NAMES
a01b9b29ee22  docker.io/library/caddy:latest  caddy run --confi...  3 minutes ago  Up 11 seconds ago              amazing_matsumoto
```

# 6. Run a service inside a container
## running commands inside a remote container
we can actually spin up a remote container, run commands, and then delete after exit
```
[root@rhcsa-node-1 ~]# podman run --rm docker.io/library/caddy:latest ls
```
## runnings commands on a local container
```
[root@rhcsa-node-1 ~]# podman run -it --rm docker.io/library/caddy:latest /bin/sh
/srv #                                                                           
```
Above we can see we've spawned a shell on the local container

## run a container with additional arguments
```
[root@rhcsa-node-1 ~]# podman run -d --rm --ip=10.88.0.20 caddy:latest                                                              
040c7c96d3b296a18ad03108938d5b0dc4a584cf85bd47645383af9253ec4ed7                                                                    
[root@rhcsa-node-1 ~]# podman container ls                                                    
CONTAINER ID  IMAGE                           COMMAND               CREATED         STATUS             PORTS       NAMES
a01b9b29ee22  docker.io/library/caddy:latest  caddy run --confi...  25 minutes ago  Up 22 minutes ago              amazing_matsumoto
040c7c96d3b2  docker.io/library/caddy:latest  caddy run --confi...  14 seconds ago  Up 14 seconds ago              mystifying_swartz

[root@rhcsa-node-1 ~]# podman container inspect 040c7c96d3b2 | grep -i 10.88.0.20
               "IPAddress": "10.88.0.20",
                         "IPAddress": "10.88.0.20",
                    "--ip=10.88.0.20",
```

## executing commands on a container
```
[root@rhcsa-node-1 ~]# podman exec -it 040c7c96d3b2 ls
[root@rhcsa-node-1 ~]#                                
```

# 7. Configure a container to start automatically as a systemd service

As we can run podman containers as a non-root user, we can also use `systemd` with the `--user` parameter. For this case, we will use root
First, let's generate a container template
```
[reaya@rhcsa-node-1 ~]$ podman create --name caddy docker.io/library/caddy
Trying to pull docker.io/library/caddy:latest...
Getting image source signatures
Copying blob 90648ba22c19 done  
Copying blob 27e3e797818b done  
Copying blob ef5531b6e74e done  
Copying blob 5cebb4c80b39 done  
Copying config d8464e23f1 done  
Writing manifest to image destination
Storing signatures
d7592931ff97d5f99d7e58c6afb37a4bf50fd126ad2fdc087b97128d81190dc3
[reaya@rhcsa-node-1 ~]$ podman generate systemd --name caddy
# container-caddy.service
# autogenerated by Podman 4.2.0
# Tue Feb 28 17:59:30 GMT 2023

[Unit]
Description=Podman container-caddy.service
Documentation=man:podman-generate-systemd(1)
Wants=network-online.target
After=network-online.target
RequiresMountsFor=/run/user/1000/containers

[Service]
Environment=PODMAN_SYSTEMD_UNIT=%n
Restart=on-failure
TimeoutStopSec=70
ExecStart=/usr/bin/podman start caddy
ExecStop=/usr/bin/podman stop -t 10 caddy
ExecStopPost=/usr/bin/podman stop -t 10 caddy
PIDFile=/run/user/1000/containers/overlay-containers/d7592931ff97d5f99d7e58c6afb37a4bf50fd126ad2fdc087b97128d81190dc3/userdata/conmon.pid
Type=forking

[Install]
WantedBy=default.target
```
We can also run the below command in the directory `$HOME/.config/systemd/user`
```
[reaya@rhcsa-node-1 user]$ podman generate systemd --new --files --name caddy 
/home/reaya/.config/systemd/user/container-caddy.service                      
[reaya@rhcsa-node-1 user]$ systemctl daemon-reload --user                     
[reaya@rhcsa-node-1 user]$ systemctl daemon-reload --user                     
[reaya@rhcsa-node-1 user]$ systemctl --user start container-caddy.service     
[reaya@rhcsa-node-1 user]$ systemctl --user is-active container-caddy.service 
active                                                                        
```
Above, we can see we've generated a systemd file with the flags `--new` and `--files`. This will place the `systemd` configuration in our current location

# 8. Attach persistent storage to a container
## Attach local directory to a container
```
[root@rhcsa-node-1 ~]# podman run --name="log_test" -v /dev/log:/dev/log --rm registry.redhat.io/ubi8/ubi logger "Testing logger"

Trying to pull registry.redhat.io/ubi8/ubi:latest...
Getting image source signatures
Checking if image destination supports signatures
Copying blob b92727ef7443 done  
Copying config 270f760d3d done  
Writing manifest to image destination
Storing signatures
[root@rhcsa-node-1 ~]# journalctl -b | grep 'Testing'
Feb 28 18:13:23 rhcsa-node-1.lab root[10385]: Testing logger
```
Here we have run a new container and attached our `/dev/log` device, and have enacted the `logger` command. This has then shown up on our local OS's
journal as the `/dev/log` drive is shared between the two now.

## Mounting the docker container directory to the local OS
```
[root@rhcsa-node-1 ~]# podman mount a01b9b29ee22
/var/lib/containers/storage/overlay/03b72c447216d880068d37179301c8538284f9dda42b9aa01c438ed2b4aa8428/merged
[root@rhcsa-node-1 ~]# ls -l /var/lib/containers/storage/overlay/03b72c447216d880068d37179301c8538284f9dda42b9aa01c438ed2b4aa8428/merged
total 8
drwxr-xr-x. 2 root root 4096 Feb 10 16:46 bin
drwxr-xr-x. 1 root root   19 Feb 11 07:38 config
drwxr-xr-x. 3 root root   19 Feb 11 07:38 data
drwxr-xr-x. 2 root root    6 Feb 10 16:46 dev
drwxr-xr-x. 1 root root   25 Feb 28 17:27 etc
drwxr-xr-x. 2 root root    6 Feb 10 16:46 home
drwxr-xr-x. 1 root root   17 Feb 10 16:46 lib
drwxr-xr-x. 5 root root   44 Feb 10 16:46 media
drwxr-xr-x. 2 root root    6 Feb 10 16:46 mnt
drwxr-xr-x. 2 root root    6 Feb 10 16:46 opt
dr-xr-xr-x. 2 root root    6 Feb 10 16:46 proc
drwx------. 1 root root   19 Feb 15 19:19 root
drwxr-xr-x. 1 root root   42 Feb 28 17:27 run
drwxr-xr-x. 2 root root 4096 Feb 10 16:46 sbin
drwxr-xr-x. 2 root root    6 Feb 10 16:46 srv
drwxr-xr-x. 2 root root    6 Feb 10 16:46 sys
drwxrwxrwt. 1 root root    6 Feb 15 19:19 tmp
drwxr-xr-x. 1 root root   17 Feb 10 16:46 usr
drwxr-xr-x. 1 root root   19 Feb 10 16:46 var

[root@rhcsa-node-1 ~]# podman unmount a01b9b29ee22
a01b9b29ee22
[root@rhcsa-node-1 ~]# ls -l /var/lib/containers/storage/overlay/03b72c447216d880068d37179301c8538284f9dda42b9aa01c438ed2b4aa8428/merged
ls: cannot access '/var/lib/containers/storage/overlay/03b72c447216d880068d37179301c8538284f9dda42b9aa01c438ed2b4aa8428/merged': No such file or directory
```
When we `mount` a container, we will be presented with the location, which we can then access from our local OS

## Creating persistent volumes
```
[root@rhcsa-node-1 ~]# podman volume create new_volume
new_volume
[root@rhcsa-node-1 ~]# podman inspect new_volume
[
     {
          "Name": "new_volume",
          "Driver": "local",
          "Mountpoint": "/var/lib/containers/storage/volumes/new_volume/_data",
          "CreatedAt": "2023-02-28T18:18:34.995108446Z",
          "Labels": {},
          "Scope": "local",
          "Options": {},
          "MountCount": 0,
          "NeedsCopyUp": true,
          "NeedsChown": true
     }
]
[root@rhcsa-node-1 ~]# podman rm -f caddy
9b05934016466bdb724fbad343d7944057862dfc89a8bf5c0aa31ba2538bdd16
[root@rhcsa-node-1 _data]# podman run -it --name=caddy -v new_volume:/containervolume docker.io/library/caddy /bin/sh
/srv # df -kh
Filesystem                Size      Used Available Use% Mounted on
overlay                  26.4G      2.8G     23.6G  11% /
tmpfs                    64.0M         0     64.0M   0% /dev
/dev/mapper/root--vg-root--lv
                         26.4G      2.8G     23.6G  11% /containervolume
tmpfs                   896.0M     11.4M    884.5M   1% /etc/resolv.conf
shm                      62.5M         0     62.5M   0% /dev/shm
tmpfs                   896.0M     11.4M    884.5M   1% /etc/hostname
tmpfs                   896.0M     11.4M    884.5M   1% /run/.containerenv
tmpfs                   896.0M     11.4M    884.5M   1% /run/secrets
tmpfs                   896.0M     11.4M    884.5M   1% /etc/hosts
tmpfs                   896.0M         0    896.0M   0% /sys/fs/cgroup
tmpfs                   896.0M         0    896.0M   0% /proc/acpi
tmpfs                    64.0M         0     64.0M   0% /proc/kcore
tmpfs                    64.0M         0     64.0M   0% /proc/keys
tmpfs                    64.0M         0     64.0M   0% /proc/timer_list
tmpfs                    64.0M         0     64.0M   0% /proc/sched_debug
tmpfs                   896.0M         0    896.0M   0% /proc/scsi
tmpfs                   896.0M         0    896.0M   0% /sys/firmware
tmpfs                   896.0M         0    896.0M   0% /sys/fs/selinux
tmpfs                   896.0M         0    896.0M   0% /sys/dev/block
/srv # cd /containervolume/
/containervolume # touch test.txt
/containervolume # exit
[root@rhcsa-node-1 _data]# ls -ltr
total 0
-rw-r--r--. 1 root root 0 Feb 28 18:22 test.txt
```
If we have issues with trying to run the new container, we can remove the old one with `podman rm -f caddy`

We also have the option to use this new persistent volume on a `new` container
```
[root@rhcsa-node-1 _data]# podman run -it --name=caddy2 -v new_volume:/containervolume docker.io/library/caddy /bin/sh
/srv # cd /containervolume/
/containervolume # ls -ltr
total 0
-rw-r--r--    1 root     root             0 Feb 28 18:22 test.txt
```
