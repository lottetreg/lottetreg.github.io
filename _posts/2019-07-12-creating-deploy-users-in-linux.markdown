---
layout: post
title:  "Creating Deploy Users in Linux"
date:   2019-07-12
---

This week I set up a server on DigitalOcean to host a new project. The server uses Ubuntu (a Linux distribution) and I can SSH into it from my local machine like this:

{% highlight bash %}
ssh root@[SERVER IP ADDRESS]
{% endhighlight %}

The `root` portion of this command denotes the user we want to log in as; because this is a brand new server, the root user is the only user we have. Different users on an operating system can have varying priviliges and levels of access. Root, the most priviliged user on the Linux system, has access to every command and file—for this reason, it is also known as the _superuser_ or _administrator_.

Logging in as the root user gives you a dangerous level of power, and it is therefore reccommended that you create a limited user account to use at all times. If you want to conduct administrative tasks, you can use the `sudo` command (short for "superuser do") to temporarily elevate your limited user's privileges. Because I want to deploy to my DigitalOcean server from multiple machines, I'm going to create a new `deploy` user.

{% highlight bash %}
adduser deploy
{% endhighlight %}

I'm prompted for a password, which I enter (and some other identifying information, which I skip). I store this password, which I'll need to run `sudo` commands, in my password manager.

To give our deploy user access to the `sudo` privileges, we need to explicitly add it to the `sudo` group:

{% highlight bash %}
usermod -aG sudo deploy
{% endhighlight %}

`usermod` is the command to modify user accounts and privileges, and `-aG` instructs it to add the specified user to the specified group.

Now that we have our new user with root privileges, let's try connecting to our server as `deploy` instead of `root`.

{% highlight bash %}
ssh deploy@[SERVER IP ADDRESS]
{% endhighlight %}

The previous command is correct, but it returns the following error: `Permission denied (publickey).`

This error occurs because each user maintains its own `~/.ssh` directory, which needs to contain an `authorized_keys` file of public keys to accept connections from. If we log in to our server as root again and view `/home`, we can see the all of the users' home directories.

{% highlight bash %}
root@my-server-name:~# ls /home
deploy
{% endhighlight %}

We only see one  user, `/deploy`, because `/root` is not accessible from here. In order to SSH into our server as the deploy user, we need to set up a `/home/deploy/.ssh/authorized_keys` file.

Let's first switch over to our new deploy user with the following command. This will automatically set the deploy user as the owner of any files we create.

{% highlight bash %}
su deploy
{% endhighlight %}

Now, let's add the `authorized_keys` file to our deploy user's `~/.ssh` directory:

{% highlight bash %}
touch /home/deploy/.ssh/authorized_keys
{% endhighlight %}

Finally, let's pipe the contents of our root user's `authorized_keys` file into our new file:

{% highlight bash %}
sudo cat /root/.ssh/authorized_keys > /home/deploy/.ssh/authorized_keys
{% endhighlight %}

You'll need to use the `sudo` command to do this because we don't have access to root's files as the lowly deploy user.

Now, when I try again to connect as the deploy user from my local machine, it works!

Because I also want to deploy to my server from a continuous integration service, I need to generate another set of SSH keys. Once the continous integration service has the private key, all I need to do is add the corresponding public key to the `/home/deploy/.ssh/authorized_keys` file, and I'll be able to connect to my server from there, as well!

From now on, anything that needs access to my server for deployment will connect via the deploy user, which password-protects administrative actions, and the all-powerful root user will be hidden away.
