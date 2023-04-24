---
categories:
  - appturbo
  - android
  - app
  - business
  - game
  - voxel
  - invaders
alias: /app-of-the-day.html
subtitle: How we switched our app to a free+ads business model
title: How our ads based game makes more revenue than our paid version
date: 2014-06-02
---


For more than one year, our game [voxel invaders] was present on google play as
both a free demo version and a full paid version.

Recently we decided to remove the demo version and replace it with a free full
version with ads.

To make it simple, we just updated our demo package with a new package
incorporating the [admob] SDK.

In order to boost the new version, we also cooperated with [app of the day]
(previously known as appturbo) to make a promotion of the game.  On April 21,
people who downloaded the free version of the game would get the full version
for free, without ads.  In exchange, 'app of the day' did a promotion on their
website and to the users of their app.

The promotion was a big success in term of download: for the day the promotion
was running, we had more than 60,000 users download voxel invaders.

![Number of active devices on which the application is currently
installed](/assets/imgs/app-of-the-day/active-devices.png)


## How did it go in term of downloads and revenues?


If we do not consider the huge number of downloads from the day of the
promotion.  Before the switch, we had:

                            | demo     |   full
    ----------------------- | -------- |--------
    Average download / day  | 280      | 14
    Average revenues / day  |          | 11 USD


And after the switch:

                            | free/ads |   full
    ----------------------- | -------- |--------
    Average download / day  | 260      | 6
    Average revenues / day  | 90 USD   | 5 USD


So we went from an average of 11 USD / day to an average of 95 USD / day,
**more than 8 times more!**.

Interestingly, the average number of downloads of the free full version with
ads is more or less the same than the demo version, even though I was expecting
an increase due to the promotion.  But maybe without the promotion the number
would have decreased.  It is hard to tell since we did both the promotion and
the switch to a free+ads model at the same time.

[admob]: http://www.google.com/ads/admob
[app of the day]: http://appturbo.it
[voxel invaders]: http://noctua-software.com/voxel-invaders
