---
layout: post
title: Release often
---

It's easy to not release as you build up more and more features with
more and more dependencies. It feels riskier since you haven't had a
chance to verify and test everything.

But release it immediately. Stop adding features. Remove code, segment it off
my removing the paths to it, use feature flipping or do whatever you can
to release what you have right now. Get it out there early.

A great example is the recent Heroku client tools that have had a series
of bugs since May. They can easily be tracked between version 
[2.25.0 released on 04/24/2012](https://github.com/heroku/heroku/blob/master/CHANGELOG#L325) 
and [2.26.0 released on 05/23/2012](https://github.com/heroku/heroku/blob/master/CHANGELOG#L269). 
The releases were a month apart and 2.25.0 released 5 changes whereas
2.26.0 released 51 changes. Needless to say, the .1 release the
following day contained 4 fixes and the .2 release the day after that
contained another 2 fixes. The next stable release that didn't contain a
fix was 2.27.0 on 06/14/2012 which released only 13 changes and no other
release since then has had more than 12 changes.

Don't push for those extra hours thinking you'll get it out in time
because you'll be spending even more hours fixing all those last minute
mistakes.

The lesson is, don't release 51 changes after 4 weeks of development.
You should release only a dozen changes every few days at most.

Simply avoid big releases.

