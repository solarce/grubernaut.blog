---
layout: post
title: "Syncing Sublime Text with Dropbox"
modified: 2014-04-29 22:49:50 -0400
tags: [dev, development bed, configurations]
image: abstract-3.jpg
comments: true
share: true
---
<figure>
	<img src="/images/sublime/sublime1.png">
	<figcaption>Current Sublime Text 2 Setup</figcaption>
</figure>

### Syncing User Packages for Sublime Text 2
Recently (read as: "This Week") I've nearly fallen in love with [Sublime Text](http://www.sublimetext.com) and you should seriously try it out if you haven't already. You can use Sublime Text and evaluate it for free, yet a license must be purchased for continued use. I've used it for the past week now, and started the process of configuring my development bed around Sublime Text. 
I'll cover my Sublime Text configuration later on in the post. 

One of the first issues I ran into with Sublime Text was maintaining a concurrent setup across my iMac at work and my MBP at home. As a native Vim lover, I was used to creating a symbolic link to a Dropbox file of my .vimrc and .vim directories. And in fact, it is quite the same process for Sublime Text. 

## OSX
In OSX, first find which machine you wish to copy the settings from. 
_hint:_ it's the Sublime Text 2 that you have already fully configured to your liking at the time. 

On that machine make sure that Sublime Text 2 is fully closed, then create a Dropbox folder to hold your User Packages folder. Then create a symbolic link between the two. 
{% highlight bash linenos %}
{% raw %}
cd ~/Library/Application\ Support/Sublime\ Text\ 2/Packages/
mkdir ~/Dropbox/Sublime
mv User ~/Dropbox/Sublime/
ln -s ~/Dropbox/Sublime/User
{% endraw %}
{% endhighlight %}

_Note:_ If your Dropbox folder is mounted in a different directory than ```~/Dropbox``` then replace the directory path specified above with the path to your local Dropbox directory. 

**Sublime Text 3:** Simply replace the ```Sublime\ Text\ 2/``` directory with: ```Sublime\ Text\ 3/``` directory path. 

Then on every subsequent machine that you wish to sync your Sublime Text settings with, remove your User Packages directory and create a symbolic link to your Dropbox User directory. 
{% highlight bash linenos %}
{% raw %}
cd ~/Library/Application\ Support/Sublime\ Text\ 2/Packages/
rm -r User
ln -s ~/Dropbox/Sublime/User
{% endraw %}
{% endhighlight %}

When you start up Sublime Text again on the subsequent machines, Sublime Text will start to download and install all the packages from your User Packages directory. As well as implement all of your settings specified in your User Settings. 
_Note:_ If you are working with a fresh install of Sublime Text, you will have to install [Package Control](https://sublime.wbond.net/installation#st2) before any packages can be synced. 

### My Sublime Text Configuration
_As of 4/29/14. WIP_

My Currently Used Packages:

* [Git](https://github.com/kemayo/sublime-text-git/wiki)
* [Markdown Preview](https://github.com/revolunet/sublimetext-markdown-preview)
* [SFTP](http://wbond.net/sublime_packages/sftp)
* [MacTerminal](https://github.com/afterdesign/MacTerminal)
* [GitGutter](https://github.com/jisaacks/GitGutter) - Extremely Nice, Highly recommended. 
* [Sublime Block Cursor](https://github.com/netpro2k/SublimeBlockCursor) - Because with vintage mode it's near impossible to see the cursor in default. 
* [Theme - Soda](https://github.com/buymeasoda/soda-theme/) - Really makes Sublime more _Mac_ like. Beautiful. 
* [Package Control](https://sublime.wbond.net/installation)

My Current User Settings: 
{% highlight json linenos %}
{% raw %}
{
	"bold_folder_labels": true,
	"color_scheme": "Packages/Color Scheme - Default/Monokai.tmTheme",
	"fade_fold_buttons": false,
	"highlight_line": true,
	"ignored_packages":
	[
	],
	"line_padding_bottom": 1,
	"line_padding_top": 1,
	"theme": "Soda Dark.sublime-theme",
	"vintage_start_in_command_mode": true
}
{% endraw %}
{% endhighlight %}

Yes, of course I use vintage mode, and start in command mode. Some habits never change.



> Life is like topography, Hobbes. There are summits of happiness and success, flat stretches of boring routine and valleys of frustration and failure.  