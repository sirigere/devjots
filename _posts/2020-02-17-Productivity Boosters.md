---
layout: post
author: Sanjeev
comments: true
categories: [General]
tags: [NodeJS, Express]
identifier: ProductivityBoosters
title: Productivity Boosters
---
Below are some of tips/tricks that I use regularly while working on web projects

<div class="card">
    <div class="card-header bg-dark text-white">Serve static content from local folder</div>
  <div class="card-body">
    <h5 class="card-title"></h5>
    <p class="card-text">
	
	If you are a web developer, to tryout a quick sample, you would like to create a web site using the static sample content obtained from some source. Copy past the below content into an <kbd>server.js</kbd> file and excute the commmand <kbd>npm install express</kbd> and <kbd>node server.js</kbd>
	<pre><code class="javascript">
		var express = require('express');
		var app = express();

		app.use(express.static(__dirname + '/'));

		app.listen('3000');
		console.log('working on 3000');
	</code></pre>
	</p>
  </div>
</div>
<br/>

<script>hljs.initHighlightingOnLoad();</script>