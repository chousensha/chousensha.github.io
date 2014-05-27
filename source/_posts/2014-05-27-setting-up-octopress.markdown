---
layout: post
title: "Setting up Octopress"
date: 2014-05-27 21:44:43 +0300
comments: true
categories: [octopress, blog]
---


Since I now got the blog up and running, I want to walk through the steps of setting it up and where I had to do some extra work not mentioned in the official documentation, which by the way, is a good place to start reading about Octopress.
<!-- more -->

First, I want to mention that starting an Octopress blog is not a daunting task at all. You don't need to know Ruby, the main commands for getting it to work are already implemented for you. The files you need to modify are intuitive and structured in a logical way, and the <code>markdown</code> language is easy to use even if you never heard of it before ( I certainly hadn't before starting with Octopress ). Some basic familiarity with a command shell and Git is all that's really needed! So if you want the awesome looks and other perks of Octopress, it's not hard at all to get them!

So, Octopress is built on top of Jekyll, which is a simple, blog-aware, static site generator based on Ruby. Jekyll is actually the engine behind GitHub Pages. Being static, it's very fast and no myriad features that you don't really need are cluttering up the server side. And Octopress gives you the necessary tools to customize and maintain a blog in a flexible and convenient way. It also comes loaded with plugins for many 3rd party services that you might want to use.

### Installation

You need to have Git and Ruby 1.9.3 or greater before installing Octopress. You can check your Ruby version with this command: <code>ruby --version</code>

To start with, clone the Octopress repository and give it the name you want:
``` plain
git clone git://github.com/imathis/octopress.git blogname
```

Then, install dependencies. I had to install the <code>ruby1.9.1-dev</code> package before the *bundle install* would work.
``` plain
gem install bundler
bundle install
```

Next, install the default Octopress theme:
``` plain
rake install
```

### Configuration

To configure your blog, you need to edit the <code>_config.yml</code> file. There you can add your blog URL, description categories, 3rd party integration, and other stuff.

### Content

To create a post, you run the following rake task:
``` plain
rake new_post["title"]
```

This will create a <code>markdown</code> post in the <code>source/_posts</code> directory. Use your favorite text editor to edit it and add your post body. Markdown is really easy to use, just look at one of reference sheets available on the web to get started with it.

Similarly, if you want to create a new page, you do it like this:
``` plain
rake new_page[page]
```

To see how your posts look:
``` plain
rake preview
```
This will start a small server on port 4000 that will let you view your pages on your local machine. Check it out in your browser by navigating to <code>http://localhost:4000/</code>

### Deployment

To generate your blog (populate the public directory with your posts and pages) run:
``` plain
rake generate
```

Now you are ready to deploy it. I use Github Pages for the blog. If you want it hosted on Github, there's a rake task available for it, but first you need to create a new repository that should look like this: <code>username.github.io</code>

Then run:
``` plain
rake setup_github_pages
```

You will be asked for the URL of your newly created repository, which you can copy from the HTTPS or SSH URL available on its page. 

After that's done, you can deploy your generated files with:
``` plain
rake deploy
```

This will commit your files and push them to the master branch. You will want to commit your sorce, too. In your blog directory, run these git commands:
``` plain
git add .
git commit -m 'message here'
git push origin source
```

And now you have an awesome Octopress blog out there on the web!

#### Disqus comments

To enable comments, in your _config.yml file, look for these lines and change them accordingly:

``` plain
disqus_short_name: your-disqus-shortname # your website shortname on disqus 
disqus_show_comment_count: true
```

When setting up your site on Disqus, be careful with the URL you give it. Pass it like this:

*mysite.com*

instead of this:

*mysite.com/*

Because Disqus adds a trailing slash after your URL, if you also provide one, you will end up with 2 slashes and Disqus won't work.

And that's pretty much what I had to do for this blog! 

Not forgetting the cookie:

> You will give someone a piece of your mind, which you can ill afford.

