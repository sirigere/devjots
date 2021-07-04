---
layout: post
author: Sanjeev
comments: true
categories: [Certification]
tags: [Certification, AZ-204]
identifier: az204Certification
title: Exam AZ-204 - Developing Solutions for Microsoft Azure
---
Planning on taking **MS Certification AZ-204: Developing Solutions for Microsoft Azure**. I will keep updating these notes during the course of my preparation for the certification. 

As this certification is for developers, it would be good idea to do labs part of the preparation. For some of the services we may need to write C# code. For almost all the other services, knowing how we can accomplish the task using ```Azure CLI```, ```Powershell``` and ```ARM Templates``` is recommended.


<ul>
{% for post in site.posts %}
	{% if post.identifier contains 'Az204-' %}
      		<li><a href="{{ post.url }}">{{ post.title }}</a></li>
	{% endif%}
{% endfor %}
</ul>

#### Reference material
* [Required Exam Skill](https://docs.microsoft.com/en-us/learn/certifications/exams/az-204)
* I liked the [study guide from Thomas Maurer](https://www.thomasmaurer.ch/2020/03/az-204-study-guide-developing-solutions-for-microsoft-azure/)
