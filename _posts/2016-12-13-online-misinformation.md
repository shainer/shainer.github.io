---
layout: post
title:  "A summary on online misinformation"
date:   2016-12-13 21:10:00 +0100
categories: nlp
---

### Introduction
Not so long ago, I mentioned that I hate bad news and that I am planning to implement a clickbait classifier. But obviously bad news are not limited to articles with a clickbait title.

Inspired by a reference on a debunking blog, I had a look at some of the recent scientific literature on the spreading of
misinformation over the Internet.

In this article I will summarize some interesting things I have found. All material is linked in full form in the References
section at the end of the post.

Brace yourself, this is going to be a long one!

### The background
Most papers or articles mentioned the same report, produced by the World Economic Forum in 2013.

In this report, it is commented how the Internet enables pretty much everybody to become the source of widely-spread information, whether true or not. This contrasts with the past, where it was still possible, but restricted to the few "powers" with the practical means to reach many people at once (e.g. large TV stations).

False information has often clear consequences, regardless of whether the original source intended them. Even satire news, meant to mock the gullibility of users, can cause them if taken seriously for too long. Moreover, malign sources, with an agenda to push, abound.

The report then discusses possible actions to take to combat these so-called _digital wildfires_: it explains why restrictions on freedom of speech may backfire, and explores the idea of "augmenting" the Internet experience with tools to warn users about low credibility sites, perform basilar fact-checking, and similar functionalities.

This is very similar to what many large IT companies, such as Facebook, have been proposing these days.

### The spread of online misinformation
This is the study I started with. Here the analysis is also focused on how Internet communities, and users, are influenced by the spread of conspiracy theories or other forms of misinformation. 

The authors prepared three datasets: one with sound scientific news, one with conspiracy theories, and a third control set with articles taken from "troll news sites", i.e. sites that intentionally spread fake news in order to mock. The elements of the two main sets are discriminated based on the verifiability of the content.

From each article, a _sharing tree_ is built, modeling users as nodes. User A and B are linked together if user B has reshared a post received by user A.

#### Cascades
The first analysis is on how news spread: _lifetime_ (time between first and last share of the post), size and degree of the cascade (how many users are reached, and how far from the original source).

A difference emerge: science news reach the maximum cascade degree sooner (2 hours), and a longer lifetime does not correlate with larger cascades. Conspiracy theories are slow to diffuse (lifetimes up to 20 hours), but live longer if the cascade they generate is bigger.

#### Homogeneous clusters
Here we drive a bit into sociology to talk about what drives users to share something. The users' opinion is defined by a degree of **polarization**: a number between 0 and 1 that explains how likely the user is to share conspiracy articles rather than sound articles.

It is the found that the majority of links in the sharing trees is **homogeneous**. This means that the link connects two users of similar polarization. The content, of any nature, circulates better inside the echo chamber formed by a community of similar opinions.

_Personal note_: there is quite a bit of statistical modeling theory applied here to reach these conclusions on a large dataset, and to validate the assumptions. I only know fundamentals of statistics, so I won't comment on that.

As a final validation, a model of rumour spreading is built and tested. In this model, news are initially shared by a group of users. Each article is assigned a fitness, i.e. the opinion it conveys. A user is more likely to reshare when the fitness is closer to its own opinion (with a fixed threshold). Both fitnesses and opinions are uniformly distributed, and independent with each other. Finally, link homogeneity is added as a factor that lets a user decide whether to reshare or not. The model has been found to simulate the real data very significantly.

### Trend of narratives
Another paper tried to classify the type of misinformation we see, and how users react to each different type.

Big data analysis has been performed on a corpus of public data retrieved with the Facebook API: posts, likes and comments.

Four main categories for misinformative articles were listed: geopolitics, diet, health and environment. Term occurrence and co-occurrence was used on the corpus to derive which words are more likely to appear, or to appear together, in each category.

Then consumption of the content by the users has been studied, in terms of:

- **average distance** between first and last comment on a post;
- **sentiment analysis of comments**: positive, negative or neutral? Note that negative here does not imply a disagreement with the content of the post, but rather whether the tone expresses unhappiness, indignation etc. rather than being optimistic in nature.
- **user persistence**: how much are users active on known "misinformation" pages? Does this imply they are more likely to become active on other similar pages?

There were no significant differences discovered between consumption in the four different categories, but there were some. One jumped to attention: geopolitics conspiracy theories tend to travel around the Internet more, and to engage users for longer time (e.g. in debates), while diet and health engage many users for a short timespan and then tend to disappear.


### Another clickbait classifier
I am of course not the first to undertake this task :-) The most complete paper I could find has been published on LinkedIn a few months ago.

In this implementation, the goal is to assign to each title a probability of the title being a clickbait, between 0 and 1. Two main features have been used:

- identification of the more clickbaity and less clickbaity n-grams, using TF-IDF. Turns out that numbers are very clickbaity; many such articles are organized as lists for easy consumption, so titles like "15 things you never knew about X!" were common enough in the corpus to generate this result.
- Word2Vec: every word or n-gram in the corpus is represented as a multi-dimensional vector. Words with similar meaning should be "close" in terms of distance between the vectors. This allows to find more clickbaity words that did not  show up in the previous analysis, and also to remove some N-grams that the previous feature marked as clickbaity for tangential reasons, such as "Donald Trump".

Two models were trained, one with Logistical Regression and one with Gradient Boosting, using an ensemble of the two features. The validation showed very positive results: precision, recall score and F1 score of 0.93 (for both, but GB had higher following decimals for all three).


### References

* [Digital wildfires in a hyperconnected world](http://reports.weforum.org/global-risks-2013/risk-case-1/digital-wildfires-in-a-hyperconnected-world/), report by WEF, 2013.
* [The spreading of misinformation online](http://www.pnas.org/content/113/3/554.abstract), Del Vicario et al., 2015
* [Trend of narratives in the age of misinformation](http://journals.plos.org/plosone/article?id=10.1371/journal.pone.0134641), Bessi et al., 2015
* [Identifying clickbaits using machine learning](https://www.linkedin.com/pulse/identifying-clickbaits-using-machine-learning-abhishek-thakur)

