---
author: ceberhardt
title: Windows Phone App Studio - Shows Potential ...
tags: 
categories: 
summary: A couple of days ago Microsoft announced Windows Phone App Studio, a web based tool for the rapid creation of Windows Phone applications. In this blog post I take this new technology for a spin to see what it's capable of, and the interesting potential it has for creating 'personal' apps.
layout: default_post
---
A couple of days ago Microsoft announced [Windows Phone App Studio](http://blogs.windows.com/windows_phone/b/wpdev/archive/2013/08/06/making-it-easier-to-get-started-with-windows-phone-app-studio-beta-simplified-phone-registration-support-options-amp-more-payout-markets.aspx), a web based tool for the rapid creation of Windows Phone applications. Whilst the graphical app-builder idea is not a new one, the App Studio has a few interesting features, such as the ability to distribute your apps outside of the Windows Phone Marketplace. This allows you to create much more 'personal' apps.

> The idea of 'personal apps' is such a cool concept that I think the App Studio should be re-designed entirely around this purpose.

If you are interested in my experiences with this tool - read on ...

## App Studio - Hardly a Groundbreaking Idea!

Recently there has been a veritable explosion of rapid app-builder solutions. When I attended [last year's AppsWorld event](http://www.shinobicontrols.com/blog/posts/2012/10/08/shinobicontrols-and-the-apps-world-experience/), there were at least ten such companies exhibiting, including [Snappii](https://www.snappii.com/search.aspx), [AnyPresence](http://www.anypresence.com/), and [TheAppBuilder](http://www.theappbuilder.com/). It's easy to see how these solutions are quite appealing - everyone seems to want to have a presence in the App Store and Marketplaces. However, a professionally produced app will cost thousands of pounds to produce. These app-builders provide a very low cost alternative, and will often produce cross platform apps as a bonus.

I've been meaning to give these app-builder solutions a try for a long time now. It is clear that developing apps through system that don't require programming knowledge will result in some limitations, but I have no idea how limiting this approach is. The App Studio has given me an excuse to give rapid app-building a try ...

## Getting Started

To build an application all you need is your regular Microsoft account, with this, you can head over to the [App Studio](http://apps.windowsstore.com/Home/StartBuilding) and start building via the web-based interface.

On hitting the 'Create' button you are faced with a list of templates that you can use as the starting point of your application:

<img src="{{ site.baseurl }}/ceberhardt/assets/AppStudioTemplate.png" alt="App Studio Templates"/>

These templates cover some of the common application types, restaurant apps, company apps and catalogs. And some less obvious ones such as personal photo albums, file reviews and wedding invitations. It is clear that Microsoft want you to use this tool for creating applications that are very personal ... apps that you just use yourself or share with a few friends rather than adding them to the Marketplace.

Initially when I started looking at the App Studio my idea was to try and create a [PropertyCross](http://propertycross.com/) implementation. If you haven't come across this project before, PropertyCross is an open source project which seeks to compare the many cross platform frameworks by showing the same, simple, property finder app implemented with each framework. It is a great benchmark for comparing frameworks.

Unfortunately it quickly became obvious that this was not going to be possible. The App Studio allows you to create apps that have pages of static or dynamic data that the user can navigate between. However, it doesn't allow you to add *any* conditional logic to your applications at all. As a result, it is very limited in terms of the types of app you can create with this tool.

Time for a re-think ... instead I decided to re-implement [SandwichFlow](http://www.scottlogic.com/blog/2011/05/16/metro-in-motion-5-sandwichflow.html), a sandwich recipe app, using the App Studio.

## Adding Content

Once you select a template, you proceed to the next screen where you add content. The App Studio has the concept of datasource, which can be RSS feeds, or static data, which you add directly via the web interface:
 
<img src="{{ site.baseurl }}/ceberhardt/assets/SandwichFlowContent.png" alt="App Studio Content Editing"/>

At this point you can add / remove datasources and add pages that render this data. I did find that regardless of which template you select at the start, you basically end up with the same structure. In other words, the templates provide suggestions for how you might use App Studio, however, each results in the same 'panorama' style application.

Adding content is pretty painful, the web-based form you are presented with isn't ideal if you have to input large quantities of data:

<img src="{{ site.baseurl }}/ceberhardt/assets/AppStudioAddData.png" alt="App Studio Content Editing"/>

After adding around four sandwiches I got sick of the process and gave up on it!

On a serious note, this does have a significant impact on the usefulness of this tool. The templates are encouraging people to use it for catalogues, so I would expect to see a more rapid data input interface at the very least. Or better still, Excel / CSV data import.

## Changing Layout

Through the 'App Content' process you can also configure the layout of the various 'detail' pages that the user will see when they tap on content items:

<img src="{{ site.baseurl }}/ceberhardt/assets/AppStudioLayout.png" alt="App Studio Changing Layout"/>

This screen gives you a decent preview of that area of your application.

On that note, the web preview is a a very useful utility, allowing you to test your application without the need to request a build then re-deploy on your device. However, the web preview is currently restricted in that it only shows the current context, i.e. the item that is currently being edited. It would be much more useful if you could see a web simulation of your entire application.

I'd have to say that I did find the whole layout / content generation to be pretty confusing. It was not clear to me where the content I was adding would appear within the overall app structure. Furthermore, the reference to datasource and databindings feels pretty low-level for this type of tool. Again, a more complete preview would help people understand the impact of the changes they are making.

## Styling

Once your content and layout is complete you move onto styling your app. At this point you can select a few colours, your app tile, backgrounds and a few other features:

<img src="{{ site.baseurl }}/ceberhardt/assets/AppStudioStyle.png" alt="App Studio Styling"/>

Again, this is pretty limited, you cannot change fonts for example.

The changes you make are reflected in the web preview, which is a welcome feature. 

## Generating Your App

Once your app is styled, you can review, then generate the final application.

Generation typically takes a few minutes, and you are sent an email once it is complete. Once generated you are free to distribute your application.

<img src="{{ site.baseurl }}/ceberhardt/assets/AppStudioGenerate.png" alt="App Studio Generate"/>

Here is where it gets interesting, there are a few different ways you can distribute your app:

1. **Download the XAP** - You can download the generated XAP file and deploy / distribute as you would any other Windows Phone App.
2. **Download the Source** - You can download the generated source. I was expecting the generated code to be quite generic and data driven, i.e. an engine for rendering templated content. Surprisingly it is not. The app is structured using MVVM, with classes named to reflect your domain objects. Clearly Microsoft are hoping that a developer can take the generated code and extend it further.  
3. **Share by Email** - This is the most interesting distribution option. Using this distribution model you can share the app with anyone you wish, without the need to submit it to the Marketplace. When you share the app they receive some simple instructions which involve installing a certificate, then a one-click install of the app.

## The Final App

So, what did the final SandwichFlow app look like?

I've recorded a brief video of the app which you can see on [YouTube](https://www.youtube.com/watch?v=nZMGrKI91ck). As you can see, it is pretty basic and uninspiring. Because I got bored of adding static data, I also included an RSS feed from my blog. The finished app works just as you would expect and looks very similar to the web-based previews. However, I did have an issue with the splashscreen image dimensions, which you can see in the video, there are no indications or hints on the web interface regarding what the dimensions should be.

As a developer, I of course know the correct dimensions, and could easily fix this, although i do wonder whether the target audience for App Studio would have this knowledge? 

## Make 'personal apps' central to App Studio 

The App Studio is in beta, so I am going to avoid picking too many holes in it. As a general observation, the idea of a web-based app-generator is not a new one, and existing commercial offerings have the cross-platform advantage.

The App Studio interface is a bit confusing (and that's coming from a developer!), and the lack of a full app preview means that you have to run a full generation in order to test the structure of your application. I don't think it would take much effort on Microsoft's part to remedy these issues.

Personally I think the most powerful and interesting feature of the App Studio is the way it allows you to create personal apps. It is such a cool concept that I think the App Studio should be re-designed entirely around this purpose.

The templates should be changed so that they do not just pre-populate the layout / content. Instead, each template should produce a distinct 'type' of application. For example; A 'Photo Album' template that makes it very easy for people to bulk-upload images, with an application that allows pinch-zoom and social media sharing. A "My Trip", template which allows you to share memories of a holiday, with a travel-blog style interface.

If Microsoft make it really easy to create and share personal apps, they might be onto a winner. At the moment App Studio tries to be too generic and versatile. Unfortunately the end result is an interface that makes it hard to create a decent application of any type.
