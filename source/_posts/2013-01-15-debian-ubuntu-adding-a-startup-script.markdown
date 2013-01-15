---
layout: post
title: "Debian & Ubuntu: Adding a StartUp Script"
date: 2013-01-15 07:39
comments: false
categories: Notes Linux
---

I recently had the need to run some code with each server restart.  [update-rc.d](http://manpages.ubuntu.com/manpages/precise/en/man8/update-rc.d.8.html) provides a great and easy way execute your own code on server startup (in my case, Ubuntu 12.04).

<!--more-->

As root, create your script:  

```bash spaghetti.sh
#!/bin/sh
mkdir /tmp/spaghetti && touch /tmp/spaghetti/meatballs.txt  
```

Make the script executable:  

```bash
root@cameron ~ % chmod +x spaghetti.sh
```

Add your script to `/etc/init.d`:  

```bash
root@cameron ~ % mv spaghetti.sh /etc/init.d
```

Run `update-rc.d` on the new script:  

```bash
root@cameron ~ % update-rc.d spaghetti.sh defaults
```

So, what is the `defaults` argument?

{% blockquote Ubuntu Manuals, Installing Init Script Links, http://manpages.ubuntu.com/manpages/precise/en/man8/update-rc.d.8.html %}
 If  defaults  is  used  then  update-rc.d  will make links to start the service in runlevels 2345 and to stop the service in runlevels 016.
{% endblockquote %}

Here's a great article describing runlevels in exhausting detail:

{% blockquote linux.com, An introduction to services, runlevels, and rc.d scripts, https://www.linux.com/news/enterprise/systems-management/8116 %}
And what is a runlevel? You might assume that this refers to different levels that the system goes through during a boot up. Instead, think of the runlevel as the point at which the system is entered. Runlevel 1 is the most basic configuration (simple single user access using an text interface), while runlevel 5 is the most advanced (multi-user, networking, and a GUI front end). Runlevels 0 and 6 are used for halting and rebooting the system.
{% endblockquote %}
