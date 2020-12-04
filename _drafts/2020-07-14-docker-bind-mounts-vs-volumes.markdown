---
layout: post
title:  "Docker Bind Mounts vs Volumes"
date:   2020-07-14
---

While volumes are the preferred mechanism(https://docs.docker.com/storage/volumes/) for persisting data in Docker, there is a second option: bind mounts(https://docs.docker.com/storage/bind-mounts/).

We saw in the previous post(LINK) that we can access the data in a volume simply by referencing the volume's name. This is possible because when you create a volume, a new directory is created within a Docker-managed storage directory on the host machine.

Unlike volumes, bind mounts exist _outside_ of the Docker storage area. In fact, they can exist anywhere in the host's file system. This means that we don't get to simply reference our bind mounts by name to access their data. Instead, we have to use the bind mount's full or relative path on the host machine.

**Use diagram from Docker docs**

Another drawback to using bind mounts is that you cannot manage them directly with Docker CLI commands, as you can with volumes. Docker provides commands to create, inspect, remove, and list volumes (https://docs.docker.com/storage/volumes/#create-and-manage-volumes), where as the only support Docker provides with bind mounts is the ability to mount them to a running container (https://docs.docker.com/storage/bind-mounts/#start-a-container-with-a-bind-mount).


So why have two different ways of doing the same thing, especially if volumes seem like the more convenient approach to persisting data?

Well, they don't do exactly the same thing.

###### FIX this beased on bind mounts for non-empty container directories (https://docs.docker.com/storage/bind-mounts/#mount-into-a-non-empty-directory-on-the-container): #######
To understand this difference, I find it helpful to think of the direction that you want the data to flow between the container and the host machine's directory. When you create a volume, it is empty, and there is no way to add data to it before mounting it to a container—the volume can only be updated from within a running container. A bind mount, on the other hand, can be any directory you like, including one with files already in it. When you mount this directory to a container, the container immediately has access to those files. Within this context, bind mounts can perform like volumes do, but volumes cannot perform like bind mounts do. Both volumes and bind mounts can be used to persist data and, if this is your objective, you should use volumes for their greater convenience. If, however, you want to inject additional data into a brand new container, you will have to use bind mounts.

As a side note, bind mounts can be read-only (https://docs.docker.com/storage/bind-mounts/#use-a-read-only-bind-mount), which may be preferrable if you're only injecting data into a container. Volumes can also be read-only (https://docs.docker.com/storage/volumes/#use-a-read-only-volume); however, this setting only makes sense if at least one container is writing to the volume (otherwise, remember, the volume will just remain empty). Volumes, like bind mounts, can be mounted to multiple containers simultaneously, so a volume could be written to by one or more containers, and only read from by others.

I find comparing the Docker commands for mounting a directory as a bind mount vs mounting a volume to be particularly helpful in understanding the core difference between the two.

We have a `/data` directory that has some files in it:

![Data directory contains three test files, Data1.txt, Data2.txt, and Data3.txt](/assets/images/data-directory-contents.png)

Here's how to start a container with a bind mount, mounting our `/data` directory.

{% highlight bash %}
docker run -d \
-it \
--name bindmounttest \
--mount type=bind,source="$(pwd)"/data,target=/shareddata \
node:12-alpine
{% endhighlight %}

The bind mount command tells Docker that the source (the dirctory on our host machine we want our container to access) is a directory called `/data` (The `$(pwd)` sub-command expands to the current working directory on Linux or macOS hosts), and the target (the path within the container where the data from the source will live) is `/shareddata`. 

If we SSH into our `bindmounttest` container, we see those same files within `/data` in the `/shareddata` directory.



And here's how to start a container with a volume called `myvol` (if `myvol` doesn't already exist, Docker will create it for you now).

{% highlight bash %}
docker run -d \
-it \
--name volumetest \
--mount type=volume,source=myvol,target=/app \
node:12-alpine
{% endhighlight %}

These commands start named containers using the `node:12-alpine` image, but for the sake of this lesson, focus on the lines beginning with `--mount `.



This starts a container called `bindmount` using the `node:12-alpine` image, but for the sake of this lesson, focus on the line `--mount type=bind,source="$(pwd)"/data,target=/shareddata`. Here, we're specifying the type of mount as a `bind` (for a bind mount, as opposed to a volume), we saying the source (the dirctory on our host machine we want our container to access) is a directory called `/data` (The `$(pwd)` sub-command expands to the current working directory on Linux or macOS hosts), and the target (the path within the container where the data from the source will live)

Let's create a simple bind mount.

Let's say you have a directory on your host machine, `/data`, that you want to make available to a new container. The `/data` directory contains the following files:

- screenshot `ls /data`

We can then start a container using the `--mount` flag (https://docs.docker.com/storage/bind-mounts/#start-a-container-with-a-bind-mount).

In the previous post, we mounted a volume using the `-v` (or `--volume`) flag, which can also be used with bind mounts; however, we're going to use the `-m` (or `--mount`) flag this time ... better to see differences, also reccommended by docker (?), they're similar but not the same (another post on the differences? just link to the section on differences in Docker docs?)






TODO:
- find it useful to think of the direction that I want the data to move
- volumes are for persisting data, and while bind mounts can do that, they tend to be for injecting additional data into out containers. Volumes are just more convenient, so if you want to persist data, just use those
  - when using volume, you only give it the name of the volume (source) and the path of the *target* directory within the container. So you can think of the data as flowing out of the container and into the volume
    - you can't use volumes to inject data into a container—when you create a volume, it is empty, and the only way to add data to it is by mounting it to a running container
  - when using a bind mount, you give it the source directory path (on the host machine) and the path of the target directory within the container
     - you can inject data into a container with bind mounts, because you can use any directory on your host machine as the source


Further evidence for different "directions" of data flow:
  - If you bind-mount into a non-empty directory on the container, the directory’s existing contents are obscured by the bind mount. This can be beneficial, such as when you want to test a new version of your application without building a new image. However, it can also be surprising and this behavior differs from that of docker volumes. (https://docs.docker.com/storage/bind-mounts/#mount-into-a-non-empty-directory-on-the-container)
  - If you start a container which creates a new volume, as above, and the container has files or directories in the directory to be mounted (such as /app/ above), the directory’s contents are copied into the volume. The container then mounts and uses the volume, and other containers which use the volume also have access to the pre-populated content. (https://docs.docker.com/storage/volumes/#populate-a-volume-using-a-container)

  ^ I kind of want a diagram with arrows for this (bind mount ==> container's directory  /  volume <== container's directory)


  In fact, some bind mounts can be read-only (https://docs.docker.com/storage/bind-mounts/#use-a-read-only-bind-mount)

  Volumes can also be read-only (https://docs.docker.com/storage/volumes/#use-a-read-only-volume), but it would only make sense to do this if at least one other container was writing to the volume, otherwise the volume would remain empty (multiple containers can mount the same volume, and it can be mounted read-write for some of them and read-only for others, at the same time)

Create examples like the ones here: https://4sysops.com/archives/introduction-to-docker-bind-mounts-and-volumes/#:~:text=The%20main%20difference%20a%20bind,volumes%2C%20similar%20to%20bind%20mounts

Found the difference between the --mount commands very helpful

Find starting with bind mounts, then looking at volumes to be the best way of understanding






