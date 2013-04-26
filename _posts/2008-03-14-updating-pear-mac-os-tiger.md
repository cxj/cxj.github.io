---
layout: post
title: Updating PEAR on Mac OS X Tiger
---

Do you have a Mac still running Tiger (Mac OS 10.4)? And did you recently decide to install a PEAR component for use with some PHP? And lastly, did it fail with some cryptic HTTP error "410 Gone" that you had very little luck finding with Google? I did.

The solution is a bunch of steps backward and then forward again. Unfortunately, it's not very clear to the perhaps large number of Mac users who did not keep their PEAR libraries up to date all along on Tiger. But the answer is on [pear.php.net](http://pear.php.net/), at the moment (it will scroll off when more news is added).

In particular, the items under February 1, 2007 and January 3, 2008 apply. Mac OS X Tiger shipped with PEAR 1.3.6, which is now quite old. Plus, the folks at PEAR decided to radically change their interface, tossing XML-RPC.

In short, what you need to do is to run these commands as root:


    pear upgrade --force PEAR-1.3.6 Archive_Tar-1.3.1 Console_Getopt-1.2
    pear upgrade --force PEAR-1.4.11
    pear upgrade PEAR
    pear upgrade --force http://pear.php.net/get/Archive_Tar http://pear.php.net/get/XML_Parser http://pear.php.net/get/Console_Getopt
    pear upgrade --force http://pear.php.net/get/PEAR-1.4.3.tar
    pear upgrade PEAR

With a little luck, and crossed fingers, you should end up with a functional PEAR library on your Mac again. Good luck!
