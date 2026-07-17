---
layout: post
title: "I replaced UIPath in a weekend"
description: >-
  Our customer's RPA platform failed silently for five and a half days — 592
  tasks, zero errors reported. How we knew anyway, and what I built over the
  weekend to replace it.
---

**TL;DR**

- On a Friday evening in June, our customer's RPA platform (UIPath) that writes their staff's schedule changes back into their source system failed. It was reported after hours — and the team that owns it doesn't monitor it on weekends or after hours. So it sat dark, unattended, all weekend.
- **It never reported a single error.** 592 tasks over five and a half days, zero failures logged. The bot accepted every job and picked up none of them. Inbound traffic looked perfectly normal. Every dashboard was green.
- We knew it broke right away, because a separate service we'd built months earlier doesn't ask the bot whether the bot succeeded — it re-imports the schedule from the source system and checks what's actually true. It lit up on Friday night while nobody was watching.
- Over that weekend, with Claude and Codex as pair programmers, I built **Scribe** — a service that performs the same write-backs UIPath did. First code was in production at 2:33 AM Monday. A build that would normally take weeks took a weekend.
- We never turned it off. Five weeks later, with UIPath fully healthy, **17.7% of its writes still need Scribe to fix them.** The outage was just the loud version of a problem we'd been fixing manually ourselves.

---

## The dependency we didn't control

A bit of context on what we do. Our platform provides an AI assistant that manages healthcare staffing schedules — swaps, coverage, pickups, partial shifts, schedule changes. When a change is finalized in our system, it has to be written back into the customer's _source_ scheduling system, the system of record.

The vendor for that source system is building out a proper API, but it isn't ready yet. So in the meantime, the write-back happens the way a human would do it: by driving the scheduling UI. And because of our customer's requirements, those writes don't run on our infrastructure — they're routed to **their** RPA team, who own and operate a fleet of UIPath bots. We hand them the work; their bots type it in.

That's a clean separation of duties on paper. In practice it means a critical step in our pipeline runs on a platform we can see but cannot fix — operated by a team on a different schedule than ours.

## The Friday night nobody was watching

On Friday, June 5, the RPA system failed. The report came in after hours. And here's the detail that matters most: the team that owns that platform doesn't monitor it on weekends or after hours.

So the failure just… sat there. From Friday evening until Monday morning, the thing responsible for writing our users' schedule changes into their system of record was completely dark, and the only people who could fix it weren't looking. The root cause was on the UIPath side — the same class of infrastructure failure described in [this writeup](https://www.linkedin.com/posts/kaplann-ahmet_uipath-rpa-automation-activity-7470046977079119872-3q-r) — and it would take their IT team **a week** to get the fix in place.

## How we knew: the confirmation service

Months before Scribe existed, we'd built something unglamorous: a confirmation service that independently verifies the round trip.

The logic is simple. Our system finalizes a schedule change. UIPath writes it into the source system. Then, separately, we re-import the schedule _from_ the source system on an ongoing basis. The confirmation service checks that every finalized request actually shows up in that re-import. If a change doesn't materialize within a threshold, it alerts — faster when the affected shift starts soon, because an unwritten shift starting in two hours is a genuine staffing emergency, not a data quality nit.

Crucially, it doesn't trust the writer's own report of success. It doesn't ask UIPath whether UIPath succeeded. It asks the _system of record_ what's actually true, and compares. That independence is exactly why it worked when the bot was dead: a failed write and a lying write look identical to that service, and both trip the alarm.

As our monitoring service flagged issues, we would route them to the original RPA team to beef up their bot and manually fix the source system ourselves. The team took turns clearing daily alerts. Sometimes we'd have one alert and sometimes a few but it always seemed manageable.

So on a Friday night, with nobody at the RPA team watching, our confirmation alerts told us the truth — nothing was landing. Not some things. Nothing. It wasn't the 1-2 issues we'd normally see, it was pouring in. That's how we knew UIPath was completely down.

### The number that still bothers me

I went back and queried production while writing this, and the result is worth sitting with.

Across the five and a half days UIPath was down, **592 RPA tasks (aka scheduling changes) were created. Exactly zero of them failed.**

Not a single `error`. Not a single `abandoned`. The bot never threw. It never reported a problem. The system was hard down — the jobs weren't attempted and botched, they were accepted into the queue, moved to `NEW`, and then never picked up at all. If you had been watching UIPath's own status reporting that weekend, you would have seen a clean board.

You can watch it flatline in the completion timestamps:

| Date           | Tasks completed                 |
| -------------- | ------------------------------- |
| Thu, Jun 4     | 135                             |
| **Fri, Jun 5** | **119** — last one at 20:27 UTC |
| Sat, Jun 6     | 0                               |
| Sun, Jun 7     | 0                               |
| Mon, Jun 8     | 0                               |

Normal throughput was ~135 a day. The last completion of the pre-outage era landed Friday afternoon, and then the queue simply stopped draining. Not slowly. Not with errors. It just stopped.

There was no error to catch and no exception to page on. The only observable difference between "working perfectly" and "completely dead" was whether the work actually appeared in the source system — which is to say, the only thing that could tell the difference was a system that went and _looked_.

This is the thing I'd put on a poster: **the failure was invisible in every system that reported on itself, and obvious in the one system that checked reality.** A silent failure and a successful write look identical from the inside. They look completely different from the outside, if you bother to look from the outside.

That service is the unsung hero here. Everything that follows was possible because we already had ground truth.

## The weekend

I had a choice. Wait an unknown number of days for someone else's unmonitored outage to get noticed and fixed, or build the write-back myself.

We already had the hard-won knowledge of _how_ to drive the source system's UI — that's the exact knowledge UIPath encoded. What we didn't have was a service to run it. So over the weekend of June 6–7, I sat down with Claude and Codex and built one. (Why both: that's a story for another blog post.)

