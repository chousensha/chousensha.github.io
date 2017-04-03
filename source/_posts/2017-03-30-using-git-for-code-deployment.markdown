---
layout: post
title: "Using Git for code deployment"
date: 2017-03-30 07:09:46 -0400
comments: true
categories: [sysadmin]
keywords: git, deployment, git deploy, code deployment, git code deployment, git web development, git development, git production
description: Pushing code from development to production with Git
---

Imagine you have a local server where you work on developing the code for a web application. After doing your magic, you want to upload it to the web server so the rest of the world can access your shiny new web app. To do that, you have to place your code on another server, which serves as the production environment. Moreover, when making changes to your code, you only want to update the modified files, not everything.

<!-- more -->

You could keep track of what files have been changed on your dev server, and then manually transfer them to the web server in production. Now that we have established how much time and sanity you would lose, especially with a big app, let's see how Git can make this kind of code deployment not only possible, but also smooth and efficient.

I will be using the root account for this demo, which is of course a bad idea in a live environment, so keep that in mind =D

On the server side, say we have a folder named Prod. Here we will push the code for a website, and then we'll transfer it to the web directory.

First, initialize a bare repository inside the Prod folder: 

``` plain
git init --bare
Initialized empty Git repository in /root/Prod/
```

For better security, we'll keep the git repository outside the web root directory. To automatically update the code, we'll use a git hook, which is a script that runs when a certain event occurs. In this case, we'll create a <code>post-receive</code> hook, which runs on a remote repository after a git push

In the hooks directory, I created a hook called post-receive, with the following code in it:

``` plain
#!/bin/sh
GIT_WORK_TREE=/var/www/html/myweb/site git checkout -f # path to the web root, it has to be created beforehand
```

Looking inside the hooks directory, you will see the samples are executable. We also have to make ours executable, so use <code>chmod +x</code>

With that, the production server setup is complete. On the local development server, make a repository and add some files that you want to end up in production:

``` plain
root@pwnbox:~/Testing/Dev#git add .
root@pwnbox:~/Testing/Dev#git status
On branch master

Initial commit

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

	new file:   index.html
	new file:   updates.html

```

Now add a remote, which will point to the production repository:

``` plain
git remote add production root@192.168.217.131:Prod
```

Make a commit:

``` plain
git commit -m '1st commit'
[master (root-commit) bac80ea] 1st commit
 2 files changed, 2 insertions(+)
 create mode 100644 index.html
 create mode 100644 updates.html
```

And now push the changes to production -- note the use of the [refspec](https://www.git-scm.com/book/tr/v2/Git-Internals-The-Refspec):

``` plain
git push production +master:refs/heads/master
root@192.168.217.131's password: 
Counting objects: 4, done.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (4/4), 331 bytes | 0 bytes/s, done.
Total 4 (delta 0), reused 0 (delta 0)
To 192.168.217.131:Prod
 * [new branch]      master -> master
```

Let's see if it worked! On my production server:

{% img center /images/sysadmin/git1.png 'Code updated' 'Code was pushed to web server' %}

{% img center /images/sysadmin/git2.png 'Code updated' 'Code was pushed to web server' %}

It did work! Now let's update a file, and only push this file which has been modified. After making the modifications, look at the current status:

``` plain
git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   updates.html

no changes added to commit (use "git add" and/or "git commit -a")
```

Commit the modified file:

``` plain
git commit -a -m 'Update to 1 file'
```

And now push to production and also set the remote branch so next time you can do it with a simple <code>git push production</code>:

``` plain
git push --set-upstream production master
root@192.168.217.131's password: 
Counting objects: 3, done.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 345 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To 192.168.217.131:Prod
   bac80ea..6fcdb86  master -> master
Branch master set up to track remote branch master from production.
```

{% img center /images/sysadmin/git3.png 'Changed file uploaded' 'The modified file was successfully updated' %}

And to confirm that on the production environment, only the modified file was updated, but the unchanged one remained the same:

``` plain
[root@localhost site]# ls -la
total 8
drwxr-xr-x. 2 root root 42 Mar 30 14:03 .
drwxr-xr-x. 3 root root 76 Mar 30 13:30 ..
-rw-r--r--. 1 root root 26 Mar 30 13:21 index.html
-rw-r--r--. 1 root root 71 Mar 30 14:03 updates.html
```

It might not look like much for the 2 small files I have here, but in a huge code environment, you will feel the difference! 

``` plain
 _________________________________________
/ Your boss climbed the corporate ladder, \
\ wrong by wrong.                         /
 -----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||

```





