---
layout: post
author: Sanjeev
comments: true
categories: [General]
tags: [NodeJS, Express]
identifier: ProductivityBoosters
title: Productivity Boosters
---

<div class="card">
    <div class="card-header bg-dark text-white">Serve static content from local folder</div>
  <div class="card-body">
    <h5 class="card-title"></h5>
    <p class="card-text">
	
	If you are web developer, to tryout a quick sample, you would like to create a web site using the static sample content obtained from some source. Copy past the below content into an `server.js` file and excute the commmand `npm install express` and `node server.js` <br\>
	```javascript
		var express = require('express');
		var app = express();

		app.use(express.static(__dirname + '/'));

		app.listen('3000');
		console.log('working on 3000');
	```
	</p>
  </div>
</div>
<br/>