---
title: "Let's Encrypt on Uberspace with Ghost"
date: 2015-06-15T19:28:52+01:00
draft: false
---

I haven't really published a lot of blog posts in a long time due to time constraints and this post will be quite short as well.

Just wanted to give some pointers for people using the awesome webhost Uberspace who want to secure their site with Let's Encrypt, especially when using Ghost. I was using Cloudflare's SSL before, but it felt kind of wrong to have a third-party MITM your requests to serve them over HTTPS.

For the basics you can follow the [instructions provided by Uberspace](https://blog.uberspace.de/lets-encrypt-rollt-an/). 

**TL;DR**

     [yourusername@host ~]$ uberspace-letsencrypt   
     [yourusername@host ~]$ letsencrypt certonly
     [yourusername@host ~]$ uberspace-prepare-certificate -k ~/.config/letsencrypt/live/yourdomain.tld/privkey.pem -c ~/.config/letsencrypt/live/yourdomain.tld/cert.pem
 
 
And you are done, your Uberspace site is now secured with a Let's Encrypt cert. 
 
**Total time investment:** 1 minute. 

**Total financial investment:** $0. 

There really is no excuse anymore to not use HTTPS.

If you are using Ghost just like I am, you need to do the following to force a secure connection for everyone. You can check out the instructions [provided by burakb.net](https://burakb.net/ghost-auf-uberspace-https-erzwingen/).

**TL;DR**

Add `RequestHeader set X-Forwarded-Proto https env=HTTPS` to your .htaccess (probably located at /html/.htaccess)

And while you are at it, you might also add `Header set Strict-Transport-Security "max-age=31536000" env=HTTPS` and enabled HSTS so clients always connect securely.

And that's it. Your Ghost site is now using Let's Encrypt and is forcing secure connections. 


![Yay](https://i.imgur.com/AcdQL24.png)


