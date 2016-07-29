---
layout: post
title: "Python virtualenv"
date: 2016-07-29 17:55:18 -0400
comments: true
categories: [python]
keywords: virtualenv, virtualenvwrapper, python, python virtual environments, python virtualenv, python virtualenvwrapper
description: Introduction to Python virtual environments
---


In this post I am going to introduce Python's virtual environments, through the **virtualenv** tool. With this tool, you can keep separate environments for your projects, with their own dependencies and executables. By isolating the project environments, you can keep them neat and organized, without messing up your global installation. Depending on the requirements of what you're working on, you can use different versions of Python or keep older libraries on a per-project basis. Nothing outside the virtual environment will be modified.

<!-- more -->

# Installation

To begin with, install *virtualenv*. You can do it in several ways. Via pip: <code>pip install virtualenv</code>, or with easy_install: <code>easy_install virtualenv</code>. Now let's look at the help menu:

``` plain
root@kali:~# virtualenv -h
Usage: virtualenv [OPTIONS] DEST_DIR

Options:
  --version             show program's version number and exit
  -h, --help            show this help message and exit
  -v, --verbose         Increase verbosity.
  -q, --quiet           Decrease verbosity.
  -p PYTHON_EXE, --python=PYTHON_EXE
                        The Python interpreter to use, e.g.,
                        --python=python2.5 will use the python2.5 interpreter
                        to create the new environment.  The default is the
                        interpreter that virtualenv was installed with
                        (/usr/bin/python)
  --clear               Clear out the non-root install and start from scratch.
  --no-site-packages    DEPRECATED. Retained only for backward compatibility.
                        Not having access to global site-packages is now the
                        default behavior.
  --system-site-packages
                        Give the virtual environment access to the global
                        site-packages.
  --always-copy         Always copy files rather than symlinking.
  --unzip-setuptools    Unzip Setuptools when installing it.
  --relocatable         Make an EXISTING virtualenv environment relocatable.
                        This fixes up scripts and makes all .pth files
                        relative.
  --no-setuptools       Do not install setuptools in the new virtualenv.
  --no-pip              Do not install pip in the new virtualenv.
  --no-wheel            Do not install wheel in the new virtualenv.
  --extra-search-dir=DIR
                        Directory to look for setuptools/pip distributions in.
                        This option can be used multiple times.
  --download            Download preinstalled packages from PyPI.
  --no-download, --never-download
                        Do not download preinstalled packages from PyPI.
  --prompt=PROMPT       Provides an alternative prompt prefix for this
                        environment.
  --setuptools          DEPRECATED. Retained only for backward compatibility.
                        This option has no effect.
  --distribute          DEPRECATED. Retained only for backward compatibility.
                        This option has no effect.
```

# Usage

So you're about to begin a new project. You want to start from scratch and keep it contained to itself. Creating a virtual environment will help you accomplish that:

<code>virtualenv my_project</code>

Now you can go to the newly created directory and see that there are already some folders inside. <code>bin</code> has the executables, <code>include</code> holds the header files, and <code>lib</code> contains the files of the installed modules in the virtual environment.

Before you begin to work with your new environment, you need to activate it with <code>source bin/activate</code> or <code>source env_name/bin/activate</code> if you are outside the environment's directory.

You will now see your propmt change, to confirm that the environment is active. For me, it looks like:

``` plain
(my_project) root@kali:~/projects/my_project#
```

Now you can begin your work, installing packages, etc. When you're done, you can type <code>deactivate</code> to exit your environment. If you want to delete it, just remove its directory.

When working with virtual environments, it might be helpful to take a snapshot of your installed packages and their versions, in case you want to recreate the environment later. You can do this with <code>pip freeze > requirements.txt</code>. Check the newly created file for a list of your snapshotted items. In my case, I only installed requests in my environment so far:

``` plain
cat requirements.txt 
requests==2.10.0
```

If you need the exact same environment later, maybe to let some one of your team to work on the project, you can install the same packages and versions with the command <code>pip install -r requirements.txt</code>

## virtualenvwrapper

To make working with virtual environments even more convenient, you can use *virtualenvwrapper*, which provides wrappers for managing your environments and helps keep all your environments in one place. It also allows you to switch between environments with only one command, and has tab completion.

First, install it (outside your environments), with <code>pip install virtualenvwrapper</code>.

Then you need to add a few lines to your shell startup script. In my <code>~/.bashrc</code> file, I added the following:

``` plain
# set the location where your virtual environments will be stored
export WORKON_HOME=$HOME/.virtualenvs
# set the location of the virtualenvwrapper script (where it was installed)
source /usr/local/bin/virtualenvwrapper.sh
```

Now reload the startup file: <code>source ~/.bashrc</code> and you will see that your .virtualenvs folder has been created and populated with some scripts. Let's look at the help menu to see the available commands:

``` plain
virtualenvwrapper

virtualenvwrapper is a set of extensions to Ian Bicking's virtualenv
tool.  The extensions include wrappers for creating and deleting
virtual environments and otherwise managing your development workflow,
making it easier to work on more than one project at a time without
introducing conflicts in their dependencies.

For more information please refer to the documentation:

    http://virtualenvwrapper.readthedocs.org/en/latest/command_ref.html

Commands available:

  add2virtualenv: add directory to the import path

  allvirtualenv: run a command in all virtualenvs

  cdproject: change directory to the active project

  cdsitepackages: change to the site-packages directory

  cdvirtualenv: change to the $VIRTUAL_ENV directory

  cpvirtualenv: duplicate the named virtualenv to make a new one

  lssitepackages: list contents of the site-packages directory

  lsvirtualenv: list virtualenvs

  mkproject: create a new project directory and its associated virtualenv

  mktmpenv: create a temporary virtualenv

  mkvirtualenv: Create a new virtualenv in $WORKON_HOME

  rmvirtualenv: Remove a virtualenv

  setvirtualenvproject: associate a project directory with a virtualenv

  showvirtualenv: show details of a single virtualenv

  toggleglobalsitepackages: turn access to global site-packages on/off

  virtualenvwrapper: show this help message

  wipeenv: remove all packages installed in the current virtualenv

  workon: list or change working virtualenvs
```

Ok, let's create a new environment: <code>mkvirtualenv beta_proj</code>. You can list all your environments with <code>lsvirtualenv</code>

``` plain
lsvirtualenv
beta_proj
=========
```

Use the <code>workon</code> command to begin..working on your new environment: <code>workon beta_proj</code>. When finished, you can use <code>deactivate</code> again,or you can switch between multiple environments with <code>workon</code>.

Removing an environment can be done with the <code>rmvirtualenv</code> command

That's about all the basic tips needed to quick start your use of virtual environments in your Python projects. For more information, you can check the [virtualenv](https://virtualenv.pypa.io/en/stable/) and [virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/index.html) documentation.

``` plain
 _____________________________________
/ Do what comes naturally. Seethe and \
\ fume and throw a tantrum.           /
 -------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```


