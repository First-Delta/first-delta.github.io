---
title: XSS
layout: default
has_toc: false
parent: Web-Security
---

# XSS
Quick reference below for each type of XSS with an example.

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

## Stored XSS
Stored XSS can be used when an application receives data from an untrusted source which is then referenced in later HTTP responses. For example, an attacker posts a comment to a blog. When that blog data is then returned during a HTTP response, the malicious code witihin the comment will run.

```
POST /post/comments HTTP/1.1
Host: example.com
Content-Length: 100

post=1&comment=<script>alert(0)</script>&name=John&email=john@example.com
```

