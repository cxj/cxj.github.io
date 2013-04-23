---
layout: post
title: Using Burp with SSL Web Service Client
---

Developing a web service client is sometimes easier when one can see the actual data being exchanged.  In this case, the service provided used SOAP and SSL, which made the picture a bit more comples.

I had a simple proxy which could nicely display SOAP messages, but it only spoke HTTP.  While the client could speak either HTTP or HTTPS, the server only allowed HTTPS connections.  I needed something to speak HTTP to my client on one side and HTTPS to the server.  Enter [Burp](http://portswigger.net/burp/).