The work was not trivial: browser automation against a finicky, hydration-heavy scheduling UI; a task-routing layer; a verification loop; a full session lifecycle with retries, alerting, and stuck-session recovery. On my own that's a multi-week project — most of it the slow, unglamorous work of handling the twentieth weird edge case in a web UI that wasn't built to be automated.

With AI in the loop, the ratio inverted. I spent my time deciding _what_ the system should do and reviewing whether each piece was correct; the models handled the mechanical breadth — the selectors, the retry scans, the confirmation logic, the tests. Scribe's first code was running in production at **2:33 AM Monday** — the headless-browser worker image, then the add-shift automation, then the post-create confirmation logic. By Monday afternoon the full verify-and-remediate service was deployed and we were processing messages on behalf of the RPA team.

What made the weekend possible wasn't just fast code generation — it was a fast _loop_. Every commit deployed to production within minutes, so I wasn't writing against a mental model of how the source system's UI behaved; I was writing against how it actually behaved, right now, with real schedule data flowing through. Find an edge case, fix it, ship it, watch the next session, repeat. Over the following days that loop produced dozens of small, surgical hardening deploys — late-hydrating grid cells, clobbered time fields, detached iframes, locked schedules — each live minutes after I understood the problem. And by _I_, I mean a looped skill that reviewed recent Scribe Sessions, built a fix, automated a test through the training environment and then opened a PR. That cadence, with AI absorbing the mechanical cost of each fix, is what let a weekend build survive contact with production.

I want to be precise, because "AI built it in a weekend" is usually exaggerated. I still made every architectural decision. But the calendar doesn't lie: a build that would have eaten weeks took a weekend, and it went into production handling real schedule writes during a real outage.

## Giving it back (and taking it right back)

On June 10, the customer's team thought UIPath was ready. We flipped the bypass off and handed the writes back.

It still wasn't right. We hit issues immediately and flipped the bypass **back on** — Scribe resumed, and we kept going. It wasn't until June 11 that UIPath genuinely took back over.

That failed handoff is why the next part exists. Recovery isn't a moment, it's a negotiation — and I no longer wanted our users' schedules depending on someone else's declaration that things were fine.

## Meanwhile, the roadmap didn't stop

Here's the detail that makes me laugh now and did not make me laugh then.

We had a major feature launching that same week: a new **Nurse Residency** program, in development since early May. It pre-schedules an entire year of classes, up front, for every incoming resident across every cohort. It is exactly the kind of launch you want to happen on a calm week.

It went out on **Thursday, June 11 at 3:09 PM** — about six hours after UIPath had finally, genuinely taken back over that morning. In one shot it created **3,096 class shifts across 10 cohorts**, which became 3,085 shift-adds, which became **3,922 RPA write-back tasks**.

