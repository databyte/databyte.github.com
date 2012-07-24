---
layout: post
title: Heroku's uptime numbers are off
---

#### TL;DR

Heroku's uptime for June under their rules is 99.28% but really it's
96.25%.

I like how Heroku added an info icon which 
[links to a page](https://devcenter.heroku.com/articles/heroku-status-uptime-calculation) 
explaining their modified uptime numbers. They added in the number of
running applications affected in each outage. There are a few problems with this.

Firstly, idle applications do not count but unfortunately during many of
these outages - idle applications can't come online. I don't believe
I've heard of a service that said our site's availability was 100%
because you weren't logged in right before the event happened. But for 
Bob who was logged in at the time, his uptime was 99.28%. Additionally
I've not seen uptime reports take into consideration the number of
accounts.

Secondly, how do they count the number of applications affected and how
are they sure? During last month's outage, I had several clients have
their sites available but degraded and slow.  Some sites were available
one minute, gone the next and then back again. How do they count these?

At another company I recently worked with, we had several Heroku applications
working in tandem in an SOA configuration. Such an event may have
affected one application in some manner which would then affect the 
entire system. There's no way to calculate that either.

So Heroku lists their uptime for June as 99.28% with all these new
considerations in place which is still... pretty bad.

I'll give Heroku a pass on [Varnish being down](https://status.heroku.com/incidents/389) 
for an hour since Cedar is the default but I won't give them the 
[API being offline](https://status.heroku.com/incidents/383) for 4m 
as "Development" only because many DevOps operations require the use
of the API. If you can't scale up under load, then you
have a production level problem.

The uptime for production only outages for June is **96.25%**. Not 99.28%.

[Discuss on HN](https://news.ycombinator.com/item?id=4288582)

