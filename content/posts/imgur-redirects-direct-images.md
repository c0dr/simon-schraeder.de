---
title: "Imgur Redirects Direct Images"
date: 2014-10-06T19:49:44+01:00
draft: false
---

Imgur is a popular hoster for images used on sites like twitter or reddit. To share the imag, you have the choice between using a [hotlink](https://i.imgur.com/igPzEWp.jpg) to the file or using an [url which shows the image along with ads](https://imgur.com/igPzEWp).


But when you use the hotlink on Twitter, it seems like imgur automatically redirects to the website.

I decided to investigate this, which turned out to be quite simple.


First let's see what imgur responds to a normal requests to the file:

`curl -I "https://i.imgur.com/igPzEWp.jpg"`

The response to this was

`HTTP/1.1 200 OK
Date: Mon, 06 Oct 2014 12:45:02 GMT [..]`

The code 200 shows, that the request was successful and no redirects are used.

If you try this again with the twitter referer

`curl -I "https://i.imgur.com/igPzEWp.jpg" --referer http://twitter.com`



the server reponds with
`HTTP/1.1 302 Moved Temporarily
Date: Mon, 06 Oct 2014 12:48:12 GMT
Server: cat factory 1.0
Retry-After: 0
Location: https://imgur.com/igPzEWp 
[...]`

The 302 status code and the new location show, that imgur indeed redirects image hits to the full site when the refer is twitter.

This might be done to cover the hosting costs (the full site shows ads), but is kind of dishonest for the users. I expected to link my followers directly to the image, now they see the full site which might take a lot longer to load. 