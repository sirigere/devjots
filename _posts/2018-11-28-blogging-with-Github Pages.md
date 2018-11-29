---
layout: post
author: Sanjeev
comments: true
categories: [Personal]
tags: [Jekyll,Github,GithubPages]
---
This blog captures how this site is built. Some of my findings that could be of your interest too... 

* Blog with a custom domain name
* Hosting the blog site
* Site to support https
* Have a simple mechanism for posting new blogs

##### Host the blog with a custom domain
Went through many domain registrars and finalized with GoDaddy for purchasing this domain. Couple of reason to go with GoDaddy
* Attractive pricing
* Easy interface for managing the DNS records
* One of the known names in the industry

##### Identify the provider to host
This took a little while to decide on. There are many options out there. I decided to host this blog site using Github Pages as it offered me below advantages
* Adding a blog post is as simple as commiting a markdown page and pushing it to Github
* I do not have any private content to protect against
* Content in the site is static and Github can only host static content. This also meant I now need to learn new tools like
	* Jekyll: Jekyll is one of the most used static site generator. They have excellent [tutorial](https://jekyllrb.com/docs/step-by-step) for onboarding. Jekyll is based on Ruby. As a windows user, you can follow the [steps](https://jekyllrb.com/docs/installation/windows/) to install Ruby.
	* Configuring custom domain: This involve updating DNS records @ GoDaddy as well as at Github Pages. Github Pages got excellent [documentation](https://help.github.com/categories/customizing-github-pages/) for the same.
* Github Pages is a free offering
		
##### Site to support https
Out of the box, Github Pages work with [Let’s Encrypt](https://letsencrypt.org/) to provide free Domain Validation (DV) certificates. These cerificates are valid for 90 days. We should ask for a new certificates to renew the old one. This is automated through ACME protocol. Github does automatic renewal for you.

##### Have a simple mechanism for posting new blogs
As Github uses [Jekyll](https://jekyllrb.com) all I have to do is check in a new mark down file for posting a new blog.