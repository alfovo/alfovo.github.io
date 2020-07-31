---
layout: post
title: Updating this Website
date: 2019-05-23 14:49:03 -0500
tags: [ux, illustration, branding, barks]
thumbnail: /assets/img/design/happy_fish.svg
---

Now that I have a bit of extra time, I wanted to finally make some tweaks to things that have been bothering me about this website.

Isn't this just a static site? You might ask. How much energy could I possibly spend its performance? Personally, I like having performance brain worms. It seems more like an art than a science and as such is a good place for regrets and unfulfilled longing.

When I told German about updating the fileformat of all my images to , he told me I was overthinking it. Make no mistake, I am. But maintaining a website that gets at most 20 visitors a month is already an excercise in overthinking.

A caveat, I try hard to avoid over architecting what I build at work, especially the things I build from scratch. You'll rarely finding me adding aggressive code coverage tools, or domain classes we don't need now but could use later to a brand new repo. This is because I'm either making tools or a service that interacts with existing services and my users are usually other developers. As such my primary goal is for those uses to quickly see how they can interact with or add to the mvp I've built. I inherited this mindset from more experienced engineers who built tools for me when I was a software engineer in test dreaming of being a platform engineer.

- resize and compress all images. This one was hard and tedious, but I got all images down to less than 100kb from (embarassing, yes) some being over 2MBs
- remove bootstrap! Why did I use bootstrap in the first place? Well, I remember finding out about it in 2012 when I was anxiously waiting for the results of database queries that took up about 14 hours. I dreamed of getting to work on the front end and
- CSS grid and flexbox are actually even easier to use.
- load only relevant css
- remove font awesome, lol
- switch from github-pages to netlify for better image caching and a better Page Speed score.

Simplify design

My assumption with these changes is that the best optimization is deletion, so not using any third party services like bootstrap or font awesome would likely make my website load faster.

Despite all of this, my site has even worse statistics on , specifically because of the paint.
I don't know how this is possible, especially because it's an svg! I suspected it was an image cahing issue. I am hosting my site from github-pages which has a Max cache of 10 minutes and no clear way to change it.

In a huff I tried deploying my site with netlify and BAM, all my problems are solved. Something to do with cacheing I know it.
https://macwright.org/2016/05/03/the-featherweight-website.html
http://carlos.bueno.org/optimization/
http://diveintohtml5.info/canvas.html
https://swtch.com/~rsc/regexp/regexp4.html
