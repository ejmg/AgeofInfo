---
layout: single
title:  "Tutorial: Setting Up Virtualenv and Virtualenvwrapper on Linux for Python 3.x"
date:   2016-11-06 20:42:06
categories: 
- programming 
- linux
- python
- tutorial
tags: 
- ubuntu 
- bots
- twitter
- install
---

Since I last posted in November, I've been doing a lot of work with Python on a few projects of mine. As a result, I have created my own work-flow for Python development.
That being said, one of the first things necessary for proper Python development is familiarizing yourself with and configuring Python *virtual environments* and how to manage them using *virtualenvwrapper.* Consequently, I wanted to write a short and quick guide to a properly configured Python Virtual Environment for Python 3.5+ on a Debian based system for other programmers new to Python and or Linux. 

For the full prerequisites to do this tutorial, look at the appendix at the end of this blog post. Finally, this blog post wouldn't be possible with the tutorials I read during the course of my setup process when starting Python development, notably [Real Python's primer on Virtual Environments](https://realpython.com/blog/python/python-virtual-environments-a-primer/).

# Python Virtual Environments

## Why?

Let's say you want to build a twitter bot and you want to use Tweepy, a common wrapper module for interacting with Twitter's API. Without using virtual environments, installing tweepy would simply require the following command:

```bash
sudo -H pip3 install tweepy
```
What this command does is the following:

1. ``sudo`` gives root privileges to the command so that it is installed system wide. 
2. ``-H`` sets the path for $HOME to that of root.
3. ``pip3 install tweepy`` queries the Python Package Index for the Tweepy module.

You can check that tweepy exists by entering the python3 terminal by importing the module as so:

![import-tweepy]({{ site.url }}/assets/images/posts/2016-12-15/import-tweepy.png)

Now why would installing Tweepy system wide be a problem? Well let's start with where Tweepy is installed, which for Python 3.5 is something like ``/usr/lib/python3.5/dist-packages/tweepy/``. Now there's no problem with this so long as you use the same version of Tweepy across every twitter bot you ever make, but what happens when 2 years later you find yourself with 4 different bots running on four different versions of Tweepy? How do you tell Python 3.5 to use Tweepy 1.1 for bot 1, 1.2 for bot 2, 1.3 for bot 3, and 1.4 for bot 4?

The answer is you *really can't.* This is where Python Virtual Environments come into play. 

### Dependency Management is a Pain and Security is Important

Virtual environments allow you to keep project specific installations of Python and the modules it uses. This is considered a best practice with software development as it ensures your dependencies are kept clean and organized for each project so that maintaining a given project through its life cycle remains a streamlined process. This practice is also known as "sandboxing" from a security standpoint, as you isolate packages to a local directory/environment rather than a system install. You especially want to do this with unverified modules. You can probably install well known/trusted packages system wide without worrying about security (but again, still shouldn't because of dependencies), but you definitely don't want to with ones that haven't been code reviewed by a large community of developers. In other words, why would you give sudo access to a module that you don't trust/know? When someone is executed with sudo privileges, you risk escalating an exploit to system wide access which means your entire machine is compromised, yikes!

## Installing Virtual Environments with Virtualenv

Give the following command with pip3 in term:

```bash
pip3 install virtualenv
```

Now you can create a virtualenv with the following command:

```bash
virtualenv test
```

The above command does the following:

1) It created a subdirectory in your working directory with the name 'test'
2) In this directory, it created a symbolic link to your system's binary of Python3 but treats it as a local installation within this directory
3) When you now install any modules while using the 'test' virtual environment, they'll be installed under ``test/lib/<python_version>/site-packages``

Let's test this. Activate your virtualenv with ``source test/bin/activate``, and you will see your working line in term change to ``(test) <user>@<machine>:<working directory>`` like so: 

![virtualenv]({{ site.url }}/assets/images/posts/2016-12-15/test-virtualenv.png)

'(test)' tells you that you are currently working within the 'test' virtualenv! Congrats! Now see what version of Python3 you are using with the following:

```bash
which python3
```

Because of how virtual environments work, you will see that the Python3 executable used is in ``home/<user_name/test/bin/python3``, rather than the system wide installation, ``/usr/bin/python3``. Now let's do another example: try importing the Tweepy module that we imported earlier in Python 3 from within terminal. You'll notice that Python will tell you no such module exists because virtualenv only looks from within its install directory.

![tweepy-not-found]({{ site.url }}/assets/images/posts/2016-12-15/tweepy-not-found.png)

Remember above how it did work? When you work within a virtual environment, you are working with the locally installed packages of that specific virtual environment. **This is why virtualenv is such a powerful tool.** It allows you to keep your projects, and its dependencies, organized and separate from one another. 

