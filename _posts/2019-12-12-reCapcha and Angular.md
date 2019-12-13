---
layout: post
author: Sanjeev
comments: true
categories: [reCaptcha]
tags: [reCaptcha, Angular]
identifier: reCaptcha
title: reCaptcha and Angular
---
There are many npm libraries offering integration with reCaptcha and Angular. I was not convinced using any of them. Mostly because quality, maintanance etc.. I decided to go through these libraries and reCaptch documentation and comeup with a simple solution on my own so that I can support it if any issues are raised.

Below are some of the issues to consider while workign with reCaptcha.

<ul>
    <li><b>Encapsulate functionality</b>: Most of the libraries got a directive which adds a script element to download the javascript file. A container element is added to place the reCaptcha. However I decided to include the javascript file once and have a div element as a container to host the reCaptcha.</li>
    <li><b>Explict render</b>: Chose to use explict render option instead of default render option. This is necessary due to following reasons
        <ol>
            <li>I maintain different site keys for different environments like dev, qa and prod. This cannot be hard coded into view. I fetch this key part of my initial configuration fetch from server. So UI rendering needs wait until the key is available at the client side</li>
            <li>In an SPA application, once an user is authenticated, page will be cleared there by loosing the container hosting the reCaptch. Whenever user revisits the same page, it is a new element being recreated. This calls for reloading / recreating the reCaptcha instance</li>
            <li>When reCaptcha want to notify events like <code>succss, errored, expired</code> by calling callback functions, it would be good to have provided a function reference being aware of angular context</li>
        </ol>
    </li>
    <li><b>Hanlde Expiration</b>: If entering captcha needs to be made mandatory, we need to ensure we always have a valid captcha. So I subscribe to the expiration notification to clear out the cached captcha. To do this, part of <code>window.grecaptcha.render</code> pass the callback function for expiry. Ex: <code>window.grecaptcha.render('..', {..., 'expired-callback': () => this.updateCaptchaStatus()});</code></li>
    <li><b>Captcha can only be verified once</b>: When a captcha response is verified using the google api, captcha becomes invalidated. In other words, any subsequent API calls for the same captcha response will always fail. To ensure that we always get new one call 'reset' on the reCaptcha instance Ex: <code>window.grecaptcha.reset('&lt;instance id&gt;')</code></li>
    <li>If site is protected with <code>Content Security Policy</code>, then consider adding below exceptions to the existing policy
        <ol>
            <li>font-src 'self' fonts.gstatic.com</li>
            <li>script-src 'self' https://www.google.com https://www.gstatic.com/</li>
            <li>frame-src https://www.google.com</li>
        </ol>
    </li>
</ul>
