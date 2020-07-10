---
layout: post
title:  "Docker Volumes"
date:   2020-07-08
---

By default, all files created inside a Docker container are stored in a writable container layer. This layer is like the container's "scratch space", so any changes won't be seen by other containers, even if they're using the same image. Also, if you stop and restart the container, any file changes you previously made will disappear.

Let's say you have a todo app that uses a SQLite Database, so you have a relational database in the form of a single file. You run your todo app inside a Docker container, open it up in your browser, and add some new list items to your todo list. You inspect the SQLite file and see each list item being written to it. Cool. You refresh the page, the items are still there. Great. But then you restart the Docker container, and now your list items are gone and your SQLite file is empty again.

The changes to the SQLite file were happening within the "scratch space", which only lives within _that_ container instance. So how do we ensure that the files we write to don't disappear when we stop the container?

One solution is to use [volumes](https://docs.docker.com/storage/volumes/).

Think of volumes as buckets of data that live on the Docker host. (Quick sidenote: if you're using Docker Desktop, the Docker host is actually a small VM that's running on your machine.) Volumes exist independently from containers.

Let's persist our SQLite file with a volume.

First, create the volume:

{% highlight bash %}
docker volume create todo-db
{% endhighlight %}

We've given our volume the name, `todo-db`. You do not have to provide a name, but anonymous volumes are assigned a unique, random name. As long as you know the volume's name, you can rely on Docker to maintain the volume's physical location on the disk and always return the correct data.

We've created a volume, but at this point it doesn't really do anything. The next step is to _mount_ the volume to the directory we want to capture. We mount the volume to this directory when we start our container, using this formula.

{% highlight bash %}
docker run -dp <host-port>:<container-port> -v <volume>:<directory> <image>
{% endhighlight %}

The SQLite file we want to persist lives at `/etc/todos/todo.db`, so we're going to tell our volume to track the files inside the `/etc/todos` directory. If the image of our app was called `todo-app`, and we wanted to map the host's port `3000` to port `3000` on the container, our command would look like this.

{% highlight bash %}
docker run -dp 3000:3000 -v todo-db:/etc/todos todo-app
{% endhighlight %}

Now our application data will survive restarting the container!
