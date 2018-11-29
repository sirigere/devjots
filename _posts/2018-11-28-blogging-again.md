---
layout: post
author: Sanjeev
comments: true
categories: [Personal]
tags: [Jekyll,Github,GithubPages]
---
I have been busy procastinating about starting my blog. There were few challenges I had to cross before I could comfortably start blogging. Many out there might have similar challenges to cross before they start blogging! I thought, I should blog about it if that helps someone like me to cross those challenges fast. Below were few of my challenges... 

* Blog with a custom domain name
* Hosting the blog site
* Site to support https
* Have a simple mechanism for posting new blogs

##### Host the blog with a custom domain
* Identify name for your custom domain. This is a very simple task! but it took a while to decide on.
* 
I went through many domain registrar and finalized with GoDaddy for purchasing this domain. Couple of reason to go with GoDaddy
	* Attractive pricing
	* Easy interface for managing the DNS records
	* One of the known names in the industry

##### Identify the provider to host
This took a little while to decide on. There are many options out there. I decided to host this blog site using Github Pages as it offered me below advantages
* Adding a blog post is as simple as commiting a markdown page and pushing it to Github
* I do not have any private content to protect against
* Most of the content in the site is static and Github can only host static content. This also meant I now need to learn new tools like
	* Jekyll: They have excellent [tutorial](https://jekyllrb.com/docs/step-by-step) for onboarding. Jekyll is based on Ruby. As a windows user, you can follow the [steps](https://jekyllrb.com/docs/installation/windows/) to install Ruby. 
	* Configuring custom domain: Github Pages has excellent [documentation](https://help.github.com/categories/customizing-github-pages/) for the same.
		
##### Site to support https
Out of the box, Github Pages work with [Let’s Encrypt](https://letsencrypt.org/) to provide free Domain Validation (DV) certificates. These cerificates are valid for 90 days. We should ask for a new certificates to renew the old one. This is automated through ACME protocol. Github does automatic renewal for you.

##### Have a simple mechanism for posting new blogs
As Github uses [Jekyll](https://jekyllrb.com) all I have to do is check in a new mark down file for posting a new blog.

-----
Do let me know if you have any clarification/suggestion/details on this post. I will be happy to know your feedback!