---
layout: post
title: Burp as a Web Service Proxy
---

Building a web service client can be easier when one can see the actual data    being exchanged.  If the web service uses SSL, this can be difficult.

There are a variety of proxies one can use to view HTTP message exchanges but few support SSL.  Enter [Burp](http://portswigger.net/burp/).

1.  Obtain Burp.
2.  Launch Burp (e.g. java -jar burpsuite.jar)

    By default, it listens on localhost (127.0.0.1) port 8080.

3.  Configure settings.  For this purpose, these are the basics:
    1. Proxy -> Intercept:  Intercept is on
    2. Proxy -> Options:
        1.  Click listener listed to highlight, then Edit button to change.
        2.  If you want a different IP address or port, change them on the Binding tab.
        3.  On the Request handling tab, enable invisible proxy support (most non-web browser clients will not be proxy aware).
        4.  Enter the HTTPS server address in the Redirect box.
        5.  Enable Force use of SSL.
