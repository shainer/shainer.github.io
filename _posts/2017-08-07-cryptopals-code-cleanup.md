---
layout: post
title:  "Cleaning up the Matasano repositories"
date:   2017-08-07 13:00:00 +0100 
categories: matasano cleanup
---

I started the Github repository for the Matasano crypto challenges mostly for myself. It's not software with actual users, so I didn't pay a lot of attention to code health or general organization.

But as I kept adding more files, and referencing my code in posts, I realized it was a pretty bad shape and it would benefit from some love. So I went back nand reorganized all the files, making sure viewers (and myself) can easily figure out what Python file is a binary you run to solve a given challenge, and what is a library used to abstract common operations. The README also gives more information about the status of the work and what dependencies the applications have.

Therefore I officially apologize to the people who looked at it or forked it when it was in a much worse shape :-)

This cleanup also had an unintended consequence: I realized I didn't actually solve all the challenges. I skipped the MD4 one on purpose, as I already explained, but I actually "left for later" another 3-4 challenges, and then completely forgot about this. Well, better late than never, so I am now going back and solving them. Challenge 20 was the first of the list, and I just submitted the solution this morning.

So double yay for code cleanup!
