---
layout: post
title: "Setting up my Development Environment"
modified: 2014-05-04 14:21:39 +0100
tags: [Setup, Development]
image: 
  feature:	tools.jpg 
  credit: 
  creditlink: 
comments: true
share: true
---
Setting up your development environment can be a pain. On my first attempt I spent most of the day searching google and stackoverflow trying to solve installation issues. I even tried running other people's install scripts and my mac ended up like frankinstein with crazy customisations I didn't need. This guide is intended to get a new OSX instance up and running for development using Ruby on Rails and Node JS. You won't need everything I have installed just pick and choose what you need.

##Install Chrome
I love using google chrome, and its the first thing I download and install on a new install of mac osx.

[Download](http://www.google.com/chrome)

##Install iTerm2
iTerm2 is much better than using the standard OSX terminal.

[Download](http://www.iterm2.com)

now to add our /usr/local/bin path to our PATH environment.

{% highlight bash %}
$ export PATH=/usr/local/bin:$PATH
{% endhighlight %}


##Install XCode Command Line Tools
{% highlight bash %}
$ xcode-select --install
{% endhighlight %}

##Homebrew
{% highlight bash %}
$ ruby -e "$(curl -fsSL https://raw.github.com/Homebrew/homebrew/go/install)"
{% endhighlight %}

##RVM
Download and install the latest stable release of RVM

{% highlight bash %}
$ \curl -sSL https://get.rvm.io | bash -s stable
{% endhighlight %}
Then add the rvm binaries folder to my PATH environment settings

{% highlight bash %}
$ source ~/.rvm/scripts/rvm
{% endhighlight %}

##Upgrade Ruby
By default, OSX Mavericks installs ruby version 2.0. To upgrade to the latest ruby version 2.1.1 do the following:

{% highlight bash %}
$ rvm get latest
$ rvm install 2.1
{% endhighlight %}

now to check what version of ruby our mac is running.

{% highlight bash %}
$ ruby -v
ruby 2.1.1p76 (2014-02-24 revision 45161) [x86_64-darwin12.0]
{% endhighlight %}

if it still says its on ruby 2.0 just execute the following command in your terminal.

{% highlight bash %}
$ rvm use 2.1
Using /Users/yoshi/.rvm/gems/ruby-2.1.1
{% endhighlight %}

##Install the Heroku toolbelt
Heroku is a PaaS (Platform as a Service) for web applications. It allows for easy deployment of webapps to their cloud based servers and its free for development instances. 

[Download](https://toolbelt.heroku.com/)
{% highlight bash %}
$ heroku login
{% endhighlight %}

We will also need to create a ssh key and add it to our heroku account.

{% highlight bash %}
$ ssh-keygen -t rsa
$ heroku keys:add
{% endhighlight %}

Now to test if its all working with heroku.

{% highlight bash %}
$ mkdir ~/myapp
$ cd ~/myapp
$ heroku create
Creating sheltered-dawn-1053... done, stack is cedar
http://sheltered-dawn-1053.herokuapp.com/ | git@heroku.com:sheltered-dawn-1053.git
{% endhighlight %}

##Git and Github
Installing the heroku toolbelt will also install git. To update to the latest version of git do the following:

{% highlight bash %}
$ brew install git
$ git --version
git version 1.9.2
{% endhighlight %}


Setup your global variables

{% highlight bash %}
$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com
$ git config --global push.default simple
{% endhighlight %}

Now to link it up to our github account and to test if its all working.
First create a `myapp` repo in github.

{% highlight bash %}
$ git init
$ touch README.md
$ git add .
$ git commit -m "first commit"
$ git remote add origin https://github.com/yoshdog/myapp.git
$ git push -u origin master
{% endhighlight %}

check back on your github repo and the README file should appear.

##Install Postgres

{% highlight bash %}
$ brew install postgresql
{% endhighlight %}

Check if its installed correctly and is the latest version.

{% highlight bash %}
$ postgres --version
postgres (PostgreSQL) 9.3.4
{% endhighlight %}

##Install Ruby Gems

To install Rails, just install it via ruby's package management system.

{% highlight bash %}
$ gem install rails
$ rails -v
Rails 4.1.0
{% endhighlight %}

Now to install RSPEC.

{% highlight bash %}
$ gem install rspec
$ rspec -v
2.14.8
{% endhighlight %}

##Install node.js
Although node.js is another language and framework I like to install so I can run Javascript files in terminal and using the npm packaging module I can install front end frameworks pretty easily.
[Download](http://nodejs.org)

Check if its install correctly.

{% highlight bash %}
$ node --version
v0.10.28
{% endhighlight %}

Install Yeoman / Grunt / Bower

{% highlight bash %}
$ npm install -g yo
{% endhighlight %}

##Install Alfred
To be honest, OSX's spotlight search sucks. I love using alfred instead as you can customise it.

Install it from: [here](http://www.alfredapp.com)
To unlock all the customised features and to build workflows you will need to purchase it.

##Dash Doc
I love having dashdoc as it lets me have an offline version of documentation of the popular programming languages and frameworks.

Download it from: [here](http://london.kapeli.com/Dash.zip)

The great thing also is I can customise sublime text and alfred to lookup certain documentation.

##Sublime Text
I am currently using sublime text as my text editor. Although I still use vim, I still like using sublime as its fast and customisable.

Download it: [here](https://www.sublimetext.com/3)

After this you will need to create some shortcuts for sublime and subl for your terminal.


{% highlight bash %}
$ ln -s "/Applications/Sublime Text.app/Contents/SharedSupport/bin/subl" /usr/local/bin/sublime
$ ln -s /Applications/Sublime\ Text.app/Contents/SharedSupport/bin/subl /usr/local/bin/subl
{% endhighlight %}