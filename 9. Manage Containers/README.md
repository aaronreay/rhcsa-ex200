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
