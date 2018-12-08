---
layout: post
author: Sanjeev
comments: true
categories: [Git]
tags: [Git, Commit Comments]
identifier: GitCommitComments
title: Follow Git commit comments convention
---
Many of us may not be aware that, there are convention being followed for comments while commiting our changes to source code repositories. This is one thing I got to learn while commiting to an open source project.

Open source projects follow conventions that are well accepted by the industry. Most of the open source projects specify a convention to follow while you are commiting your changes to the repository. Here is a commit convention documentation for [AngularJs](https://docs.google.com/document/d/1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y/edit#). This document has become a standard for many libraries built for Angular. Example [ng-lightning](https://github.com/ng-lightning/ng-lightning/commits/master),  [ng-bootstrap](https://github.com/ng-bootstrap/ng-bootstrap/commits/master).

These conventins evolved from the Git's recommendation and command support. [Here is what Git has to say w.r.t commit comment](https://mirrors.edge.kernel.org/pub/software/scm/git/docs/git-commit.html#_discussion) when you checkin.
<p><code>
Though not required, it’s a good idea to begin the commit message with a single short (less than 50 character) line summarizing the change, followed by a blank line and then a more thorough description. The text up to the first blank line in a commit message is treated as the commit title, and that title is used throughout Git. For example, git-format-patch(1) turns a commit into email, and it uses the title on the Subject line and the rest of the commit in the body
</code></p>

These comments can be effectivly used if one takes a good look at Git commands like <kbd>git log</kbd>, <kbd>git shortlog</kbd>

There are many blog posts which discusses in details on this topic. So not gone in detals to explain those in here..