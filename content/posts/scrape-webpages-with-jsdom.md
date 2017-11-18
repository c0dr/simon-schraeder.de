---
title: "Scrape Webpages With JSDOM"
date: 2015-05-03T19:49:12+01:00
draft: false
---

A few days ago I took part in the "Data Münster Hackathon", the kickoff event of a usergroup/meetup here in Muenster. It was a three day long hackaton (1 day of coding), which I won together with a colleague (recieved an iPad Air 2 :) ).

So together with a friend I knew from "Code For Muenster", a meetup where we work on open data problems (my projects are on [GitHub](https://github.com/c0dr) we decided to work on a logo parser.

The goal of the application was to get a logo for any given website. This could have some intersting use cases, too. For example imagine a customer signs up for your SaaS app and the dashboard is already branded with his companies logo and colors. How cool would that be?

Anyway, scraping the logo turned out to be quite easy. I started using Cheer.io, which didn't really parse the pages correctly. So I decided to use NodeJS together with JSDOM.

In the following I'm going to show you how to scrape webpages with JSDom at the example of my website :).

####Installing Node.jS
Most of you should have node.js already installed, but if not you can follow the instructions [here](http://nodejs.org/download/).

####Installing JSDom
After creating a new folder where you put the files related to this project in you can simply do `npm install jsdom`. If you want to install it globally, append `-g`. 
You can also create a package.json to make it easier to deploy.

**Update 3rd of May 2015**: JSDom decided to stop NodeJS development and focus on io.js. You can still install the nodejs version by using `npm install jsdom@3`.

####Using JSDom

Create a `.js` file for your project.

First of all, you need to require jsdom.
`var jsdom = require('jsdom');`


The following will load the page simon-schraeder.de, add jQuery to it and allows you to use all jQuery codes you know from front-end development like `$('.blog-title').text();`

`jsdom.env('http://simon-schraeder.de, ['http://code.jquery.com/jquery.js'],
   function(err, window) {
      var $ = window.jQuery;
      // You can access jQuery by $ in here
      $('.blog-title').text(); //"Simon Schräder"
     });
`

As a front-end developer this makes it really easy to scrape websites, because you don't need to learn a new syntax to target an object, but you can use the familiar jQuery.

Of course you can not only inject jQuery, but any script you like. So if you prefer jqLite, that works, too.

In case you are interested in the logo scraper, check it out on [GitHub](https://github.com/c0dr/LogoParser)
