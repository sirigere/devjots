---
layout: post
author: Sanjeev
comments: true
categories: [Personal]
tags: [Jekyll,Github,GithubPages]
identifier: GitPages
title: Blogging with Github Pages
---
This blog captures how this site is built. Some of my findings that could be of your interest too... 

* [Blog with a custom domain name](#host-the-blog-with-a-custom-domain)
* [Hosting the blog site](#identify-the-provider-to-host)
* [Site to support https](#site-to-support-https)
* [Have a simple mechanism for posting new blogs](#have-a-simple-mechanism-for-posting-new-blogs)

##### Host the blog with a custom domain
Went through many domain registrars and finalized with GoDaddy for purchasing this domain. Couple of reason to go with GoDaddy
* Attractive pricing
* Easy interface for managing the DNS records
* One of the known names in the industry


<div class="row">
<div class="col-3">
</div>
<div class="col-5">
<small class="text-info">Below are DNS entries @ GoDaddy. Click on image to enlarge. </small>
<a href="/assets/images/godaddy_dns.png">
<img src = "/assets/images/godaddy_dns.png" class="img-thumbnail">
</a>
</div>
<div class="col-4">
</div>
</div>

##### Identify the provider to host
I am hosting this blog site using Github Pages as it offered me below advantages
* Adding a blog post is as simple as commiting a markdown page and pushing it to Github.
* Github Pages can only serve static pages. Github Pages use Ruby based Jekyll static site generator. Whenever you checkin, CI/CD pipeline starts a Jekyll based build to generate the static pages for your site. Below are some of the notes related to Jekyll.
	* If you are a windows user like me, you would have to install Ruby on your machine to start with. This [page](https://jekyllrb.com/docs/installation/windows/) guides you with steps to follow to install both Ruby and Jekyll.
	* To build a site locally from command prompt, run command <code>jekyll build</code>
	* To build and run a site locally from command prompt, run command <code>jekyll serve</code>
	* This [page](https://jekyllrb.com/docs/step-by-step/01-setup/) provides step by step guide to get started on Jekyll
	* Configure DNS records to point to Githup Pages. Added A Record entries following the documentation from the [page](https://help.github.com/articles/setting-up-an-apex-domain/#configuring-a-records-with-your-dns-provider).
	* Configure Github Pages to use custom domain. Used the documentation from the [page](https://help.github.com/articles/adding-or-removing-a-custom-domain-for-your-github-pages-site/)
* This service is free
		
##### Site to support https
Out of the box, Github Pages work with [Let’s Encrypt](https://letsencrypt.org/) to provide free Domain Validation (DV) certificates. These cerificates are valid for 90 days. We should ask for a new certificates to renew the old one. This is automated through ACME protocol. Github does automatic renewal for you.

##### Have a simple mechanism for posting new blogs
As Github uses [Jekyll](https://jekyllrb.com). By default Jekyll is blog aware. With this, all I have to do is check in a new mark down file for posting a new blog. Refer to [Pages](https://jekyllrb.com/docs/posts/) for details