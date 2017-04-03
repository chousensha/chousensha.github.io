---
layout: post
title: "Remote task automation with Fabric"
date: 2017-04-03 03:01:29 -0400
comments: true
categories: [python, sysadmin]
keywords: fabric, python, automation, fabric automation, python automation, remote task automation, automatic code deployment
description: Using Fabric to automate tasks on remote hosts
---



Today I will go over the use of [Fabric](http://docs.fabfile.org/en/1.13/), a Python library and CLI tool for executing local / remote tasks via SSH.

**Requirements**: 

- Python 2.5 or 2.7
- Paramiko
- SSH

**Features at a glance**:

- run local or remote shell commands (also with the sudo option)
- upload / download files
- prompt for input
- run Python functions
- abort execution when encountering errors by default, but also allow for error handling

Install Fabric with <code>pip install fabric</code>

<!-- more -->

## Usage 

The way Fabric works is by creating a file named <code>fabfile.py</code>, containing some functions, and then using the **fab** CLI tool to run them.

Let's look at the options available with the **fab** tool:

``` plain
fab --help
Usage: fab [options] <command>[:arg1,arg2=val2,host=foo,hosts='h1;h2',...] ...

Options:
  -h, --help            show this help message and exit
  -d NAME, --display=NAME
                        print detailed info about command NAME
  -F FORMAT, --list-format=FORMAT
                        formats --list, choices: short, normal, nested
  -I, --initial-password-prompt
                        Force password prompt up-front
  --initial-sudo-password-prompt
                        Force sudo password prompt up-front
  -l, --list            print list of possible commands and exit
  --set=KEY=VALUE,...   comma separated KEY=VALUE pairs to set Fab env vars
  --shortlist           alias for -F short --list
  -V, --version         show program's version number and exit
  -a, --no_agent        don't use the running SSH agent
  -A, --forward-agent   forward local agent to remote end
  --abort-on-prompts    abort instead of prompting (for password, host, etc)
  -c PATH, --config=PATH
                        specify location of config file to use
  --colorize-errors     Color error output
  -D, --disable-known-hosts
                        do not load user known_hosts file
  -e, --eagerly-disconnect
                        disconnect from hosts as soon as possible
  -f PATH, --fabfile=PATH
                        python module file to import, e.g. '../other.py'
  -g HOST, --gateway=HOST
                        gateway host to connect through
  --gss-auth            Use GSS-API authentication
  --gss-deleg           Delegate GSS-API client credentials or not
  --gss-kex             Perform GSS-API Key Exchange and user authentication
  --hide=LEVELS         comma-separated list of output levels to hide
  -H HOSTS, --hosts=HOSTS
                        comma-separated list of hosts to operate on
  -i PATH               path to SSH private key file. May be repeated.
  -k, --no-keys         don't load private key files from ~/.ssh/
  --keepalive=N         enables a keepalive every N seconds
  --linewise            print line-by-line instead of byte-by-byte
  -n M, --connection-attempts=M
                        make M attempts to connect before giving up
  --no-pty              do not use pseudo-terminal in run/sudo
  -p PASSWORD, --password=PASSWORD
                        password for use with authentication and/or sudo
  -P, --parallel        default to parallel execution method
  --port=PORT           SSH connection port
  -r, --reject-unknown-hosts
                        reject unknown hosts
  --sudo-password=SUDO_PASSWORD
                        password for use with sudo only
  --system-known-hosts=SYSTEM_KNOWN_HOSTS
                        load system known_hosts file before reading user
                        known_hosts
  -R ROLES, --roles=ROLES
                        comma-separated list of roles to operate on
  -s SHELL, --shell=SHELL
                        specify a new shell, defaults to '/bin/bash -l -c'
  --show=LEVELS         comma-separated list of output levels to show
  --skip-bad-hosts      skip over hosts that can't be reached
  --skip-unknown-tasks  skip over unknown tasks
  --ssh-config-path=PATH
                        Path to SSH config file
  -t N, --timeout=N     set connection timeout to N seconds
  -T N, --command-timeout=N
                        set remote command timeout to N seconds
  -u USER, --user=USER  username to use when connecting to remote hosts
  -w, --warn-only       warn, instead of abort, when commands fail
  -x HOSTS, --exclude-hosts=HOSTS
                        comma-separated list of hosts to exclude
  -z INT, --pool-size=INT
                        number of concurrent processes to use in parallel mode
```

The **fab** tool can execute functions from a fabfile.py by passing the function name, like so: <code>fab func-name</code>

## Examples

Let's take a look at a couple of simple fabfiles, before making a more contrived one:

### Run commands on the local host

``` python
from fabric.api import *

@task
def local_info():
        local('whoami')
```

Run it:

``` plain
fab local_info            
[localhost] local: whoami
root

Done.
```

### Run commands on remote host - password authentication

Next, let's use SSH with a password to log in to a remote host and do something on it:

``` python
from fabric.api import *

env.hosts = ['192.168.216.128']
env.user = 'root' # default is the local user
env.password = 'password' # can be used both for SSH authentication and sudo

@task
def remote_info():
        run('uname -a')
```

And the output:

``` plain
fab remote_info 
[192.168.216.128] Executing task 'remote_info'
[192.168.216.128] run: uname -a
[192.168.216.128] out: Linux localhost.localdomain 3.10.0-514.2.2.el7.x86_64 #1 SMP Tue Dec 6 23:06:41 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
[192.168.216.128] out: 


Done.
Disconnecting from 192.168.216.128... done.
```

### Run commands on remote host - key authentication

SSH with a password might be ok for demonstration purposes, but we will most likely want to use public key authentication. First, generate the key that we'll use later:

``` plain
ssh-keygen -t rsa -b 2048 -v
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): mykey.pem
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in mykey.pem.
Your public key has been saved in mykey.pem.pub.
```

Next, copy the key to the remote host's <code>~/.ssh/authorized_keys</code> location:

``` plain
ssh-copy-id -i ~/mykey.pem.pub root@192.168.216.128 
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/mykey.pem.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@192.168.216.128's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@192.168.216.128'"
and check to make sure that only the key(s) you wanted were added.
```

Make the .pem key read-only and then try logging in by passing it to the ssh -i option:

``` plain
chmod 400 mykey.pem
ssh -i mykey.pem root@192.168.216.128
Last failed login: Tue Mar 28 12:28:43 EEST 2017 from 192.168.216.1 on ssh:notty
There were 2 failed login attempts since the last successful login.
Last login: Tue Mar 28 11:30:19 2017 from 192.168.216.1
[root@localhost ~]#
```

Now add the key file in Fabric:

``` python
from fabric.api import *

env.hosts = ['192.168.217.131']
env.user = 'root'
env.key_filename = '~/.ssh/mykey.pem'

@task
def remote_info():
        run('uptime')
```

### Run commands on multiple hosts

If you want to run a command on a host, and a different command on another host, you can do so by specifying which host you want to run a command on:

``` python
from fabric.api import *

env.user = 'root'
env.key_filename = '~/mykey.pem'

@task
def local_info():
    print('This runs only on the localhost')

@hosts('192.168.216.128')
@task
def remote_info():
        print('This runs only on the remote host')
```

Here is how it would look like:

``` plain
fab local_info remote_info
This runs only on the localhost
[192.168.216.128] Executing task 'remote_info'
This runs only on the remote host

Done.
```

### Download / upload files

Fabric also has built-in functionality for downloading and uploading files:

``` python
@task
def download():
        get('~/Desktop/ssh.sh')
```

Result:

``` plain
fab download
[192.168.23.128] Executing task 'download'
[192.168.23.128] download: path/192.168.23.128/ssh.sh <- /root/Desktop/ssh.sh

Done.
Disconnecting from 192.168.23.128... done.
```

And the upload:

``` python
@task
def upload():
        put('file.txt', '~/Desktop')
```

Output:

``` plain
fab upload
[192.168.23.128] Executing task 'upload'
[192.168.23.128] put: file.txt -> /root/Desktop/file.txt

Done.
Disconnecting from 192.168.23.128... done.
```

### Run sudo commands

We saw some examples of using the root user, but Fabric allows running sudo commands from users with lower privileges:

``` python
from fabric.api import *

env.hosts = ['192.168.217.131']
env.user = 'nixhat'

@task
def cfg_test():
        sudo('apachectl configtest')
```

And here it is in action:

``` plain
fab cfg_test
[192.168.217.131] Executing task 'cfg_test'
[192.168.217.131] sudo: apachectl configtest
[192.168.217.131] Login password for 'nixhat':
[192.168.217.131] out: sudo password:
[192.168.217.131] out: AH00558: httpd: Could not reliably determine the server's
 fully qualified domain name, using localhost.localdomain. Set the 'ServerName'
directive globally to suppress this message
[192.168.217.131] out: Syntax OK
[192.168.217.131] out:


Done.
Disconnecting from 192.168.217.131... done.
```

### Specifying multiple hosts with different users and passwords

In case you have many hosts that you would like to run commands on as different users, you can do so:

``` python
env.hosts = ['root@192.168.217.130', 'nixhat@192.168.216.128'] # hosts to run cmds on
env.passwords = {'root@192.168.217.130':'password1', 'nixhat@192.168.216.128':'password2'}
```

### Running tasks on predefined hosts

If you have tasks that you want to run on certain hosts only, you can specify that:

``` python
@hosts('host1', 'host2')
def mytask():
    run('cmd')
```

### Classifying hosts into roles

In a large environment, you are most likely to have many hosts that fulfill some roles, such as development machines and production servers. You can organize them in the Fabric code:

``` python
env.roledefs = {
    'production': ['prod1', 'prod2'],
    'development': ['dev1', 'dev2']
}
```

### Automating code deployment

In a [previous post](https://chousensha.github.io/blog/2017/03/30/using-git-for-code-deployment/), we used Git for updating the code on a live server from a development machine. This time, we'll use Fabric to automate the process!

Besides the development and production servers we had before, we'll use a third computer to run the Fabric commands from. Let's assume this is your sysadmin machine, from which magic happens on the network. The developers made some changes to the code on the dev server, and you want to push them to production:

``` python
from fabric.api import *

# script must be called fabfile.py
# fab -l - list available cmds
# fab CMDNAME - run cmd

# the env dictionary allows for setting different variables for customizing the configuration
env.colorize_errors = 'true' # color errors in red and warnings in magenta



env.roledefs = {
    'dev': ['root@192.168.217.130:22 '],
    'prod': ['root@192.168.217.131:22'],
    }

env.passwords = {'root@192.168.217.130:22':'5t0rm80rn', 'root@192.168.217.131:22':'hakubo10'}


@roles('dev')
@task
def prep():
    '''
    Stage and commit new and modified files
    :return:
    '''
    with settings(warn_only=True): # continue executing even if errors were encountered
        with cd('/root/Testing/Dev'):
            run('git add . && git commit -m "Add changes"')

@roles('dev')
@task
def push():
    with cd('/root/Testing/Dev'):
        run('git push production')

@roles('prod')
@task
def chk():
    '''
    Confirm that files have been updated on the web server
    :return:
    '''
    run('ls /var/www/html/myweb/site')
```

Now list the available commands:

``` plain
fab -l
Available commands:

    chk   Confirm that files have been updated on the web server
    prep  Stage and commit new and modified files
    push
```

And run them:

``` plain
fab prep push chk
[root@192.168.217.130:22] Executing task 'prep'
[root@192.168.217.130:22] run: git add . && git commit -m "Add changes"
[root@192.168.217.130:22] out: [master 5719a02] Add changes
[root@192.168.217.130:22] out:  1 file changed, 1 insertion(+), 1 deletion(-)
[root@192.168.217.130:22] out:

[root@192.168.217.130:22] Executing task 'push'
[root@192.168.217.130:22] run: git push production
[root@192.168.217.130:22] out: root@192.168.217.131's password:
[root@192.168.217.130:22] out: Counting objects: 3, done.
[root@192.168.217.130:22] out: Compressing objects:  33% (1/3)
[root@192.168.217.130:22] out: Compressing objects:  66% (2/3)
[root@192.168.217.130:22] out: Compressing objects: 100% (3/3)
[root@192.168.217.130:22] out: Compressing objects: 100% (3/3), done.
[root@192.168.217.130:22] out: Writing objects:  33% (1/3)
[root@192.168.217.130:22] out: Writing objects:  66% (2/3)
[root@192.168.217.130:22] out: Writing objects: 100% (3/3)
[root@192.168.217.130:22] out: Writing objects: 100% (3/3), 294 bytes | 0 bytes/
s, done.
[root@192.168.217.130:22] out: Total 3 (delta 2), reused 0 (delta 0)
[root@192.168.217.130:22] out: To 192.168.217.131:Prod
[root@192.168.217.130:22] out:    39e598d..5719a02  master -> master
[root@192.168.217.130:22] out:

[root@192.168.217.131:22] Executing task 'chk'
[root@192.168.217.131:22] run: ls /var/www/html/myweb/site
[root@192.168.217.131:22] out: index.html  new.html  updates.html
[root@192.168.217.131:22] out:


Done.
Disconnecting from root@192.168.217.130... done.
Disconnecting from root@192.168.217.131... done.
```

In a file with many tasks, it will get tiresome to always specify them on the command line. You can organize your code to have a function that calls other tasks, respecting the roles and settings for each. Add this to the previous code:

``` python
@task
def deploy():
    '''
    Execute previous tasks by taking into account the defined roles 
    :return:
    '''
    execute(prep) # on dev
    execute (push) # on dev
    execute(chk) # on prod
```

Now you can run only <code>fab deploy</code> to get the same results!

``` plain
/ In the stairway of life, you'd best \
\ take the elevator.                  /
 -------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||

```