To get out of your virtualenv, simply execute the command ``deactivate`` from within terminal. Go ahead and delete it as well with the following command (be careful to execute it as exactly from within the directory you had created 'test' in):

```bash
sudo rm -rf test
```
While we are at it, let's go ahead and uninstall Tweepy since such a package doesn't need to be accessible system wide:

```bash
sudo -H pip3 uninstall tweepy
```

## Organizing Virtualenv with Virtualenvwrapper

Already, you have a very powerful tool for programming in Python. However, notice the command for activating your virtualenv from earlier, ``source path/to/bin/activate``. This seems a bit burdensome, doesn't it? Also, look at how we had to delete it, ``sudo rm -rf test`` is a sudo level command that has to pass both the recursive and force flags to rm which is a pain and a little dangerous! Furthermore, imagine what happens once you have 5, 15, or 30 different environments - you will have to remember the sourcing path for each one of these, which becomes one extra thing you have to keep track of. This is where Virtualenvwrapper enters: as its name suggests, virtualenvwrapper acts as a wrapper for managing all of your virtual environments that you install, along with making the process of entering, creating, and deleting them much easier. 

Install (system wide) with the following command:

```bash
sudo -H pip3 install virtualenvwrapper
```

After this, we have to finish the install by running the virtualenvwrapper.sh script which should be installed under ``/usr/local/bin/virtualenvwrapper.sh`` but to be sure, run the following to check:

```bash
which virtualenvwrapper.sh
```

Using the directory given, insert at the bottom of your ``.bashrc`` in your user directory the following four lines of code:

```bash
export WORKON_HOME=$HOME/.virtualenvs
export PROJECT_HOME=$HOME/projects
export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3 #this would change if you were doing python2 instead or the non-default python3 (3.4, 3.6, etc)
source /path/to/your/virtualenvwrapper.sh
```

These lines do is the following, in order:
1. set ``.virtualenvs``, the directory where all your new virtualenvs will be organized, to the base of your home directory.
2. set the default directory for your projects at the base of your home directory
3. tells your system to use Python3 instead of Python2 for running virtualenvwrapper.sh, which does not exist as a module for Python2 (assuming you haven't installed it like we have installed it for Python3)
4. sources the script ``virtualenvwrapper.sh`` which will be needed every time you create a new virtual environment with virtualenvwrapper.

Now source your ``.bashrc`` so the above changes are loaded by typing the following command in terminal:

``source ~/.bashrc``

You should see something like the following:

![virtualenvwrapper-install]({{ site.url }}/assets/images/posts/2016-12-15/virtualenvwrapper-install.png)

Congrats! Now under ``home/<user_name>/``, you will see the directory ``.virtualenvs/`` which will contain every virtualenv you manage with virtualenvwrapper! Go ahead and create one with the following command:

```bash
mkvirtualenv test
```

You'll automatically be entered into your new virtualenv and from here, it acts no differently had you done it with only the ``virtualenv test`` command from earlier. But unlike before, you can now use ``rmvirtualenv <virtualenv>`` to delete a virtualenv in one simple command and ``workon <virtualenv>`` to switch to your virtualenv of choice! Even if you can't remember the names of your virtual environments, just entering ``workon`` and hitting tab will then show all existing virtual environments so far. This behavior is mirrored with the other commands of virtualenvwrapper.

And with that, you are done! Congrats on setting up virtualenv and virtualenvwrapper on Linux, you are now on your way to being a Python ninja!

# Appendix: Prerequisites

1. Ubuntu 16.04 LTS
   This guide will assume you are running Ubuntu 16.04, the most recent version of the Ubuntu Linux distribution. That said, the instructions found here can easily be applied to Ubuntu derived distributions, Debian Wheezy+, and other Debian derivatives with some tweaking.

2. Python 3.4+
   While Ubuntu 16.04 comes pre-installed with Python 3.5, the instructions below apply to any version from 3.4 on.
   
3. pip3 for Python 3
   pip (Pip Installs Packages) is a module management system for Python. If you do not have pip3, it can be installed with the following command for Ubuntu 16.04:
   
   ```bash
   sudo apt install python3-pip
   ```
   
   for 14.04:
   
   ```bash
   sudo apt-get install python3-pip3
   ```
   
   After installing, make sure to update pip with:
   
   ``pip3 install --upgrade pip``
   
   To update the system wide (root) version of pip:
   
   ``sudo -H pip3 install --upgrade pip``
   
   ``pip3 --version`` should now give you the most recent version (as of this writing, that's 9.0.1).