For scale: the entire outage — five and a half days of a dead platform — produced 592 tasks. The residency launch produced almost seven times that in an afternoon.

And it went through Scribe. 3,704 of those write-backs were routed to the bot we'd finished building 72 hours earlier, on a system whose first line of code had existed for exactly three days. That's the day I stopped thinking of Scribe as an emergency measure. An emergency measure gets you through the emergency; it doesn't absorb your biggest feature change that month three days after it was written.

I'd love to claim we planned it that way. We didn't. The launch date and the outage had nothing to do with each other; they just collided. But that's sort of the point: you don't get to schedule the week when your dependencies fail, and you don't get to pause your roadmap while they sort it out.

## Belts, suspenders, and another pair of suspenders

When UIPath came back for real, the obvious move was to switch Scribe off. I didn't. The outage had made the actual risk legible: a single write-back, on _any_ platform, operated by a team that doesn't watch it after hours, is a single point of failure with a multi-day blast radius.

So we kept Scribe running and added layers. Today the stack looks like this:

1. **Independent detection (predates Scribe).** The confirmation service verifies every finalized change against the schedule re-imported from the source system. It never trusts the writer's self-report — it checks the system of record. This is ground truth, and it's the backstop under everything else.

2. **Automatic takeover when the bot stalls.** If a job sits in UIPath's queue in the `NEW` stage for more than 20 minutes, we delete it from their queue and take it over ourselves. This is the direct answer to "nobody monitors it after hours." An unattended failure on their side can no longer quietly become a problem for our users on ours — a stalled job now has a deadline, and when it blows it, we just do the work.

3. **Scribe as stand-in.** When UIPath is unavailable wholesale, a routing switch sends everything to Scribe. That's what carried us through the outage week, and it's still there.

4. **Scribe as botsitter.** In normal operation UIPath does the primary write, and Scribe independently re-reads the live schedule afterward, diffs it against what our system intended, and remediates discrepancies on the spot. UIPath does the work; Scribe checks the work and fixes what's wrong — immediately, without waiting for a human to notice.

5. **The eventual API.** The vendor's real API, when it lands, becomes the primary path — and every layer above stays exactly where it is.

Read those together and the shape is: we detect independently, we take over automatically, we verify everything a bot claims it did, and we confirm the round trip from the source of truth regardless of who wrote it. Any single layer failing no longer means schedule drift for a nurse who needs to know when they're working.

### Was keeping it on worth it?

Our botsitter keeps running now that UIPath is stable. Take a representative week, five weeks after the outage ended. Scribe ran **395 sessions**. 389 of them executed, and 345 of those were adjudicating an actual UIPath write — the other 44 were confirmations of schedule imports from elsewhere, which Scribe checks but UIPath never touched. Of the 345 that were grading UIPath's homework:

- **284 (82.3%) came back `verified`** — UIPath did the work correctly, Scribe confirmed it, nothing to do.
- **61 (17.7%) came back `remediated`** — Scribe found a discrepancy between what we intended and what the source system actually showed, and fixed it on the spot.

**More than one in every six automated writes still needs correcting.** Not during a crisis — during a totally normal week, on a healthy platform. Those aren't outages; they're the quiet stuff. An edit that silently didn't commit. A shift that didn't have a note written in explaining what changed. Each one is a nurse whose schedule says something different from what they agreed to, and none of them would have thrown an error. Previously we would have manually fixed these ourselves, but now we have a system that can catch these issues automatically and fix them in real-time.

The stuck-job takeover fired **7 times** that week, too. Seven jobs that sat in the queue past 20 minutes and would otherwise have waited for someone to notice.

That's my answer on whether to turn it off. The outage was never the real problem — it was just the loud version of a quiet one we'd been living with.

The outage was the forcing function. The defense-in-depth is what we kept.

---

## The technical deep-dive

For the engineers. Here's roughly how it's built.

**Scribe drives the real UI, in Python.** It uses Playwright (headless Chromium, baked into our worker image) to log into the source scheduling system and perform the same actions a human scheduler would: navigate to a date, open a unit grid, add a shift, edit a shift's time or notes, read a row back to confirm. There's no secret API — it's the same surface UIPath automated, which is exactly why Scribe could stand in one-to-one.

