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
