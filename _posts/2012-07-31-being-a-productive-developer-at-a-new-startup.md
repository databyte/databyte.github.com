---
layout: post
title: Being a Productive Developer at a New Startup
---

Meetups are always fun because you get to meet a wide range of
developers. A great event in NYC is [Hacker Hours by NYC on Rails](http://www.meetup.com/nyc-on-rails/)
which offers programming help for any developer regardless of the
problem.

A developer I helped out there recently learned Rails and joined a new startup.
He asked me an interesting question regarding working at the new
startup:

> The other dev is very productive. I feel very slow and unproductive.
> Is this what it's like every time you join a project? Everything is
> just a friggin' mess?

#### Ignorance or Incapable?

I think there's a distinction between ignorance, which is not knowing
something because you haven't had the opportunity, and being incapable,
meaning not being able to do it even though you know how. When I start
new projects, it takes hours and days of reading to get past the
ignorance stage. It takes even longer if they use external libraries or
APIs that I've not used before.

Don't expect to be productive within days or even weeks. In larger
projects, it could take months.

#### Legacy Code

On some projects, the developers are disorganized and unskilled or even
advanced developers who used so much meta programming that the code
reverted itself into being even harder to read. I have to read through
the mess and the confusion to determine the program's purpose and the
programmer's intent.

A developer's goal is to make the program meet the goals of the business
and it's just as important to make it readable to other developers.

I think we would all agree that Ruby and Python reads easier than
Assembler though all languages are capable of meeting their business
needs. So you have to look at your highly productive developer and ask,
is she writing functioning code, readable code or both?

#### Falling behind

From the new developer:

> I'm burning out because I stare at "magic" all day and produce at a
> snail's pace. I don't get the reward of finishing something frequently
> and it decreases my morale.

If pairing through large chunks of the application isn't an option, I
suggest focusing on something small that's part of the whole and working
on it until you grok it. You can also start on a new section of code
based on technology you already know.

Starting on a new project with lots of existing code and developers
can be a daunting proposition. Your first commit will probably be a bug
fix or small tweak.

Personally, I always start in the tests. That's an easy place to
find the interesting business logic and the uninteresting CRUD code.
Test code needs to be refactored and structured just as much as the code
it tests which provides another opportunity to learn more about the
codebase and remove more of that "magic".

#### Doing FAST vs Doing it RIGHT

Make your code a shining example of how it should work. Don't spend too
much time polishing it. Any true craftsman will want to continue working
on their craft and as they get better and learn more, you'll look back
at your old code and wonder what you were thinking.

Any code written more than an hour ago is crap and will eventually be
revisited. One of the Ruby Rogues said that all code is experimental.

Therefore, take the time to do it right. Don't try to play catch-up and
throw away quality in favor of speed.

The difference between that highly productive developer that writes
messy code and the code that you write starts to shift the culture from
one of productive mess which inherently contains technical debt to quality
code that's easier to work with later.

It's always better to know about the engineering culture before you sign
the dotted line but if it's truly a startup, then you can shape its
future.

#### Readable code

Recently there was a popular gist from a contractor that was let go due to
applying aggressive SRP to a simple PostComment class in Ruby. The
thread is a good read but when I think of the new developer's comments
regarding "magic" code, I think of something a previous co-worker,
[Donald Ball](https://gist.github.com/2838490#gistcomment-362468), wrote:

> ... getting code to do what it's supposed to do is the lowest
> threshold for acceptability in my book. Getting code to clearly
> communicate its intent and to be easy and safe to modify is a higher
> bar, but is the standard by which I judge any code that's intended to
> work for longer than a day. We spend more time reading code than
> writing code.

