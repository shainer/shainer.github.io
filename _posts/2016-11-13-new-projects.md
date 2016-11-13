---
layout: post
title:  "New personal projects"
date:   2016-11-13 16:58:00 +0100
categories: nlp software opensource 
---

These days I am just bursting with ideas :) so let's talk about some new projects going around my mind.

### Cookbook

I enjoy cooking, especially cookies, cakes and pastries. One year ago or so I started a Google Document with all the recipes I have ever tried, and recipes that I want to try someday.

The problem? This document is now huge. The table of contents alone fills the first 5 pages, and the total size is around 60 pages. So I have decided I need advanced technology. I have looked around for existing software, but I was not satisfied: KRecipes had several bugs in the creation of categories that made the feature unusable, and others I was not able to start on a standard Chakra installation.

I considered forking KRecipes, fixing the bugs, and extending it, but then decided to just have fun and start my own project.

**Cookbook**, as it will be named, is going to be written in C++ and Qt5. It is going to be a nice opportunity to refresh my experience with Qt5 and CMake. I have not created a Github repo yet, but I will as soon as I have a minimal product.

A sneak peek on the set of features I am planning:

  - attach custom categories, or *tags* to recipes.
  - export the recipe set in a sqlite database, and upload to common storage services (Dropbox, Google Drive, ...).
  - search recipes by tag or by ingredient.
  - attach a picture to recipes.

### Machine learning for clickbaits

I have recently read two interesting articles about handling clickbait articles using Machine Learning: [one talks about detection](https://www.linkedin.com/pulse/identifying-clickbaits-using-machine-learning-abhishek-thakur), while the other [talks about generation](https://larseidnes.com/2015/10/13/auto-generating-clickbait-with-recurrent-neural-networks/). I also followed a presentation on the subject at the Grace Hopper Conference in 2015.

My own background is in Natural Language Processing. I also hate bad news articles :-)

So I am thinking of doing something in this area. Not sure what exactly. I also want to look at
[TensorFlow](https://www.tensorflow.org/). I might find a paper and try to reproduce the results (as I did for my [Semantic Similarity](https://github/shainer/semsim) university project), or put together my own dumb classifier, or work with somebody else. I'll post more details when I start something :-)

### Rust
I want to learn Rust. I also need a nice little project I can develop to get my hands dirty, or some under development library I can contribute to. I am still searching for a good idea, ideally related to cryptography.
