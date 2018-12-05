---
layout: post
author: Sanjeev
comments: true
categories: [Personal]
tags: [Jekyll,Github,GithubPages]
identifier: GitPages
title: Blogging with Github Pages
---
This blog captures my jots on how I built this site. 

* [Configure Godaddy DNS entries](#configure-godaddy-dns-entries)
* [Configure Github Pages for custom domain](#configure-github-pages-for-custom-domain)
* [Site to support https](#site-to-support-https)
* [Have a simple mechanism for posting new blogs](#have-a-simple-mechanism-for-posting-new-blogs)

##### Configure Godaddy DNS entries
Once the custom domain is purchased, Navigate to the panel which allows to update the DNS entries for the domain. In this page, we need to add A record to point to our site or main domain. Optionally we can add CNAME record to point to www sub domain too. In the below screenshot, you would see both A and CNAME records. Refere [documentaion](https://help.github.com/articles/setting-up-an-apex-domain/#configuring-a-records-with-your-dns-provider) @ Github Pages for any updated information.


<div class="row">
<div class="col-3">
</div>
<div class="col-5">
<small class="text-info">Below are DNS entries @ GoDaddy. Click on image to enlarge. </small>
<a href="/assets/images/godaddy_dns.png">
<img src = "/assets/images/gitpages/godaddy_dns.png" class="img-thumbnail">
</a>
</div>
<div class="col-4">
</div>
</div>

##### Configure Github Pages for custom domain
+ Assuming that you already have a Github login, create a repository where the site is going to be hosted. 
+ Create a text file with name CNAME having your custom domain name within it and push the changes to the repository. Below is the screenshot of my CNAME file. Bwlow is the screenshot of CNAME file
<div class="row"><div class="col-3"></div><div class="col-5">
<small class="text-info">Below is the content of CNAME. Click on image to enlarge. </small>
<a href="/assets/images/cname.png">
<img src = "/assets/images/gitpages/cname.png" class="img-thumbnail">
</a>
</div><div class="col-4"></div></div>
+ Navigate to the repository > Settings > GitHub Pages section.
	- Update Custom domain with the name of your custom domain. Below is the screehshot.
<div class="row"><div class="col-3"></div><div class="col-5">
<small class="text-info">Section Githup Pages. Click on image to enlarge. </small>
<a href="/assets/images/custom_domain.png">
<img src = "/assets/images/gitpages/custom_domain.png" class="img-thumbnail">
</a>
</div><div class="col-4"></div></div>

Refer to [documentation](https://help.github.com/articles/adding-or-removing-a-custom-domain-for-your-github-pages-site/) for more details.
		
##### Site to support https
Out of the box, Github Pages work with [Let’s Encrypt](https://letsencrypt.org/) to provide free Domain Validation (DV) certificates. These cerificates are valid for 90 days. We should ask for a new certificates to renew the old one. This is automated through ACME protocol. Github does automatic renewal for you.

##### Have a simple mechanism for posting new blogs

As Github uses [Jekyll](https://jekyllrb.com). By default Jekyll is blog aware. With this, all I have to do is check in a new mark down file for posting a new blog. Refer to [Pages](https://jekyllrb.com/docs/posts/) for details.
* Adding a blog post is as simple as commiting a markdown page and pushing it to Github.
* Github Pages can only serve static pages. Github Pages use Ruby based Jekyll static site generator. Whenever you checkin, CI/CD pipeline starts a Jekyll based build to generate the static pages for your site. Below are some of the notes related to Jekyll.
	* If you are a windows user like me, you would have to install Ruby on your machine to start with. This [page](https://jekyllrb.com/docs/installation/windows/) guides you with steps to follow to install both Ruby and Jekyll.
	* To build a site locally from command prompt, run command <code>jekyll build</code>
	* To build and run a site locally from command prompt, run command <code>jekyll serve</code>
	* This [page](https://jekyllrb.com/docs/step-by-step/01-setup/) provides step by step guide to get started on Jekyll
* This site overrides the Jekyll theme to use [Bootstrap](https://getbootstrap.com).