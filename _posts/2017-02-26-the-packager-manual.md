---
layout: post
title:  "The packager manual (for Arch and derivatives)"
date:   2017-02-26 15:40:00 +0100
categories: chakra
---

I have poor memory. One thing I have especially poor memory for is Unix commands and parameters. On top of that, I don't do
packaging often, so some less common commands escape my mind quickly. This
post tries to address this problem by listing some of the commands I find both
useful and hard to commit to memory.

It follows that this post is likely more useful for me than for anybody reading
this, but you never know. If there are other commands you use during packaging
that you would like me to include, feel free to let me know.

The reason I specify "Arch and derivates" in the title is because some of the commands are for ```makepkg```. The rest, however,
are more generic.

## Makepkg

### Only download and extract sources

```
makepkg -o
```

This also runs the prepare function, if present.

### Do not download sources, use existing ones


```
makepkg -e
```

### Repackage only, no build

```
makepkg -R
```

### Cleanup after build

```
makepkg -c
```

### Cleanup before build

```
makepkg -C
```

## Diffs and patches

### Creating a patch from two files

Make sure you execute this from the root of the source directory, so that
file paths are complete.

```
diff -Naur old_file new_file > my_patch.patch
```

### Applying a patch

Make sure the cwd is set to the root of the source directory when this is
called:

```
patch -N -i my_patch.patch
```

The ```-pN``` flag is useful to strip N parent directories from the paths
the patch refers to, in case you are located at a deeper path than the root.

