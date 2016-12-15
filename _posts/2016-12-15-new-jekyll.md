---
layout: post
title:  "New Jekyll version"
date:   2016-12-15 23:19:00 +0100
categories: jekyll 
---

I managed to accidentally break my website, meaning I could not push any change because Github Pages
refused to build it.

Before starting this website, I had absolutely zero experience with Ruby, Rubygems and Jekyll. I have never
written one line of Ruby, although I occasionally looked at Ruby scripts, and I never used any Rubygem
beside what was needed to set this website up. So here is how my troubleshooting went!

I notice there is a problem after I add the previous post: the post is not appearing on the website, so I open
the commit on Github and find that the website is not building. Sadly, the notification I receive on
my email address has an empty error message attached.

I suspect some Markdown or templating error in my latest post, by after stripping it almost bare, I
cannot find any obvious problem. I git-revert all new commits, going back to what I knew was a sane
state, but Github pages still refuses to build it.

I then try and serve the website from my local repository:

```
bundle exec jekyll serve --safe
```

This is preceded by me trying to find out where ```bundle``` is installed; it is in the gem directory,
in my case ```~/.gem/ruby/2.3.0/bin/bundle```. Note that you can also run jekyll independently, but
you will likely run into some issues as the dependencies are not in the expected versions.

Anyway, no errors, the site is served as expected. I decide there is some version mismatch. Github
Pages publishes the versions it uses [here](https://pages.github.com/versions/), and lo and behold, my
jekyll is outdated (I had 3.1.6). One very slow ```gem update``` later, my jekyll is at the right
version. Still, the website is served with no errors.

Running short on ideas, I try to update ruby. The latest version was not available on the Chakra
repositories, so I update the package myself, and later push it for the all the users. No change
detected as a result of this.

Oh wait; I try calling

```
bundle exec jekyll --version
```

it seems ```/usr/bin/jekyll``` is updated, but the version launched by bundle is still the previous
one. I am still not sure where that version was installed, since all jekyll binaries found by

```
which -a jekyll
```

plus the binary installed in the gem binary directory are at the new version. Thanks to a random
Stackoverflow post (which I lost in the three hundred tabs I was keeping open) I find
out that bundler needs to be updated too.

```
bundle update
```

Why was this not covered by the gem update? I have no idea. Anyway now the jekyll version is the
one I expect, and I can run the serve command.

The error was in the Flex tutorial linked at the top of the page: two
invalid UTF-8 characters sneaked into the file. I suspect the old Markdown interpreter used to skip
them, while this one throws an error. But most editors skip the characters, and the tutorial is quite
large, how I am going to find them?

Grep to the rescue!

```
grep -axv '.*' tutorials/flex.md
```

Now I can delete them with a regular editor. Day saved, website back up!

