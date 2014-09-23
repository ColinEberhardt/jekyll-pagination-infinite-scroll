---
author: ceberhardt
title: "Swift Adoption Statistics"
categories: 
tags:

summary: "It was just two months ago that Apple took us by surprise in releasing the Swift programming language. This blog post reflects on the first few months of Swift adoption."
layout: default_post
---

It was just two months ago that Apple took developers by surprise in releasing the Swift programming language. Ever since I've been spending my evenings thoroughly immersed in the 'all new' Swift programming language. 

In just over a month I will be giving a couple of Swift talks at the [iOSDevUK conference](http://www.iosdevuk.com/), and for the past few days I have been debating what level to pitch these talks at. Do I assume familiarity with Swift? Or start at a more basic level?

> Just how many developers are using Swift?

This month's TIOBE Index (a proprietary measure of language popularity, computed from a range of sources) has shown [Swift entering the charts at 16th position](http://www.tiobe.com/index.php/content/paperinfo/tpci/index.html). However, what I want to determine is something more specific, what proportion of iOS developers have started using Swift?

Before moving on to my method and results, I just want to briefly state that Swift is in Beta, and I do not see this early measure as an indication of success or failure. It was more for my own benefit, to give me a rough idea of just how active the Swift community is at this very early stage.

##The Statistics

These days there are two sites that form the 'backbone' of software development, [GitHub](https://github.com/), where developers store and share their projects, and [StackOverflow](http://stackoverflow.com/) where developers discuss coding problems. Fortunately both have excellent APIs which have provided me with a mechanism for determining the amount of Swift vs. Objective-C activity on each site.

###GitHub

Let's start with GitHub. The following chart shows the number of newly created repositories in each language on a daily basis for the past two months:

<img src="{{ site.baseurl }}/ceberhardt/assets/SwiftAdoption/GithubRepos.png"></img>

As you can see, the day after Swift was release, a large number of Swift repos were created as developers raced to create the [first FlappyBirds clone](https://github.com/fullstackio/FlappySwift) or the de-facto utility belt library (a race which [Dollar.Swift](https://github.com/ankurp/Dollar.swift) seems to have won).

Following this sudden rush of activity, repo creation has steadied at a rate of around 50 new Swift repos per day.

In contrast, Objective-C is holding steady at around 200 repos per day. In other words, there is around 4x as much Objective-C activity on GitHub.

One other interesting difference is that Objective-C activity follows a clear weekly cycle, with the weekend seeing around half as many repos created compared to weekdays. Swift doesn't appear to follow a strong weekly cycle, with Swift development being just as active on the weekend. Possibly an indication that people are primarily using Swift in their spare time?

###StackOverflow

The number of Swift and Objective-C questions asked on StackOverflow follows a similar pattern:

<img src="{{ site.baseurl }}/ceberhardt/assets/SwiftAdoption/SwiftQuestions.png"></img>

Again, a sudden burst of Swift activity, which gradually subsiding to the point where there are roughly 3x as many Objective-C questions than Swift questions asked.

###Conclusions

The above results indicate that iOS development activity is still predominantly Objective-C, with roughly 3x more activity vs. Swift. However, considering that Swift is still in beta, this does indicate a significant amount of activity!

Based on this result, when talking with a group of iOS developers I certainly wouldn't assume familiarity with Swift, at least not yet! It will be interesting to see how quickly Swift is adopted once it is officially released.

Regards, Colin E.








