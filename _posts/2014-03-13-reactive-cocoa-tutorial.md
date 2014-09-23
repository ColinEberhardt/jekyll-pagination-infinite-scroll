---
author: ceberhardt
title: ReactiveCocoa - The Definitive Guide
categories: 
summary: It feels like everyone in the iOS community is talking about ReactiveCocoa at the moment. In this blog post I talk briefly about what ReactiveCocoa is and the 'Definitive Guide' which I wrote for raywenderlich.com
layout: default_post
originalArticleLink: http://www.raywenderlich.com/62699/reactivecocoa-tutorial-pt1
---

It feels like everyone in the iOS community is talking about [ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa) at the moment. Each day on Twitter I am seeing new blog posts and tutorials appearing, and every iOS conference seems to have at least one ReactiveCocoa presentation.

So what is ReactiveCocoa? and why is it attracting so much attention? The [reactive.io](http://reactivecocoa.io/philosophy.html) website describes its philosophy:

> Instead of telling a computer how to do its job, why don't we just tell it what it's job is and let it figure the rest out? 

Sounds rather ambitious, and to be honest I don't think ReactiveCocoa really delivers on its philosophy. But at least it does indicate where they are going with this framework, they are trying to make the way we write our apps easier in quite a fundamental way. This is how I would describe ReactiveCocoa in my own words:

ReactiveCocoa is a framework developed by the GitHub team that changes the way in which you structure your applications. Rather than handling a disparate collection of 'events' in the form of delegates, target-action, KVO etc... with ReactiveCocoa there is a standard interface for all these events, which are called **signals**. Once a standard interface is applied it is much easier to transform, compose, map, filter and combine these various event sources. This results in applications where it is much easier to follow the program flow and removes the need for instance variables that represent transient state which clutter your code.

A few months ago I volunteered to write a series of tutorials for Ray Wenderlich's website on the subject of ReactiveCocoa. These have just been published:

 + [ReactiveCocoa Tutorial – The Definitive Introduction: Part 1/2](http://www.raywenderlich.com/62699/reactivecocoa-tutorial-pt1)
 + [ReactiveCocoa Tutorial – The Definitive Introduction: Part 2/2](http://www.raywenderlich.com/62796/reactivecocoa-tutorial-pt2)

**NOTE:** I originally called this "An introduction to ReactiveCocoa", the editors must have liked it because they changed it to the "Definitive Guide".

ReactiveCocoa is a tricky topic, so I used these tutorials to very gradually introduce the concepts with a lot of diagrams and simple examples. My personal favourite part of the tutorial series is the following code:

{% highlight objc %}
[[[[[[[self requestAccessToTwitterSignal]
  then:^RACSignal *{
    @strongify(self)
    return self.searchText.rac_textSignal;
  }]
  filter:^BOOL(NSString *text) {
    @strongify(self)
    return [self isValidSearchText:text];
  }]
  throttle:0.5]
  flattenMap:^RACStream *(NSString *text) {
    @strongify(self)
    return [self signalForSearchWithText:text];
  }]
  deliverOn:[RACScheduler mainThreadScheduler]]
  subscribeNext:^(NSDictionary *jsonSearchResult) {
    NSArray *statuses = jsonSearchResult[@"statuses"];
    NSArray *tweets = [statuses linq_select:^id(id tweet) {
      return [RWTweet tweetWithStatus:tweet];
    }];
    [self.resultsViewController displayTweets:tweets];
  } error:^(NSError *error) {
    NSLog(@"An error occurred: %@", error);
  }];
{% endhighlight %}

The above first signs in to twitter, then switches to a new signal which receives events when the user types in text into a search field. This is filtered so that it is more than 3 characters in length, throttled so that updates are sent twice per second, this then kicks off a twitter search, the results are marshalled back onto the UI thread, finally [Linq-to-ObjectiveC](http://www.scottlogic.com/blog/2013/02/15/linq-to-objective-c.html) is used to transform the returned JSON objects into a simple `RWTweet` class which is rendered by the UI.

There is *no way* you could code this logic so clearly and concisely without ReactiveCocoa.

Here it is in diagram form:

<img src="{{ site.baseurl }}/ceberhardt/assets/CompletePipeline.png"></img>

If you are an iOS developer, I would thoroughly recommend having a look at ReactiveCocoa. It's [not everyone's liking](http://inessential.com/2014/03/10/reactivecocoa), but it is certainly worth taking a look to see what all the fuss is about.

Regards, Colin E. 


 




