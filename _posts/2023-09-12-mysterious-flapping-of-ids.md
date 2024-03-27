---
layout: post
title: The Mystery of "Flapping" IDs
date: 2023-09-12 14:49:03 -0500
tags: [code, data, talk]
thumbnail: /assets/img/gouache/summer_tree_thumbnail.png
---

This week I solved and extremely satisfying bug that caused a single user's activity on the Khan Academy website and mobile application to be incorrectly associated with multiple users. I’d like to do a future presentation on this bug at Tech Confluence, but here are some of the more juicy details, still fresh in my mind.

## What’s a Browsing Session?

A user's browsing session represents the time that user is active on the application. We set it when they first land on the site or open the mobile app, via a `browsing_session_id` cookie in Fastly, our CDN, and reset it if they've been inactive for 30 minutes or log out.

## What’s the bug?

This bug first surfaced when we noticed multiple users that logged in consecutively on the same device shared a browsing session ID. While looking at the Fastly logs, I also noticed that the initial series of requests a user makes on mobile web causes a toggling or “flapping” between requests. This behavior suggests there must be a general problem with clearing an old browsing session and establishing a new one.

I was able to reproduce the behavior where my same browsing session ID persisted after I logged out. When I looked at the logs for my IP address, I noticed that for a logout request preceded by other requests in the same second, the browsing session ID never updated. I also saw requests that completed milliseconds apart caused the browsing session ID to “flap” between several values until settling on one value. It became clear that we had a race condition: when many simultaneous requests occur as a new browsing session is set, it causes the browsing session ID to “flap” or toggle between a few different values, sometimes landing on the old browsing session ID and sometimes a new one.

![bsid "flapping" case]({{ "/assets/img/system_diagrams/bsbug_flapping.svg" | absolute_url }})

## What’s the fix?

How would you solve this? While it makes the code more complicated, I took a stab at it by making two changes:

1. I decoupled the browsing session ID and expiration into two cookies.
2. I increment the ID when it is expired.

In order to reason about how to solve this bug, I drew a diagram of the expected behavior for seven cases, including the three race conditions.

### For Concurrent Requests

I started with the four most simple situations, including the three in which we want to start a new browsing session:

- Case 1: When the user is first interacting with the application, in which case their browser will not already have a browsing session ID nor expiration cookie.
- Case 2: When the user is has last interacted with the application less than 30 minutes ago, in which case their browser will have both the browsing session ID and expiration cookie.
- Case 3: When the user has not interacted with the application for 30 minutes, in which case their browser will have cleared the browsing session expiration cookie.
- Case 4: When a user logs out and a response from the backend expires the cookie, which is then cleared by the browser.

![Simple bsid behavior cases]({{ "/assets/img/system_diagrams/bsbug_simple_cases.svg" | absolute_url }})

### For Synchronous Requests

What about the cases in which there are synchronous requests when a new browsing session ID is generated, triggering the race condition? Those are modeled in the three cases below:

- Case 5, a synchronous version of Case 4: When a user logs out and a response from the backend expires the cookie, which is then cleared by the browser.
- Case 6, a synchronous version of Case 3: When the user has not interacted with the application for 30 minutes, in which case their browser will have cleared the browsing session expiration cookie.
- Case 7, a synchronous version of Case 1: When the user is first interacting with the application, in which case their browser will not already have a browsing session ID nor expiration cookie.

![Complex bsid behavior cases]({{ "/assets/img/system_diagrams/bsbug_hard_cases.svg" | absolute_url }})

## Conclusion

Huzzah! We’ve solved the “flapping” that occurs when a user logs out or stops interacting with the mobile app or Khan Academy browser tab for half an hour. We still observe that a lot of IDs are generated when the user begins browsing, especially with all the simultaneous requests on mobile, however, we don’t see the “flapping” back and forth between IDs nor do we see one ID being associated with multiple authenticated users.

It was difficult solving this bug without being able to order requests based on their start time. To make progress on it I had to diagram my mental model of the problem and complain about it a lot to my friends and colleagues. If you are one of them and have read through this long post at my request, thank you.

![Summer trees]({{ "/assets/img/gouache/summer_trees.png" | absolute_url }})
