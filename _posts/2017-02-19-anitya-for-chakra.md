---
layout: post
title:  "Anitya for Chakra"
date:   2017-02-19 18:20:00 +0100
categories: chakra
---

[Anitya](https://github.com/release-monitoring/anitya) is a cross-distribution monitor for project releases, born and maintained by the Fedora project. It is a service where registered users can add projects to monitor for new releases. Anitya provides a dashboard that marks projects with new releases, and also announces them through messages over fedmsg, Fedora in-house messaging system. It supports all popular coding hosting services.

The official Anitya service used by Fedora can be found at [release-monitoring.org](https://release-monitoring.org).

The name also has an [interesting origin](https://en.wikipedia.org/wiki/Impermanence).

Today I tried to set up Anitya on my local machine; I thought it could help me, and perhaps other Chakra packagers, "stay on top" of some packages I maintain in the Chakra non-core repositories. I tend to be very distracted and add new packages I like and then forget checking for updates.

Anyway, I run into some issues:

**Login**. You need to be logged in on the UI to add or delete projects to monitor. The local instance offers you login through Yahoo, Google, a generic OpenID server and Fedora. The Google login is broken (I have [filed an issue](https://github.com/release-monitoring/anitya/issues/437)). I would have preferred for a login through Github to be available. To move forward, I created an account on the Fedora Project website, which took some time to get finally confirmed.

**RPM is required**. I logged in, and tried to add a Github project. I get presented with a Python exception dump. Some internal module tried to import the rpm Python library to compare release versions. I understand that Fedora wants to be sure version sorting is compatible with what they use in their packaging system, but this severely limits the distributions where it can be run out of the box.

The offending line from the dump:
```
File "anitya-0.11.0/anitya/lib/backends/__init__.py", line 132, in rpm_cmp
```

On the flipside, the error page looks pretty nice :D And this is where I stop, for today. As soon as I find time, I may fork the project and make a version comparison that works for Chakra.

In the meantime, thanks to [OSSBlog](https://www.ossblog.org/markdown-editors/), I  put a stop to my search for a good Markdown editor. [Haroopad](http://pad.haroopress.com/) wins the contest. The only feature I don't like is that it seems to handle one file at a time; when you open a new one, the previous one cannot be accessed in any way that I could see. Good enough for what I need it, but I may change my mind in the future.
Too bad for Atom, which I would have also appreciated as an IDE for programming, but it requires nodejs 6.x, and Chakra packages 7.5.0. I could use NVM to manage which version I keep installed, since I don't even need it for anything else, but that's more work I want to put right now.