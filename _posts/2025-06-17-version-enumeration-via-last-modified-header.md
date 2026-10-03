---
layout: post
title: "Version Enumeration via Last-Modified Header"
author: "Andres Rauschecker"
tags: [Pentesting, "Version Disclosure", "Last-Modified", Cache]
excerpt_separator: <!--more-->
---

When a target does not disclose its software version directly, cache metadata can provide a useful clue. Static files are often served with a `Last-Modified` response header that reveals when the file was deployed.<!--more-->

## Last-Modified

Modern web servers commonly cache JavaScript, CSS, and image files. The `Last-Modified` header tells clients when the origin believes a resource was last changed. See MDN's [Last-Modified header reference](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Last-Modified).

Look for a static file belonging to the software under investigation and inspect its `Last-Modified` response header.

### Example: Tiki Wiki CMS

I tried to find the Tiki Wiki version through generator metadata, source comments, a `?v` parameter, and other techniques without success. Then I noticed that the target returned an unusually old `Last-Modified` value:

![Last-Modified timestamp revealing the CMS deployment date]( /assets/images/posts/version-enumeration-via-last-modified-header/image.png)

Using Google's `after:YYYY-MM-DD` search operator, I searched specifically for vulnerabilities and exploits published after that deployment date:

![Searching for exploits published after the deployment date]( /assets/images/posts/version-enumeration-via-last-modified-header/image-2.png)

The target turned out to be vulnerable to the matching exploit. Alternatively, Google's custom date-range tool can help identify release notes and estimate a software version compatible with the `Last-Modified` date:

![Using a custom date range to identify the CMS version]( /assets/images/posts/version-enumeration-via-last-modified-header/image-3.png)

Happy hacking ;)
