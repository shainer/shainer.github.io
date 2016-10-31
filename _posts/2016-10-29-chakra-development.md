---
layout: post
title:  "Updates on Chakra development"
date:   2016-10-29 21:54:00 +0100
categories: chakra calamares akabei software opensource c++
---

After returning from the Chakra meeting I felt more motivated than ever, and we have many many ideas to put into practice. So let's talk about what the [Chakra developers](https://chakralinux.org/?contributors) are up to these days.

### Akabei

Gallaecio has done awesome work improving the test coverage. He used [GMock](https://github.com/google/googlemock) to stub out the external dependencies. This means you can build and run the tests without having the external dependencies (polkit, libarchive...) installed on your system. This also required a few architectural changes.

The next piece of work is getting rid of libarchive, the only C library used in akabei, in favour of [KArchive](https://api.kde.org/frameworks/karchive/html/). Gallaecio performed a benchmark to compare the performances, and we are satisfied KArchive is not going to cause a degradation. KArchive is the official KDE archiving library, which split out of kdelibs in 2013.

In the meantime, I keep testing akabei extensively. I found a bug in the replacement which caused an infinite loop. It turned out to be triggered by a very specific type of replacement; fixed now, and I took the opportunity to improve the user messages in case of "no update needed". The next step in my list is a cleanup: I want to revisit the validation process and ensure infinite loops can never happen.

### Calamares

I finished the work for the new users dialog; it's now blocked on testing, which I had not have time to finish.

The new features I have added:

*  support for creating (and deleting) multiple users.
*  support for choosing the login shell; the list of available shells is part of the module configuration.
*  support for setting the avatar from file for KDE systems, which can be shut off.

All of this were feature requests on the Issue Tracker. Features I left aside for future versions:

* setting avatars from URL too.
* computing password strength.

#### Edit Oct 31, 2016
I tested the dialog on a newly-released Chakra Goedel, the [new beta ISO](https://rsync.chakralinux.org/releases/testing/). After a few bugfixes and some documentation, I created the [pull request](https://github.com/calamares/calamares/pull/270).

![A preview of the module]({{ site.url }}/images/vb-chakra-goedel-users-view.png) while testing in VirtualBox. Sorry, no fancy preview of Chakra :-)


### Website

I am working on what  I am calling the _commit feeds_. This is going to be a box or panel in the home page of the Chakra website, and it is going to show the latest commits from our repositories.

Given the upcoming switch, I am writing this for  Gitlab directly.

Gitlab has a pretty extensive API to get information about projects and commits. Given this, I wondered if somebody had done a similar "module" before, but I could not find anything. Tips appreciated!

You can see the progress on the [Gitlab repository](https://gitlab.chakralinux.org/shainer/commit-feeds).
