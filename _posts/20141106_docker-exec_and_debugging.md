---
layout:     post
title:      Docker Exec: Sane Docker Debugging without SSH
date:       2014-11-06 09:00:00
summary:    Docker exec enables docker debugging during development or production without SSH or other hacks.
categories: infrastructure docker debugging
---

The new [Docker Exec](https://docs.docker.com/reference/commandline/cli/#exec) command (available in Docker version 1.3+) allows users to run any command in any running container. These commands can either run in the background, or in the foreground. The canonical example given is to run bash in a container to do debugging. For example, this week I was working on a container for a client that had a run command like this:

{% highlight bash %}
docker run -p 127.0.0.1:2200:22 -p 80:80 \
  --name=my-railsapp --workdir /home/swuser \
  -e HOME=swuser mydevelopment/my-railsapp:staging \
  /etc/runit/2 syslog
{% endhighlight %}

As you'll notice, I have [runit](http://smarden.org/runit/) running within this container to start up the multiple services that this application needs. And I've included a tiny ssh server (dropbear in this example) to allow administrators and users to have an easy way to inspect the container as needed during runtime.

The new `docker exec` command would allow me to run the container like this:

{% highlight bash %}
docker run -p 80:80 \
  --name=my-railsapp --workdir /home/swuser \
  -e HOME=swuser mydevelopment/my-railsapp:staging \
  /etc/runit/2 syslog
{% endhighlight %}

Notice that I no longer have the need to open up port 22 even locally for ssh. +1 for security. 
Removing SSH also significantly reduces the complexity of having to manage keys and/or passwords. +1 for peace of mind.

##### So how to debug in this brave new SSH-less world?

First, a quick helper alias to allow us to quickly find the id of a running container:

{% highlight bash %}
alias docker-id='docker inspect --format="{{ .Id }}"'
{% endhighlight %}

And then when I want to debug the application, I can simply run:

{% highlight bash %}
docker exec -it $(docker-id my-webapp) /bin/bash
{% endhighlight %}

Or, to quickly debug the last-launched container:

{% highlight bash %}
docker exec -it `docker ps -q -l` /bin/bash
{% endhighlight %}

And I'm in my container, able to debug as expected. 

I'm planning to do additional testing to see if docker exec opens up other possibilities for streamlining our workflow and reducing the complexity of our running containers.
