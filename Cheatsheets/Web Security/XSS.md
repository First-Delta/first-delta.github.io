---
title: XSS
layout: default
has_toc: false
parent: Web Security
has_toc: false
---

# XSS
{: .no_toc }
Quick reference below for each type of XSS with an example.

---

- TOC
{:toc}
---

## Reflected XSS
Reflected XSS can be used when data is received in a HTTP request and the data is included within the response in an unsafe way e.g. no input validation or sanitisation.

```
https://example.com/search?item=<script>alert(0)</script>  
```  

{: .highlight-title }
> NOTE:
>
> You _may_ need to encode the term if entering manually in the URL as apposed to a search box

<span class="fs-2">
[PortSwigger Reference](https://portswigger.net/web-security/cross-site-scripting/reflected){: .btn .btn-purple }
</span>

---

## Stored XSS
Stored XSS can be used when an application receives data from an untrusted source which is then referenced in later HTTP responses. For example, an attacker posts a comment to a blog. When that blog data is then returned during a HTTP response, the malicious code witihin the comment will run.

```
POST /post/comments HTTP/1.1
Host: example.com
Content-Length: 100

post=1&comment=<script>alert(0)</script>&name=John&email=john@example.com
```

<span class="fs-2">
[PortSwigger Reference](https://portswigger.net/web-security/cross-site-scripting/stored){: .btn .btn-purple }
</span>

---

## DOM-Based XSS
Data that can be added into JavaScript can be utilised by an attacker when this data is passed to a sink that supports dynamic code execution.

The `document.write` sink allows `<script>` elements:

```document.write('... <script>alert(0)</script> ...');```

Some sinks like `innerHTML` don't accept `<script>` elements. As such, use an alternative element like `img` or `iframe` with a supported event handler such as `onload` or `onerror`.

```element.innerHTML='... <img src=1 onerror=alert(0)> ...'```

### Example of vulnerable JavaScript

```javascript
function trackSearch(query) {
    document.write('<img src="/resources/images/tracker.gif?searchTerms=' + query + '">');
}
var query = (new URLSearchParams(window.location.search)).get('search');
if (query) {
    trackSearch(query);
}
```

A function is declared that uses the `document.write` sink to write an `img` element including the `query` variable.

The `query` variables is defined by fetching the `search` parameter from the URL. 

Coincidentally, in this example, the `query` variable will be set to `true` even if no search term is defined. Thus triggering the run of the `trackSearch` function even if you search with an empty string. This is not specific to a DOM-Based XSS, just an error in the JavaScript logic. 

A simple `if` statement then checks if the variable has been set which then runs the `trackSearch` with the parameter `query`.

As an attacker, we can manipulate the string used in the URL that gets passed to the variable in order to exploit the vulnerable code.

### Example exploit
In this example, the following can be used as a search term in order to exploit a DOM-Based XSS:

```
" onload="alert(0)
```

Using this as the query in the search box or amending it to the end of `.../?search=` in the URL will cause the function to write the `img` tag as follows:

```html
<img src="/resources/images/tracker.gif?searchTerms=" onload="alert(0)">
```

<span class="fs-2">
[PortSwigger Reference](https://portswigger.net/web-security/cross-site-scripting/dom-based){: .btn .btn-purple }
</span>

### Other DOM-Based exploits
In the below example, jQuery is being used to udpate the reference `backLink` which uses a `href` attribute.

```javascript
<a id="backLink" href="javascript:alert(0)">Back</a>
/.../
$(function () {
    $('#backLink').attr("href", (new URLSearchParams(window.location.search)).get('returnPath'));
});
```
As the return path is set from the URL, an attacker can modify the `href` data directly from the URL.

Because the `href` is now set to the attackers input, when the button is clicked, the malicious code will run. In this example, an alert box.

<span class="fs-2">
[PortSwigger Reference](https://portswigger.net/web-security/cross-site-scripting/dom-based#dom-xss-in-jquery){: .btn .btn-purple }
</span>