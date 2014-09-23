---
author: ceberhardt
title: PropertyCross 1.3 Released
categories: 
summary: PropertyCross has just announced a v1.3 release, which includes two new frameworks, a number of updates and an improved build system.
layout: default_post
---

![PropertyCross](http://i.imgur.com/boioGZn.png)

[PropertyCross](http://propertycross.com) has just announced a v1.3 release, which includes two new frameworks, a number of updates and an improved build system.

PropertyCross, which is now part of the [TasteJS](http://tastejs.com/) family of projects that also includes [TodoMVC](http://todomvc.com/), is an open source project that allows developers to compare a wide range of cross-platform mobile application development frameworks. The project facilitates comparison by showing exactly the same application, a UK property search application, implemented using each framework. As a result, developers can compare the code, developer experience and importantly the end-user experience that each framework delivers.

## PropertyCross and TodoMVC

Whilst PropertyCross has exactly the same goal as to TodoMVC, to help developers pick the right framework for their needs, in practice PropertyCross is quite different. With TodoMVC each application must adhere to a detailed [specification](https://github.com/tastejs/todomvc/blob/gh-pages/app-spec.md) that results in each TodoMVC app looking and functioning in exactly the same way. In contrast, PropertyCross has a more relaxed [specification](https://github.com/tastejs/PropertyCross/tree/master/specification) which describes functionality rather than implementation detail. This is because cross-platform mobile frameworks differ considerably when it comes to how they physically render the UI layer (HTML, native, and other), and the abstraction layers they might have for code sharing between devices.

Another important difference is that TodoMVC is single platform, i.e. the web, whereas PropertyCross applications must run natively on a range of platforms (iOS, Android and Windows Phone). The way in which cross-platform mobile frameworks tackle this problem varies considerably. Some create an abstraction layer so that you write your code once, with the framework determining how best to run the app on each platform, where others allow you to share business logic, with the construction of bespoke UIs for each platform being left as a job for the developer.

Finally, while TodoMVC apps are all JavaScript (or at least compile-to-JavaScript), PropertyCross encompasses a wide range of languages including HTML5 (which accounts for around half the frameworks), C#, Ruby, ActionScript and Pascal.

Despite these differences, the common goal of helping developers make informed choices when faced with a numerous options, brings these two projects close together. 

![PropertyCross many many versions](http://i.imgur.com/pkwJm3o.png)

## PropertyCross v1.3

The 1.3 release of PropertyCross introduces a couple of new frameworks that couldn't be more different.

The first is [Delphi](http://propertycross.com/delphi/) from Embarcadero, which has a very long history; originating in 1995 as a tool for writing MS Windows applications (back in the days when app's were called applications!). Cross-platform Delphi apps are written using [Object Pascal](http://en.wikipedia.org/wiki/Object_Pascal), with the UI being rendered for Android and iOS using their own framework. 

The second new edition is [Emy](http://propertycross.com/emy/), a lightweight HTML5 framework developed by [Remi Grumeau](https://github.com/remi-grumeau) (and named after his cat!).

This release has also seen a lot of amazing behind the scenes work by James Phillpotts ([@mitserpotes](https://twitter.com/misterpotes)), who has been busy reviewing pull requests, upgrading PhoneGap Build, and adding a significant amount of automation. Managing 42 app releases, across 16 different frameworks and 3 mobile platforms is no easy task!  

## What's Next?

The HTML5 vs. Native battle will no doubt continue to rage on, and while the tech-media is busy trying to declare a winner, humble developers will continue to just get on with their job and write cross-platform apps!

At PropertyCross we have our work cut out updating various implementations to match the new iOS 7 look-and-feel, and I am sure 2014 will see quite a few new framework additions to the project.

If you are interested in contributing, why not take a look at the [list of framework requests](https://github.com/tastejs/PropertyCross/issues?labels=Framework+Request&page=1&state=open)?

