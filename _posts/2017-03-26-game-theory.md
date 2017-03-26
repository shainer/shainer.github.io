---
layout: post
title:  "Game theory (or what I am doing these days?)"
date:   2017-03-26 19:20:00 +0100
categories: math
---

My previous efforts are grinding to a halt a bit. Reason? I have started studying Game Theory and I still haven't managed to stop!

This isn't my first encounter with this fascinating science. In university my AI courses briefly touched on the subject,
since it is a useful tool to model interactions between autonomous agents. We did not get very far beyond basic definitions, the Nash equilibrium, and techniques to find it.

So after a recommendation, I picked up [the main course on the subject](https://www.coursera.org/learn/game-theory-1) on Coursera. Online courses, as one can expect, vary a lot in quality, even if you pick the subject you love the most. This one,
which has been around for several years now, is definitely on the good end of the spectrum. The lectures do a good job of
explaning the material, provide many real-life scenarios you can use as inspirations, and the associated exercises are fun
to go through even if you do not intend to "pass" the course.

My main criticism is that the lessons are sometimes too short for what they present, so they
remain light on the math concepts and proofs behind the theorems. I have just started [the advanced course](https://www.coursera.org/learn/game-theory-2), by the same university and professors, and for now it seems that it goes way deeper into the math. This second course explains the less "glamorous" aspects of game theory. I had no clue some of them were part of this science to begin with.

The first one is called [social choice theory](https://en.wikipedia.org/wiki/Social_choice_theory) and deals with a well-known
problem for anybody who has ever followed an election: given some "candidates" to vote upon, and a set of voters with their preferences over the candidates, how do you pick the winner? There are clearly many ways of "aggregating" the individual preferences into a final choice or ranking, so which one is the most faithful? What properties do you use to define what a good choice looks like? [Arrow's theorem](https://en.wikipedia.org/wiki/Arrow%27s_impossibility_theorem), the main result in this field, is definitely going to be a surprise!

What's next? Well I still have to find out myself...

In the meantime, thanks to a very productive conversation with the Anitya team (started by my post), I learned that they are working on removing the dependency on the RPM library, and to improve the login mechanism. I decided it's better to leave the experts to this: implementing a versioning function that is general enough to apply to as many release systems as possible is definitely a long and complex task, and the team has already started the discussions around it.

While I wait for their code to be complete, I forked the repository and replaced the dependency on RPM with a much simpler version that "works for now" (but is in no way suitable for real use). Now I can run Anitya on my machine, using virtualenv, and add projects to it. The next step is to install fedmsg, let Anitya publish notifications for new releases on the designated topic, and attach some Python script to translate this into a notification for me that one of my packages needs an update, for instance, a bug assigned to me on the Chakra GitLab instance. I will publish this on Github when ready, alongside with setup instructions for the whole system, so other packagers can benefit from it.
