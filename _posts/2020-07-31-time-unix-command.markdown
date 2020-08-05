---
layout: post
title:  "The Unix Time Command"
date:   2020-07-31
---

Today I learned about the `time` command available on Unix and Unix-like systems, which tells you how long a given command takes to run.

We wanted to see how long certain steps in a CI build were taking, but the CI tool did not provide any of this timing information itself. To quickly solve this problem, we prepended the commands in the CI config file with `time`, like so:

{% highlight bash %}
time gem install bundler -v 1.7.13
{% endhighlight %}

Once Bundler had been installed, the output was printed, along with a summary of how long it took to complete.

{% highlight bash %}
Successfully installed bundler-1.7.13
1 gem installed

real  1m17.679s
user  1m15.359s
sys 0m1.655s
{% endhighlight %}

The "real" time refers to the elapsed wall-clock time, so it's as if you had used a stop watch. The "user" and "sys" times are combined to give the total CPU time. In some instances, we saw the total CPU time was higher than the elapsed wall-clock time—this happens when tasks are being run in parallel, so it reflects the cummulative time spent on the command by the CPU.

