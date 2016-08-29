---
layout: post
title: "Cloning Octopress blog"
date: 2015-11-26 11:16:32 -0500
comments: true
categories: [octopress]
keywords: octopress, blog, cloning octopress, octopress clone, clone octopress
description: Cloning Octopress blog
---


I've recently had some problems with Octopress breaking on my old Kali 1.0 box. And since the Kali 1.0 reached its end of life, figured this might be a good time to jump ship and install the 2.0 version, and set up a fresh Octopress there. So in this post I will quickly overview the steps needed to clone an already existing Octopress blog on a new machine and resume blogging from there.
<!-- more -->

#### clone the source branch repository
git clone -b source https://github.com/chousensha/chousensha.github.io.git octopress

cd octopress

#### clone master branch to _deploy
git clone https://github.com/chousensha/chousensha.github.io.git _deploy

The <code>source</code> branch contains the source of your blog, while the content that you generate is in the <code>master</code> branch.

Before proceeding with installing dependencies, I followed the instructions on http://octopress.org/docs/setup/ to install rvm, Ruby 1.9.3 and ExecJs:

#### install rvm
command curl -sSL https://rvm.io/mpapis.asc | gpg2 --import -
curl -L https://get.rvm.io | bash -s stable --ruby

#### run below to be able to use rvm in the shell windows, and add it to your shell rc file or you will need to always run it before using rvm
source /usr/local/rvm/scripts/rvm

#### install Ruby 1.9.3
rvm install 1.9.3

You will get a message that this version of Ruby is no longer maintained, but I had problems with Ruby 2~ so keeping this for Octopress.

#### use Ruby 1.9.3
rvm use 1.9.3

#### update the rubygems to the latest available
rvm rubygems latest

#### optional: check that your Ruby version is the right one
ruby --version

#### install JS runtime for Ruby
gem install execjs

#### now install dependencies
gem install bundler
bundle install

#### setup github pages 
bundle exec rake setup_github_pages

And you're good to go!

Note for *zsh* users: because of the globbing of *zsh*, you will need to add <code>alias rake="noglob rake"</code> to your .zshrc file or you will get a no matches found error. Or you can quote the arguments given to rake: <code>rake "new_post[Whatever]"</code>

``` plain
 _______________________________________
/ Few things are harder to put up with  \
| than the annoyance of a good example. |
|                                       |
| -- "Mark Twain, Pudd'nhead Wilson's   |
\ Calendar"                             /
 ---------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```






