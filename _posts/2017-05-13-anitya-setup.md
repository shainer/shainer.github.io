---
layout: post
title:  "Anitya setup completed"
date:   2017-05-13 17:10:00 +0100
categories: chakra
---

To recap:

-  I can run an Anitya instance on virtualenv. This is enough for my personal
   use. If we decide to run an instance on the Chakra server, I will try running
   with docker too.
-  The RPM library has been made optional; an emulator is available.
-  I wrote [The GitLab hotness](https://github.com/shainer/the-gitlab-hotness).
   This small application waits for new messages on fedmsg, and when the message
   concerns a package update, it files a new bug assigned to me on the Chakra
   Gitlab instance.

I still need to do an end-to-end test to make sure everything is connected
properly, but that should be easy.

The new script also needs to be improved; I want to be able to recognize the
right project to add the bug to, based on which repository the package is hosted
on at the moment. For the moment I am just using the same one regardless.

Thanks to the team for being responsive to my queries and solve the problems I
reported. I don't think I will talk about this again, given that the bulk of the
work is done.

