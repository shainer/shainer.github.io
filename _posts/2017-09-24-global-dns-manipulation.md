---
layout: post
title:  "Usenix Security 2017: global DNS manipulation"
date:   2017-09-24 15:30:00 +0100
categories: security dns
---

Usenix Security 2017 happened recently, and Usenix has then published all the videos and proceedings
(see [their main page](https://www.usenix.org/conference/usenixsecurity17)). Since I didn't attend, I looked
through the videos to watch those I found more interesting. [One in particular](https://www.youtube.com/watch?v=W_rBPdaTojQ) struck my attention.
The presenter talks about a study performed by several academic bodies to measure DNS manipulation across the world in a reliable
and repeatable fashion.

What is DNS manipulation? DNS manipulation means changing the responses to DNS resolution queries to prevent
the requestor from accessing the actual domain they were looking for. Such "bad" responses will either be errors
or different IP addresses that redirect the user somewhere else.

Of course, the success of this depends on the DNS server(s) that perform the manipulation being somewhat popular
among the audience we intend to target. If people can easily use a truthful server instead, they are eventually
going to do so. But I digress; let's assume that problem is more or less solved (and in practice this happens
all the time, at least to non-power users).

The study asked the questions of how common DNS manipulation is in the current world and what forms does it take.
Surprisingly while we all agree that Internet censorship is a thing, and can probably name a few notorious examples that
make headlines, comprehensive data on the subject are hard to come by.

This introduces the next question: how do we collect such data? Half of the video presents their pipeline dedicated to collect
lots of data about DNS manipulation across the world. The next presents some results, divided by country, category
of domains (pornography, gambling, human rights, multimedia sharing were among those graphed on the slides) and
actual top-level domain (are Google-owned domains more often censored than, say, Wikipedia or Facebook?).

The first lesson taken from this experiment is that it is incredibly easy to introduce bias in the results by
simply not checking a diverse enough set of domains. We can only check whether a domain is manipulated by issuing
DNS queries and look at the result. We cannot infer whether other domains are also manipulated in the same context.
Therefore if we don't start with a varied input set, our picture is going to be incomplete.

The second is that manipulation is incredibly common, and it manifests itself in a variety of ways. Discussions on this
go quickly into politics of specific countries, so I encourage people to look at the data presented and form their own
opinions on all this.
