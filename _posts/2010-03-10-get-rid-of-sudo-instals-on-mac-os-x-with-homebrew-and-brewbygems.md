--- 
created_at: 2010-03-10 13:10:20.681903 +01:00
layout: post
title: Get rid of sudo installs on Mac OS X with Homebrew and Brewbygems
---

I decided to do a clean install of Snow Leopard on my 24" iMac at home over the last weekend, one of the biggest motivators behind it was to set up a Ruby development environment where I only had to **sudo** in exceptional cases and could avoid access control / ownership issues of executables or RubyGems scattered around multiple gem paths.

There were two scenarios in the past that lead to using sudo to gain access to /usr/bin (owned by root:wheel) and place executables there:

* sudo port install nmap - MacPorts, the package manager for compiling anything that Apple has left out (nmap, wget, watch, git, just to name a few)
 * sudo gem install rake - RubyGems having executables placed outside the gem path, /usr/bin/rake in this case

Homebrew and Brewbygems offer a simple and elegant solution to this problem. I've also set this up on my MacBook Pro while writing this post to test it so all scripts should work as advertised :)

Third Party Applications
========================

[Homebrew](http://github.com/mxcl/homebrew) sets itself up in /usr/local by default and it is completely self-contained. Even better, /usr/local/bin is included in the default PATH environment variable so we don't even need to add that. Running a Ruby [gist](http://gist.github.com/327800) will take ownership of /usr/local and extract Homebrew there.

{% highlight bash %}
malpais:~ rgabo$ curl http://gist.github.com/gists/327800/download | tar -xO | ruby
{% endhighlight %}

At this point, brew is set up and can install from source anything that has a *formula* for. Given it updates itself using **git** from [Github](http://github.com), it's worth installing that first, if you haven't done so before.

{% highlight bash %}
malpais:~ rgabo$ which brew
/usr/local/bin/brew

malpais:~ rgabo$ brew install git
==> Downloading http://kernel.org/pub/software/scm/git/git-1.7.0.2.tar.bz2
######################################################################## 100.0%
==> make prefix=/usr/local/Cellar/git/1.7.0.2 install
==> Downloading http://kernel.org/pub/software/scm/git/git-manpages-1.7.0.2.tar.bz2
######################################################################## 100.0%
/usr/local/Cellar/git/1.7.0.2: 403 files, 12M, built in 60 seconds

malpais:~ rgabo$ brew update
remote: Counting objects: 87, done.
remote: Compressing objects: 100% (75/75), done.
remote: Total 75 (delta 24), reused 0 (delta 0)
Unpacking objects: 100% (75/75), done.
From git://github.com/mxcl/homebrew
 * branch            master     -> FETCH_HEAD
Updated Homebrew from 8eefb17e to c6923dce.
==> The following formulae were updated:
...
{% endhighlight %}

RubyGems
========

Snow Leopard comes with Ruby 1.8.7 and RubyGems 1.3.0 along with a bunch of pre-installed gems. Running *gem env* shows the RubyGems Environment:

{% highlight bash %}
malpais:usr rgabo$ gem env
RubyGems Environment:
  - RUBYGEMS VERSION: 1.3.0
  - RUBY VERSION: 1.8.7 (2008-08-11 patchlevel 72) [universal-darwin10.0]
  - INSTALLATION DIRECTORY: /Library/Ruby/Gems/1.8
  - RUBY EXECUTABLE: /System/Library/Frameworks/Ruby.framework/Versions/1.8/usr/bin/ruby
  - EXECUTABLE DIRECTORY: /usr/bin
  - RUBYGEMS PLATFORMS:
    - ruby
    - universal-darwin-10
  - GEM PATHS:
     - /Library/Ruby/Gems/1.8
     - /Users/rgabo/.gem/ruby/1.8
     - /System/Library/Frameworks/Ruby.framework/Versions/1.8/usr/lib/ruby/gems/1.8
  - GEM CONFIGURATION:
     - :update_sources => true
     - :verbose => true
     - :benchmark => false
     - :backtrace => false
     - :bulk_threshold => 1000
  - REMOTE SOURCES:
     - http://gems.rubyforge.org/
{% endhighlight %}

While the default gem path (/Library/Ruby/Gems/1.8) is writable, /usr/bin isn't so installing without sudo results with the following warnings and a lot of headaches.

{% highlight text %}
WARNING:  Installing to ~/.gem since /Library/Ruby/Gems/1.8 and /usr/bin aren't both writable.
WARNING:  You don't have /Users/rgabo/.gem/ruby/1.8/bin in your PATH, gem executables will not run.
{% endhighlight %}

Using [Brewbygems](http://github.com/indirect/brewbygems/) combines the power and simplicity of Homebrew and RubyGems by configuring RubyGems to use a directory in Homebrew's Cellar (/usr/local/Cellar/gems) as a gem path. Brewbygems hooks *gem install* and links any executables into /usr/local/bin, keeping everything self contained under /usr/local. System-wide gems available under /Library/Ruby or even /System/Library/Frameworks/Ruby.framework will still be available with any executables sudo'd into /usr/bin, although I recommend the clean approach where all gems are self-contained inside Homebrew and no gems / executables live outside.

The following bash snippet sets up your .bashrc to export GEM_HOME, sources it, and installs brewbygems.

{% highlight bash %}
echo "export GEM_HOME='$(brew --prefix)/Cellar/gems/1.8'" >> ~/.bashrc
source ~/.bashrc
gem install brewbygems
{% endhighlight %}

We also have to make sure that ~/.bashrc gets sourced by ~/.bash_profile as Terminal.app interactive login sessions will not invoke it by default.

{% highlight bash %}
# .bash_profile
# source .bashrc if exists
if [ -f ~/.bashrc ]; then
   source ~/.bashrc
fi
{% endhighlight %}

Installing Rails (*gem install rails*) for instance will link all executables to /usr/local/bin and as an added extra, all RubyGems will show up as a single *gems* formula inside Homebrew's Cellar, making it easy to track the size it takes up or even remove it altogether.

{% highlight bash %}
malpais:bin rgabo$ gem install rails
1 links created for /usr/local/Cellar/gems/1.8
2 links created for /usr/local/Cellar/gems/1.8
3 links created for /usr/local/Cellar/gems/1.8
...
8 gems installed
...

malpais:bin rgabo$ ls -l /usr/local/bin

-rwxr-xr-x  1 rgabo  staff  11970 Mar  9 09:46 brew
lrwxr-xr-x  1 rgabo  staff     29 Mar 10 11:32 git -> ../Cellar/git/1.7.0.2/bin/git
lrwxr-xr-x  1 rgabo  staff     29 Mar 10 12:03 rackup -> ../Cellar/gems/1.8/bin/rackup
lrwxr-xr-x  1 rgabo  staff     28 Mar 10 12:03 rails -> ../Cellar/gems/1.8/bin/rails
lrwxr-xr-x  1 rgabo  staff     27 Mar 10 12:03 rake -> ../Cellar/gems/1.8/bin/rake
...

malpais:bin rgabo$ du -sh /usr/local/Cellar/*
 59M	/usr/local/Cellar/gems
 16K	/usr/local/Cellar/gist
 12M	/usr/local/Cellar/git
 48K	/usr/local/Cellar/watch
776K	/usr/local/Cellar/wget

{% endhighlight %}

For more information, check out the [Homebrew Wiki](http://wiki.github.com/mxcl/homebrew).
