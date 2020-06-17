---
layout: post
title: Updating this Website
date: 2020-05-23 14:49:03 -0500
tags: [front-end, update]
thumbnail: /assets/img/design/website_update.svg
---

Now that I have a bit of extra time, I wanted to experiment with the root of all evil, prematurely optimizing this rarely visited website.

How much energy could I possibly spend on the performance of a static site? I imagine the people visiting don't have much time to spare for my lovingly curated posts, so I wanted to make sure the home page loads as quickly as possible. Here is what I did:

# Resized and compressed all images

This was hard and tedious, but initially, some images on my site were larger than 2MB (embarrassing, yes), and now all images on my site are smaller than 100kB. I contemplated updating the non-SVG images' format to Webp, but that seemed like way too much work.

# Reduced the number of render-blocking resources per page

The first thing I tackled was removing [bootstrap](https://getbootstrap.com/). Why did I use bootstrap in the first place? It seemed very cool when I first read about it in 2012 while awaiting a SQL script that had been running for 14 hours. Well, I'm glad that's all over with, because CSS grid and flexbox load faster and are much easier to use. Next, I replaced [Font Awesome icons](https://fontawesome.com/icons) with inline SVGs or emojis. Apologies to anyone looking at those emojis with an older browser. 

Finally, I divided the CSS into multiple files so that only relevant styles load per page type. I also inlined styles only used within a single post. I didn't bother trying to eliminate the 6.2kBs of unused (render-blocking) CSS I've inherited from Jekyll's minima theme.

# Simplified the design

In my opinion, navigation on website headers is often overloaded, especially when there are only one or two types of content the site is displaying. I decided to make my header very simple and vertical, eliminating any need for a hamburger menu and a different layout for the mobile version of the site. 

It took me a few years, but like the artist [Yankee Pope](http://yankeepope.com/testcom/fonts.png), I hate all (web) fonts. This website only uses system fonts &mdash; serif headings in [Iowan Old Style](https://en.wikipedia.org/wiki/Iowan_Old_Style), sans-serif type in [Avenir Next](https://en.wikipedia.org/wiki/Avenir_(typeface)#Avenir_Next), and monospace in [Monaco](https://en.wikipedia.org/wiki/Monaco_(typeface)).


# Changed hosting from GitHub Pages to Netlify

After making all the aforementioned changes, I assumed my website would load significantly faster. However, my site's statistics on Google's [PageSpeed insights](https://pagespeed.web.dev/report?url=https%3A%2F%2Falexvolpert.com%2F) did not improve as much as I expected, and the largest contentful paint remained very slow, even when I reloaded a page I had presumed would be cached. I suspect that because the server for GitHub Pages has [a cache set to last only 10 minutes](https://webapps.stackexchange.com/questions/119286/caching-assets-in-website-served-from-github-pages), the images re-downloaded as I clicked around the site. I decided to deploy my site with netlify, which has [a different caching strategy](https://www.netlify.com/blog/2017/02/23/better-living-through-caching/), and surprisingly both the posts and homepage loaded much faster.