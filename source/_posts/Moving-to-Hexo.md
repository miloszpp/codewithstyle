---
title: Moving to Hexo
date: 2018-12-12 12:26:31
tags:
  - blogging
  - hexo
icon: fas fa-book
image: /images/posts/diary.jpg
---

Recently I decided to move my blog to [Hexo](https://hexo.io/).

I've been considering this idea for some time (shout out to [Piecioshka](https://twitter.com/piecioshka) whose [article [PL]](https://piecioshka.pl/blog/2018/05/28/jak-zalozyc-bloga-korzystajac-z-hexo.html) planted the seeds). Previously, my blog used to be based on Wordpress (to which I migrated from Blogger). Worpdress is a great platform that powers a lot of blogs and businesses but with time, more and more issues have been bugging me:

* **Performance** - with Wordpress every time you visit a post page, it has to be generated on the server. I tried to address this with caching plugins but with varying level of success.
* **Page size** - Wordpress has a plugin for everything. These plugins are very useful but it's super easy to bloat the page size with them. What's more, you're not in full control of what assets are loaded on the page.
* **Updates** - for security purposes Wordpress (and plugins) should be regularly updated. Every time an update is made, there is a risk of breaking the page.
* **Lack of flexibility** - anything custom in Wordpress has to either be added as a plugin or by manually modifying the source code. The source code is written in PHP and the only way I had to modify it was via FTP, no source control. 

Hexo is a static site generator. It is based on the simple but clever idea that a blog doesn't actually have to be dynamic. **Why generate posts' HTML over and over again while they look the same to every user**? 

With Hexo you write blog posts in markdown and the blog gets generated as a bundle of static HTML files. You can then very easily upload this bundle to [Github Pages](https://pages.github.com/) which will host your static files for free! 

Although I had a lot of content to migrate, the process was made much easier thanks to `hexo-migrator-wordpress` package.

With this setup, all of the issues I had with Wordpress are now addressed!

Please let me know how do you like the new look of the blog! I would also be grateful for an **e-mail if you find something that doesn't work** properly.