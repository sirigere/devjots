---
layout: post
author: Sanjeev
comments: true
categories: [reCAPTCHA]
tags: [reCAPTCHA, Angular]
identifier: reCAPTCHA
title: reCAPTCHA and Angular
---
There are many npm libraries offering integration with reCAPTCHA and Angular. I was not convinced using any of them. Mostly because of quality, maintanance etc.. Decided to go through these libraries and reCaptch documentation and comeup with a simple solution so as to support if any issues araise.

### What is a reCAPTCHA
reCAPTCHA is a service offered by Google. This is offered for free! Question to be answered here is to identify if the user is a human or a bot. Mostly used in the login pages. It as one of the strategy to mitigate Denial of Service attacks. Using Captcha makes attacker to work bit more.

### How does reCAPTCHA work
reCAPTCHA requires a user to click on a checkbox. When the checkbox is clicked, optionally user is asked to solve an image/audio based puzzle. Once the user completes the puzzle, a  unique identifier / token is returned to the page hosting reCAPTCHA. Now the site implementer can use this token and query the service provider to check if the user has solved the reCAPTCHA. If yes, then proceed with the functionality of the site.

### How to integrate
 * Step 1: Register with service provider.
 * Step 2: After registration, collect `Site Key` and `Secret` from the service provider.
 * Step 3: Service provider provide basic HTML template along with Javascript file which works with the HTML template. Depending on the  UI framework being used, implementor need to adapt the given template to work with the UI framework. `Site key` is used along with the HTML template.
 * Step 4: Create a mechanism to send the token provided by the service to the backend service.  
 * At the server sidem use the REST based API provided to validate if the token is valid or not. While making the call pass along `Secret` key.

### Points to consider while working with reCAPTCHA.
* **Reference documents**: 
  1. [Google's reCAPTCHA developer Guide](https://developers.google.com/reCAPTCHA/intro)
  2. [Google's admin console](https://www.google.com/reCAPTCHA/admin)
* **Encapsulate functionality**: Most of the libraries got a directive which adds a script element to download the javascript file. A container element is added to place the reCAPTCHA. However I decided to include the javascript file once and have a div element as a container to host the reCAPTCHA.
* **Explict render**: Chose to use explict render option instead of default render option. This is necessary due to following reasons
        
  1. It is better to maintain different "site keys" for different environments like dev, qa and prod etc... These keys are fetched part of initial load from server. So, UI rendering needs to wait until the "site key" is available at the client side
  2. In a SPA application, once a user is authenticated, page will be cleared there by loosing the container hosting the reCAPTCHA. Whenever user revisits the same page, container is a new element being recreated. This calls for reloading / recreating the reCAPTCHA instance
  3. When reCAPTCHA want to notify events like `success, errored, expired` by calling callback functions, it would be a good idea to have callback functions being aware of angular context
        
* **Hanlde Expiration**: If entering captcha needs to be made mandatory, we need to ensure that, captcha hold valid value. So, subscribe to the expiration notification to clear out the cached captcha. To do this, part of `window.greCAPTCHA.render` pass the callback function for expiry. Ex: `
window.greCAPTCHA.render('..', {..., 'expired-callback': () => this.updateCaptchaStatus()});`
* <b>Captcha can only be verified once</b>: When a captcha response is verified using the Google's api `https://www.google.com/reCAPTCHA/api/siteverify`, captcha becomes invalidated. In other words, any subsequent API calls with the same captcha response will fail. To ensure that we always get new captcha call 'reset' on the reCAPTCHA instance Ex: `window.greCAPTCHA.reset('&lt;instance id&gt;')`
* **Content Security Policy**: If site is protected with <a href="https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP">Content Security Policy</a>, then consider adding below exceptions to the existing policy
  1. font-src 'self' fonts.gstatic.com
  2. script-src 'self' https://www.google.com https://www.gstatic.com/
  3. frame-src https://www.google.com