**Everything is a verify-and-remediate session.** The core object is a `ScribeSession` with an explicit lifecycle: `pending → processing → remediation_planned → remediating → remediated`, plus terminal states like `needs_review` and `remediation_failed`. A session spawns per parent request (a swap, a coverage, a schedule change) once that parent's sibling tasks have _settled_. It re-scans the live grid, diffs against intended state, and either confirms a match or plans concrete adjustments.

**"Settled" includes "stuck."** This is the takeover layer in code. A task settles normally when it reaches a terminal status (`COMPLETE`/`ERROR`/`ABANDONED`/`BYPASSED`). But a task sitting in `NEW` — accepted into UIPath's queue and never picked up — blocks only until a staleness threshold passes (`SCRIBE_STUCK_NEW_THRESHOLD_MINUTES`, currently 20). Past that, we cancel the queue item and dispatch a session with `trigger_reason=STUCK_NEW`. A bot that never starts is now just another settled state, and the pipeline keeps moving.

**Routing is a single switch.** An `RPA_BYPASS_UIPATH` path resubmits a parent's tasks so they flow to Scribe instead of the UIPath queue, canceling any live queue item first. During the outage that switch was "everything routes to Scribe." It's the scriptable, auditable equivalent of an admin "resubmit these tasks" action — which is what let us flip back on June 10, discover UIPath still wasn't right, and flip off again within the hour without a deploy.

**The confirmation service is deliberately separate.** It lives in a different app (`schedule_requests`), predates Scribe by months, and shares no code path with the writers. It reconciles finalized requests against the re-imported schedule, with escalating thresholds — a standard alert window, a tighter one for shifts starting within hours — and an `ImportHealth` gate so ordinary import lag doesn't fire false alarms. Keeping it independent is the whole point: if it shared plumbing with the thing it's checking, it would fail in the same ways at the same time.

**The hard part was the web UI, not the logic.** A freshly-changed grid hydrates asynchronously — a new cell can render a beat _after_ the loader clears — so a naive scan misses it and wrongly concludes the write failed. Scribe does bounded re-scans before deciding a cell is absent. Add-confirmation is date-pinned to the target day-column to avoid opening every same-time panel on a busy unit. Saves are confirmed by toast _and_ by re-reading the new row's id. Edits set the time field last, after the panel hydrates, because setting it early gets clobbered. Transient iframe detaches self-heal. Every one of those lines exists because of a specific way the UI lied to us — and that long tail is exactly the work AI made survivable in a weekend.

**Concurrency is guarded.** A database constraint enforces at most one open session per parent request. Stale workers that try to update a session already reaped or replaced hit a `Superseded` guard and back off instead of racing. Status transitions are filtered updates (`update where status = expected`), so two workers can't both advance the same session. Sessions run on a dedicated Celery worker so a wedged browser can't starve the rest of the queue.

**It fails loud, not silent.** A scheduled reaper sweeps sessions stuck past a threshold — re-dispatching stale pending rows, retrying stale active ones below the attempt cap, hard-failing the rest. An attention monitor emails our internal team — deliberately _not_ the customer's RPA address — whenever a session needs human eyes. The whole point of a botsitter is that it never fails quietly. A discrepancy it can't fix becomes an alert, not a surprise a week later.

---

## The takeaway

The headline is that Claude and Codex let one engineer ship a production RPA replacement over a weekend. That's true, and it's remarkable.

But the lesson I keep coming back to is quieter, and airplanes got there first. A 777's flight controls don't just carry three computers — they carry three channels, each running three lanes on different processors, compiled by different compilers, isolated physically and electrically. The redundancy isn't the count; it's the dissimilarity. Three identical computers don't vote, they agree — confidently, simultaneously, right up until they're all wrong together.

That's what the confirmation service is, and why it's the least exciting code we own. It isn't a third opinion about whether the write landed — it's the only thing in the room that isn't an opinion. UIPath's report and Scribe's report both rest on the same assumption: that the system doing the work can tell you whether the work happened. The confirmation service doesn't ask. It goes and looks. When every voter can be wrong the same way, the tie-breaker can't be another vote — it has to be the one that reads reality instead of reporting on itself. It's the reason we weren't blind while the people responsible for the failure were afk.

AI gave us the speed to replace a dependency in 48 hours. But speed only helps if you already know something is broken. Build the thing that tells you the truth first.
