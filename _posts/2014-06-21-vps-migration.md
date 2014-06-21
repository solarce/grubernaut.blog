---
layout: post
title: "Setting up a Continuous Delivery Blog"
modified: 2014-06-21 14:29:10 -0400
tags: [cloud, continuous delivery, continuous integration, blog]
image: abstract-4.jpg
comments: true
share: true
---
<figure>
  <img src="/images/vps/digitalocean.jpeg">
  <figcaption>Digital Ocean - https://twitter.com/bryanl/status/479606602186895360</figcaption>
</figure>

### Background
Growing up I never really had the capital to invest in a home setup, nor were my parents ever going to let me near the networking equipment after capturing packets of web traffic from my parents. Getting my own house for the first time last year and purchasing my own networking equipment opened up a lot of doors for me. I was now able to finally publish something on the public internet that I owned. The only problem with this arose with the financial costs of running bare metal servers at home, and the overhead cost I would need to begin such a project. 

I was talking to Brandon Burton ([@solarce](https://twitter.com/solarce)) recently and he told me stories of his first stack that he hosted himself. Remembering that I still hadn't achieved that goal yet, I set out to remedy this problem as quickly as I could. 

Owning bare metal servers would still be a costly venture for me today. I would have to pay for the power consumption, the networking equipment to ensure security, and the increased bandwidth of my internet connection. Thus, configuring a VPS would be my best option. 

I chose Digital Ocean purely out of simplicity. I could have gone the Amazon AWS route, and still may in the future, but I wanted to start this project ASAP, and was itching to get tinkering with setup. This is my bare and basic write-up of how I migrated my Jekyll blog from a GitHub Pages site to a cloud hosted blog with Digital Ocean. 

## Migrating my personal blog to a Digital Ocean Droplet
First things first, I needed to actually create a [Digital Ocean](https://www.digitalocean.com) Droplet. Creating a droplet on [Digital Ocean](https://www.digitalocean.com) was actually the easiest part of this whole process. I was entirely pleased with the whole setup of my account, the ease of integrating 2-Factor Authentication, and setting up billing via PayPal for my droplet. 

### Setting up a Digital Ocean Droplet
Once I had a few coins in my Digital Ocean account and my SSH-keys added to my Digital Ocean account, starting a new Digital Ocean droplet couldn't have been easier. I selected the type of VPS that I wanted, the image I wanted on it, the SSH-Keys I needed, and I was good to go. In 60 seconds I had a working droplet with Ubuntu-14.04 and root access. I was in! 

### Buying a Domain Name
To be honest, I didn't particularly shop around for the cheapest domain name available, and went with the simpler option of GoDaddy.com. All jokes aside, it was a fairly simple and easy process that allowed me to pay via PayPal for my domain name. $13 bucks and I had grubernaut.com as a domain name. 

### Routing DNS from GoDaddy to Digital Ocean
This.... took me a while. I had to remember that GoDaddy.com was the DNR for the domain, and would need different nameservers specified to point to Digital Ocean. After removing all of the A Records, CNAME Records, and MX Records that GoDaddy.com initializes with, I was left with a bare Zone Record on GoDaddy.com. Aside from the three nameservers provided from Digital Ocean. 
<figure>
  <img src="/images/vps/nameservers.png">
  <figcaption>Nameservers from Digital Ocean</figcaption>
</figure>

After adding the correct nameservers to GoDaddy.com, I then had to edit my Digital Ocean's Zone File as well. I needed an A Record that pointed all traffic to my blog, as well as specific trafic for my blog to route correctly. 
<figure>
  <img src="/images/vps/do-dns1.jpg">
  <figcaption>DNS on Digital Ocean</figcaption>
</figure>

Having that all setup, I was now able to ping grubernaut.com, www.grubernaut.com, and blog.grubernaut.com and have them all resolve to my blog droplet's IP address. 

### Setting up Continuous Delivery
Setting up continuous delivery inside of my droplet was fairly easy since I was using Git as a versioning tool. 
First, I logged into my droplet and created a remote git server. 
Then, I initialized a bare repository, and created a new post-receive hook. (This would be where the continuous delivery comes in!). 

{% highlight bash linenos %}
{% raw %}
mkdir grubernaut.blog.git
cd grubernaut.blog.git
git --bare init
touch hooks/post-receive
{% endraw %}
{% endhighlight %}

I opened up the **only** editor there is (ViM) and wrote my post-receive hook. 

{% highlight bash linenos %}
{% raw %}
#!/bin/bash

GIT_REPO=$HOME/grubernaut.blog.git
TMP_GIT_CLONE=$HOME/tmp/grubernaut.blog
PUBLIC_WWW=/var/www/grubernaut.blog

git clone $GIT_REPO $TMP_GIT_CLONE
jekyll build -s $TMP_GIT_CLONE -d $PUBLIC_WWW
rm -Rf $TMP_GIT_CLONE
exit
{% endraw %}
{% endhighlight %}

This hook would clone my repository into a temporary directory, build my site with jekyll, and place the built site into a public directory. And this hook would run after every commit that was received. 

The only thing left to do on my droplet was to configure Apache2 to point to ```/var/www/grubernaut.blog/``` and I was all set!

### Configuring Workstation Repo
On my development workstation, I had to do some slight configuration to be able to push to my droplet and have it be automagically deployed. Luckily this is just a simple one-line git command. 
```bash
git remote add deploy <user>@<site>:~/<path-to-repo>.git
```
From there I can now run a ```git push deploy master``` and git will take care of the deployment chain for me. 

### Final Thoughts and Future Plans
Overall, this was a tremendous learning experience for me, and a great introduction to VPS's. My goal now is to spin up more Droplets on Digital Ocean and start building a full stack of Droplets. 
Things like: 

* A VPN Server
* A ZNC IRC Bouncer (Something I've wanted for a while)
* A Sensu Server to monitor my stack
* etc...

Eventually I would love to have this whole setup served from Chef cookbooks, and fully automated. This was a great introduction to how the infrastructure in Cloud-Land works though. 
