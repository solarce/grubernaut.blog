---
layout: post
title: "Kitchen Magic"
modified: 2014-12-08 12:50:10 -0500
tags: [docker, test-kitchen, tooling, technical]
image: abstract-4.jpg
comments: true
share: true

---

# Creating some kitchen magic with Puppet
At my current organization (The lovely folks at [Minted](http://www.minted.com)), we use [Puppet](http://puppetlabs.com/puppet/what-is-puppet) for configuration management. At first I was unsure of how to get up to speed on using Puppet, having gotten used [Chef](http://chef.io) and it's development ecosystem in a past life, but I decided to give it my all. The main thing that has bugged me, about learning and using Puppet is the lack of a prescribed testing framework. There are tools that are still maturing every day and actually work quite well for their designed purpose. [Beaker](https://github.com/puppetlabs/beaker) is one such tool that works fairly well, but Beaker has a dependency on a Puppet Master. We currently run a masterless Puppet, using Git and Fabric, to schedule and orchestrate Puppet runs on individual nodes. We needed a localized testing framework to test our Puppet manifests and modules without changing our whole internal infrastructure and I set out to find one.

Some of the key requirements that we held for a testing framework were:

1. Local Testing
2. Easy Integration with current testing frameworks (Serverspec, RSpec, BATS, etc...)
3. Parallel testing
4. Cheap
5. Fast

## Let's get cooking
Enter [Test-Kitchen](https://github.com/test-kitchen/test-kitchen). `test-kitchen` provides us with the framework to test all of our puppet modules, in one run, in parallel, while using a busser for our existing testing framework.

"But Wait!", you say. "`test-kitchen` is for Chef!". This is where [kitchen-puppet](https://github.com/neillturner/kitchen-puppet) comes in. `kitchen-puppet` is a provisioner driver for test-kitchen. It uses the same interfaces that Chef-Solo and Chef-Zero tap into. Since our organization uses masterless puppet, this fits into our infrastructure with grace. `kitchen-puppet` allows us to use `test-kitchen`'s existing testing framework, and provision our test subject with a `puppet apply` command. Be sure to read the full set of [Provisioner Options](https://github.com/neillturner/kitchen-puppet/blob/master/provisioner_options.md) for `kitchen-puppet` as well. Not only can you set provisioner options in the provisioner config block in the test-kitchen yaml config, but you can set provisioner options per test suite. We use this functionality at Minted to test different roles based on custom [Facter Facts](https://docs.puppetlabs.com/facter/latest/core_facts.html) passed to the test suites.

## Cooking with containers
Enter [kitchen-docker](https://github.com/portertech/kitchen-docker). `kitchen-docker` allows us to have exceedingly fast tests ran against multiple test suites at the same time. Plus it's [Docker](http://docker.io), so you know it's cool. If you're running on an inferior Operating System such as OSX, you'll need to install and become familiar with [boot2docker](http://boot2docker.io/) before moving any further. Our Kitchen setup takes advantage of _boot2docker_ by using the `DOCKER_HOST` environment variable. Depending on how many suites you want to run and how many you want to test in a convurrent mannor, it may no longer be wise to run all of your containers locally, whether through _boot2docker_ or natively. Using the environment variable `DOCKER_HOST`, it is even possible to spin up a remote EC2 instance to host all of your testing containers. This situation would be extremely beneficial for automated testing from Jenkins, TravisCI, and others.

## Our Setup

I have created a github repo, [grubernaut-kitchen-example](https://github.com/grubernaut/kitchen-example),for this post as well. Simply clone the repo, and test to your liking. :)

{% highlight bash linenos %}
{% raw %}
.
├── Gemfile
├── Gemfile.lock
├── manifests
│   └── site.pp
├── test
│   └── integration
│       └── webserver
│           └── serverspec
│               └── default_spec.rb
└── ubuntu12.04-dockerfile

{% endraw %}
{% endhighlight %}

## Mise-en-place

*Getting things setup*

**Gemfile:**
{% highlight ruby linenos %}
{% raw %}
source "https://rubygems.org"

gem 'rake'
gem 'serverspec'
gem 'puppet'
gem 'kitchen-docker'
gem 'kitchen-puppet'
gem 'test-kitchen'
{% endraw %}
{% endhighlight %}

Notice in the following configuration file, we specifically declare which manifest to run under the provisioner hash. This is not necessary, of course, but merely a way to show how one can set provisioner-specific options for individual suites. We use this at Minted to set custom facter facts per test suite.

**.kitchen.yml:**
{% highlight yaml linenos %}
{% raw %}

---
driver:
  name: docker

provisioner:
  name: puppet_apply
  puppet_version: 3.5.1-1puppetlabs1
  manifests_path: manifests
  modules_path: modules
  require_chef_for_busser: true

platforms:
  - name: ubuntu
    driver_config:
      image: ubuntu:12.04
      socket: <%= ENV['DOCKER_HOST'] %>
      use_cache: true
      dockerfile: ubuntu12.04-dockerfile

suites:
  - name: webserver
    provisioner:
      manifest: site.pp

{% endraw %}
{% endhighlight %}

One very important peice of the puzzle is the platform configuration. Here we are setting all of the settings necessary for `kitchen` and and `kitchen-docker` driver to talk to each other. Remember, if you are running [Linux](http://archlinux.org) natively, you shouldn't have to set a `DOCKER_HOST` environment variable. Unless, of course, you wish to run docker on a remote host. When running on OSX, you should already have the `DOCKER_HOST` environment variable set as a result of installing _boot2docker_.

We can, by all means, add custom provisioning steps to the base docker image, but I like to separate these steps out into a Dockerfile. There are, however, excellent base images for Ubuntu 14.04, see [Phusion's baseimage-docker project](https://github.com/phusion/baseimage-docker); but I wanted to highlight the `kitchen-docker` feature of including your own custom [Dockerfile](https://docs.docker.com/reference/builder/) inside of `test-kitchen`. This will allow you to customize the docker images for your organization beyond the scope of a base image. And that is what I am doing here with this Dockerfile

**ubuntu12.04-dockerfile:**
{% highlight bash linenos %}
{% raw %}
# Use Ubuntu Precise
FROM ubuntu:12.04

## Basic Docker Setup for Ubuntu
# Disable Upstart
RUN dpkg-divert --local --rename --add /sbin/initctl
RUN ln -sf /bin/true /sbin/initctl
ENV DEBIAN_FRONTEND noninteractive
RUN apt-get -y update
RUN apt-get install -y sudo openssh-server curl lsb-release

## Add Test-Kitchen Dependencies
RUN useradd -d /home/kitchen -m -s /bin/bash kitchen
RUN echo 'kitchen:kitchen' | chpasswd
RUN echo 'kitchen ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
{% endraw %}
{% endhighlight %}

Inside of the above kitchen config I added custom facter facts for the webserver suite to simply show how to set provisioner-specific options on a suite-by-suite basis. Now that we have the groundwork laid out, we need a manifest to provision. This is a very basic site manifest, but should be enough to demonstrate our purposes.

**manifests/site.pp:**

{% highlight puppet linenos %}
{% raw %}
# Install a package
package { 'vim':
ensure => installed,
}

# Install a user called snorlax
user { 'snorlax':
ensure => 'present',
groups => ['sudo'],
home => '/home/snorlax',
managehome => true,
shell => '/bin/bash',
}
{% endraw %}
{% endhighlight %}

We also need to write a [serverspec](http://serverspec.org/) test for the manifest above. Again, keeping with our [KISS](http://en.wikipedia.org/wiki/KISS_principle) theme, we should test all of the resources that we have defined in our manifest. Notice the file path for the serverspec test. This is required by the `test-kitchen` busser, which we will talk about later.

**test/integration/webserver/serverspec/default_spec.rb:**

{% highlight ruby linenos %}
{% raw %}
# Encoding: utf-8
require 'serverspec'

set :backend, :exec
set :path, '/sbin:/usr/local/sbin:$PATH'

describe package('vim') do
  it { should be_installed }
end

describe user('snorlax') do
  it { should exist }
end
{% endraw %}
{% endhighlight %}

We should have the groundwork all laid out for our testing to begin.

### Fire 'Ze Missiles!!!

We need gems installed,of course, to be able to test our infrastructure that we are creating.

{% highlight bash %}
% bundle install
{% endhighlight %}

Once all of our gems are installed, we should be able to run `test-kitchen` and see our instance suite waiting to be converged:

{% highlight bash %}
% kitchen list
Instance              Driver  Provisioner  Last Action
webserver-ubuntu      Docker  PuppetApply  Created
{% endhighlight %}

If everything went smoothly, we should be able to now Create, Converge, and Test our "webserver" instance. Notice how `test-kitchen` actually installs chef during the output of the following command:

{% highlight bash %}
{% raw %}
% kitchen converge webserver-ubuntu
Installing Chef
installing with dpkg...
Selecting previously unselected package chef.
(Reading database ...
(Reading database ...        39770 files and directories currently installed.)
Unpacking chef (from .../chef_12.0.3-1_amd64.deb) ...
Setting up chef (12.0.3-1) ...
Thank you for installing Chef!
Transfering files to <webserver-ubuntu>
{% endraw %}
{% endhighlight %}

We actually want `test-kitchen` to behave in this fashion. This allows us to use [chef-busser](https://github.com/test-kitchen/busser) for all of our tests. Which is the real point of this whole project! Now we can run `kitchen test` and `test-kitchen` will create our docker instance, provision the instance with puppet, test the instance with serverspec, and then destroy our instance. All in one command:

{% highlight bash %}
{% raw %}
Finished converging <webserver-ubuntu> (1m51.92s).
-----> Setting up <webserver-ubuntu>...
Fetching: thor-0.19.0.gem (100%)
Successfully installed thor-0.19.0
Fetching: busser-0.6.2.gem (100%)
Successfully installed busser-0.6.2
2 gems installed
-----> Setting up Busser
Creating BUSSER_ROOT in /tmp/busser
Creating busser binstub
Plugin serverspec installed (version 0.5.3)
-----> Running postinstall for serverspec plugin
Finished setting up <webserver-ubuntu> (0m9.15s).
-----> Verifying <webserver-ubuntu>...
Suite path directory /tmp/busser/suites does not exist, skipping.
Uploading /tmp/busser/suites/serverspec/default_spec.rb (mode=0644)
-----> Running serverspec test suite
-----> Installing Serverspec..
Fetching: net-ssh-2.9.1.gem (100%)
Fetching: net-scp-1.2.1.gem (100%)
Fetching: specinfra-2.10.4.gem (100%)
Fetching: multi_json-1.10.1.gem (100%)
Fetching: diff-lcs-1.2.5.gem (100%)
Fetching: rspec-support-3.1.2.gem (100%)
Fetching: rspec-expectations-3.1.2.gem (100%)
Fetching: rspec-core-3.1.7.gem (100%)
Fetching: rspec-its-1.1.0.gem (100%)
Fetching: rspec-mocks-3.1.3.gem (100%)
Fetching: rspec-3.1.0.gem (100%)
Fetching: serverspec-2.7.1.gem (100%)
-----> serverspec installed (version 2.7.1)
/opt/chef/embedded/bin/ruby -I/tmp/busser/suites/serverspec -I/tmp/busser/gems/gems/rspec-support-3.1.2/lib:/tmp/busser/gems/gems/rspec-core-3.1.7/lib /opt/chef/embedded/bin/rspec --pattern /tmp/busser/suites/serverspec/\*\*/\*_spec.rb --color --format documentation --default-path /tmp/busser/suites/serverspec

Package "vim"
should be installed

User "snorlax"
should exist

Finished in 0.05471 seconds (files took 0.17188 seconds to load)
2 examples, 0 failures
Finished verifying <webserver-ubuntu> (0m7.45s).
{% endraw %}
{% endhighlight %}

You can also run `kitchen verify` to test an already running instance without destroying the instance.

Hopefully this helps you setup and start testing your puppet infrastructure using `test-kitchen`, Puppet, and Docker!
