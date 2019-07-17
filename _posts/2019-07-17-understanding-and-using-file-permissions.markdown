---
layout: post
title:  "Understanding and Using File Permissions"
date:   2019-07-17
---

I recently set up the ability to deploy to my remote server via the `git push` command, but it failed on my first attempt with the error, `remote unpack failed: unable to create temporary object directory`. As the error message suggests, `git push` was attempting to create a directory on my remote server, but my user did not have permission to do this. After researching file permissions and how to modify them, I was able to solve the problem.

Everything in Linux and Unix is a file (including directories), and every file on a system has permissions that allow certain users to view, edit, or execute them. The root user is able to access any file on the system.

To view the permissions on a file or directory, use `ls -l filename`:

{% highlight bash %}
user@host:~$ ls -l /etc/hosts
-rw-r--r-- 1 root root 617 Jul 10 18:43 /etc/hosts
{% endhighlight %}

Let's break down this result into fields.

| User Permissions | Owner | Group | Size in Bytes | Date of Last Modification | File Name
|--------------|--------|--------|-------|----------------|--------------|
| `-rw-r--r--` | `root` | `root` | `617` | `Jul 10 18:43` | `/etc/hosts` |

# User Permissions

The very first character in `-rw-r--r--` represents the special permissions on `/etc/hosts`. Here, we see that the special permissions are empty (represented by `-`), which tells us that `/etc/hosts` is a normal file. If we were inspecting a directory or a symlink instead, we would see a `d` or an `l`, respectively.

The remaining `rw-r--r--` tells us which permissions are available for various users. The first three bits represent the owner's permissions, the following three are the permissions of the group, and the final three are those of all other users.

| User | Permissions | Translation
|-------|--------|---------|
| owner | `rw-` | read and write only |
| group | `r--` | read only |
| other | `r--` | read only |

`/etc/hosts` can be read from by any user, but only the owner can edit (write) the file. Some files can be executed and have the "execute" permission represented with an `x`. For example, a file that could be read from, written to, and executed by only the owner would have the permissions `-rwx------`.

Note that these permissions are not inheritable—e.g. the owner of the file is not affected by the permissions set for the group or for all other users.

These permissions behave differently if the file in question is a directory. Being able to read from a directory allows the user to view the directory's contents (such as using the `ls` command), while being able to write to it allows the user to create and delete files in the directory. A word of caution: even if a user does not have write access to a file itself, they could still delete it if they have write access to the file's directory. Finally, having execution permission allows a user to change into the directory (i.e. using the `cd` command).

# Owner

Unless otherwise specified during `touch`, the owner of the file is the user who created it. The owner of a file can be changed with the `chown` command.

{% highlight bash %}
user@host:~# ls -l demo.txt
-rw-r--r-- 1 root root 0 Jul 17 16:13 demo.txt
user@host:~# chown deploy demo.txt
user@host:~# ls -l demo.txt
-rw-r--r-- 1 deploy root 0 Jul 17 16:13 demo.txt
{% endhighlight %}

# Group

The group of a file can also be changed with the `chown` command, which will accept the owner and the group separated by a colon, like `[OWNER]:[GROUP]`. If the colon is present but the owner is missing, `chown` will still change the group accordingly.

{% highlight bash %}
user@host:~# ls -l demo.txt
-rw-r--r-- 1 deploy root 0 Jul 17 16:13 demo.txt
user@host:~# chown :deploy demo.txt
user@host:~# ls -l demo.txt
-rw-r--r-- 1 deploy deploy 0 Jul 17 16:13 demo.txt
{% endhighlight %}

# Changing Permissions

We change the permissions on a file with the `chmod` command, using either numbers or letters. I find the letters easier to understand, so let's use them.

The usage for the `chmod` command is: `chmod {users}{action}{permissions} filename`, and here are all of the available options.

| User Options | Definition |
|---------|------------|
|`u` |owner|
|`g` |group|
|`o` |other|
|`a` |all users (same as `ugo`)|

| Action Options | Definition |
|---------|------------|
|`+` |add permission|
|`-` |remove permission|
|`=` |set permission|

| Permission Options | Definition |
|---------|------------|
|`r` |read|
|`w` |write|
|`x` |execute|

Let's change the permissions of `demo.txt` so the owner can execute it.

{% highlight bash %}
user@host:~# ls -l demo.txt
-rw-r--r-- 1 deploy deploy 0 Jul 17 16:13 demo.txt
user@host:~# chmod u+x demo.txt
user@host:~# ls -l demo.txt
-rwxr--r-- 1 deploy deploy 0 Jul 17 16:13 demo.txt
{% endhighlight %}

You can change permissions for multiple users at once by simply listing them (e.g. `ug+w`), just as you can add or remove multiple permissions at the same time (e.g. `o-wx`).

I hope you now have a better understanding of file permissions!
