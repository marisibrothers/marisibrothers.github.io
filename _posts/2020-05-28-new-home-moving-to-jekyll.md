---
layout: post
title: 'New home: Moving to Jekyll'
date: '2020-05-28'
author: luciano_marisi
tags: 
- blogging
permalink: /2020/05/new-home-moving-to-jekyll.html
reviewers:
- nahuel_marisi
---

We have a new home. The blog is now generated using [Jekyll](https://jekyllrb.com/) and hosted on [GitHub Pages](https://pages.github.com/). This article describes the thought process for this decision.

The blog had been hosted on Blogger since 2015. At the time it got us up and running quickly. Gradually we noticed several limitations but we stuck with it to focus on writing articles. 


### Reasons to leave Blogger

- Hard to modify theme and layout of the blog. For example adding a link the author's Twitter profile was a lengthy and cumbersome task.
- No support for code formatting, we had to copy/paste the HTML formatter code into Blogger.
- Lack of version control support, we could not use a git repository to provide content to the blog.


### Reasons for choosing GitHub Pages

- Low maintenance, GitHub takes all hosting responsibilities.
- Free for public pages.
- Git integration, an article can be pushed to the master once it's ready to make it live.


### Reasons for choosing Jekyll

- Integrates with GitHub Pages. When new changes are pushed, GitHub Pages runs Jekyll to generate the HTML and deploys it. 
- It can run on localhost, changes to the blog can be checked locally before making them available to everybody.
- Updating the layout of the site is straightforward.
- Configuration is in the repository, so it can be version controlled.
- Post's metadata, such as the author, is in one place. It	is defined within the post's markdown file in the [front matter](https://jekyllrb.com/docs/front-matter/).
- Active open source project. The platform is likely to exist in the foreseeable future.
- Many people use this tool, so there are many resources online to find answers to questions one might have.
- Supports markdown files.


### The downsides of Jekyll

- Debugging of templates and configuration is limited. Running Jekyll locally logs some errors when something is incorrect but many times it fails silently with one wondering why something not being displayed.
- Uses many conventions, such as where files should be placed so it's not obvious what is wrong when something does not load.
- Existing templates are limited and only provide a starting point. As far as I know it's not possible to combine templates.


### Decisions made during migration

- Use a minimalist theme to make the blog simpler and more mobile friendly to focus on the content. 
- Drop features such as tag specific pages in order to release sooner. I'm confident I can rebuild any missing relevant features fairly quickly.
- Drop ability to post comments since most of them were unrelated spam messages.
- Remove some unrelated images on the articles to reduce noise.


## Conclusion

Even though I've only been using Jekyll for a few days, I've already seen the benefits compared to Blogger when it comes to customisation. If you need more control over the layout of your blog, Jekyll is a great option.

*I’d like to thank [Nahuel Marisi](http://twitter.com/nmarisi) for reviewing this article.*