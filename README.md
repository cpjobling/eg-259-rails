# A Rails-ready box for EG-259

## Creation steps

This is what I did to get this ready. I followed the instructions provided by Ryan Bates in his excellent RailsCast episode 292 [Virtual Machines with Vagrant](http://railscasts.com/episodes/292-virtual-machines-with-vagrant?view=asciicast)  (See also the [accompanying screencast](http://railscasts.com/episodes/292-virtual-machines-with-vagrant)).

First we will need to download a virual box image for Ubuntu 12.04 Server (LTS Edition)

    $ vagrant box add precise32 http://files.vagrantup.com/precise32.box

Next we create a new rails project

    $ rails new eg-259-rails

And add the `Vagrant` file that will enable this rails app to execute inside the virual machine:

    $ vagrant init precise32

(If you cloned this project from GitHub, the second two steps have already been done for you. If you wish to build your own virtual rails box, you will need to install ruby and rails for your platform yourself. There is a thorough tutorial on doing this in [Chapter 1 From Zero to Deploy](http://ruby.railstutorial.org/ruby-on-rails-tutorial-book#top) of the Michel Hartl's excellent [Rails Tutorial](http://ruby.railstutorial.org).)


## Getting rails working

First start up the virtualvagran machine

    $ vagrant up

Then ensure that the dependencies are installed. To do this we need to login to our running virtual Ubuntu server:

     $ vagrant ssh
     Welcome to Ubuntu 12.04 LTS (GNU/Linux 3.2.0-23-generic-pae i686)
     
       * Documentation:  https://help.ubuntu.com/
     Welcome to your Vagrant-built virtual machine.
     Last login: Sat Apr 13 17:08:20 2013 from 10.0.2.2
     vagrant@precise32:~$

Then we need to ensure that we have the packages we will need to build ruby and its dependencies

     vagrant@precise32:~$ sudo apt-get update
     vagrant@precise32:~$ sudo apt-get install build-essential zlib1g-dev \
     git-core sqlite3 libsqlite3-dev

We then install Ruby using [rbenv](https://github.com/sstephenson/rbenv) (please note that though I've dropped the full prompt `agrant@precise32:~$` these commands are still run from within the same `vagrant ssh` session.)

    $ cd # puts us into the "home" directory /home/vagrant
    $ git clone git://github.com/sstephenson/rbenv.git .rbenv

Now we add `rbenv`'s bin directory to our `PATH` and add the `init` command to our bash profile. This ensures that `rbenv` will be available whenever we log in to the virtual machine.

    $ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
    $ echo 'eval "$(rbenv init -)"' >> ~/.bash_profile
    $ tail ~/.bash_profile
    export PATH="$HOME/.rbenv/bin:$PATH"
    eval "$(rbenv init -)"

Now reload the bash profile for these chnages to take effect

    $ source .bash_profile
    $ rbenv -h
    rbenv 0.4.0-42-g0556849
    Usage: rbenv <command> [<args>]
    
    Some useful rbenv commands are:
       commands    List all available rbenv commands
       local       Set or show the local application-specific Ruby version
       global      Set or show the global Ruby version
       shell       Set or show the shell-specific Ruby version
       rehash      Rehash rbenv shims (run this after installing executables)
       version     Show the current Ruby version and its origin
       versions    List all Ruby versions available to rbenv
       which       Display the full path to an executable
       whence      List all Ruby versions that contain the given executable

    See `rbenv help <command>' for information on a specific command.
    For full documentation, see: https://github.com/sstephenson/rbenv#readme

The command `rbenv` provides a way to manage ruby versions and "gemsets" for development projects. We need a way to install a particular version. There is another tool [Ruby Build](https://github.com/sstephenson/ruby-build) that we can use for that. It is installed like this:

    $ git clone https://github.com/sstephenson/ruby-build.git
    $ cd ruby-build
    $ sudo ./install.sh

Since Ruby Vesrion 2.0 has jist come out, we may as well install that:

    $ rbenv install 2.0.0-p0

This will take a while. Time for a coffee!

To make 2.0.0 the default version of Ruby on our virtual machine:

    $ rbenv global 2.0.0-p0
    $ ruby -v
    ruby 2.0.0p0 (2013-02-24 revision 39474) [i686-linux]

## Installing the Rails app

Because of the way vagrant works, the folder eg-259-rails which contains the `Vagrant` file is mapped to path `/vagrant` on the guest operating system. Because we created a rails app in this same folder, `/vagrant` is also the "home folder" for rails on the guest host. To get rails running on the guest, we need to change directory:

    $ cd /vagrant

The we need to install the bundler gem to install the bundle command that manages Rails' dependencies.

    $ gem install bundle

We run the `rbenv rehash` command to get `bundle` recognised as an executable script

    $ rbenv rehash
    $ bundle --help

Rails needs a Javascript runtime to do some of the things that it does, for example to manage the "assets pipline" which concatonates and minfies JavaScript and CSS files for the production server. Ubuntu doesn't have a JavsScript run-time so we have to add the following line to the `Gemfile`:

    gem 'therubyracer'

This can be done by opening the file `eng-259-rails/Gemfile` in your favourite gost text editor, or by editing the file from the command line in the virtual machine using `vi` or `nano`. Once you've done that and saved the modified file, run the following from the virtual machine's command line:

    $ bundle exec rails server

This will start up a web server (called WEBrick) that listens to port 3000. However, the host can only forward client requests to that port, if host port 3000 is forwarded to the guest's port 3000. 

Open the file Vagrant in a text editor and look for the commented out line: 

    # config.vm.network :forwarded_port, guest: 3000, host: 3000

 remove the comment marker `#` so that the file looks like:

    config.vm.network :forwarded_port, guest: 3000, host: 3000

You will then need to close down the rails server using Ctrl-C (`^C`), exit the login using `exit`, to return to the host prompt, then issue:

     host $ vagrant reload

To get the system to load the changes in port assignment.

Log back into the guest, and restart rails:

    host $ vagrant ssh
    $ cd /vagrant
    $ bundle exec rails server

Now open your browser, and navigate to <http://localhost:3000/>. You should see the rails welcome page.

## Learning Rails

Code School has a free (fun) beginner's tutorial [Rails for Zombies](http://railsforzombies.org/) that will get you started, and before any serious development, I would recommend that you work through Michael Hartl's [Ruby on Rails Tutorial](http://ruby.railstutorial.org/). Ryan Bates' [Railscasts](http://railscasts.com/) is an excellent source of top-up knowledge. In the context of EG-259, the episodes Backbone on Rails ([Part 1](http://railscasts.com/episodes/323-backbone-on-rails-part-1)) and ([Part 2](http://railscasts.com/episodes/325-backbone-on-rails-part-2)) and [Twitter Bootstrap Basics](http://railscasts.com/episodes/328-twitter-bootstrap-basics) are paricularly relevant.






