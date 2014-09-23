---
author: ceberhardt
title: "MVVM, Swift and ReactiveCocoa - It's all good!"
categories: 
tags:

summary: "This blog post looks out how Swift makes the combination of ReactiveCocoa and MVVM even better ..."
layout: default_post
---

Around one month ago my [two-part tutorial series](http://www.raywenderlich.com/74106/mvvm-tutorial-with-reactivecocoa-part-1) on how to use the MVVM pattern with ReactiveCocoa was published on Ray Wenderlich's website. Unfortunately just before the publication date Apple launched the Swift beta, a language which is much better suited to functional programming than Objective-C.

I've ported the app to Swift, and the code looks much more elegant as a result. This blog post shares the updated code and a few observations.

The complete Swift version of this app can be found on [GitHub](https://github.com/ColinEberhardt/ReactiveSwiftFlickrSearch). Here are a couple of screenshots of the app that searches for pictures on the popular photo sharing site, Flickr.

<img src="{{ site.baseurl }}/ceberhardt/assets/MVVMSwift/FinishedApp.png"></img>

This blog post isn't going to describe the workings of this app in detail. For that I'd refer you to the original tutorials. Instead this blog post highlights some of the significant differences between the Swift and Objective-C implementations.

## A Quick MVVM Refresher

This app uses the Model-View-ViewModel ([MVVM](http://en.wikipedia.org/wiki/Model_View_ViewModel)) pattern. At the core of this pattern is the ViewModel which is a special type of model that represents the UI state of the application. It contains properties that detail the state of each and every UI control, for example the current text for a text field or whether a specific button is enabled. It also exposes the actions that the view is able to perform, for example button taps or gestures.

<img src="{{ site.baseurl }}/ceberhardt/assets/MVVMSwift/MVVMPattern.png"></img>

Looking at the MVVM pattern specifically from the perspective of iOS development, the View is composed of the ViewController plus its associated UI (whether that is a nib, storyboard or constructed though code): 

<img src="{{ site.baseurl }}/ceberhardt/assets/MVVMSwift/MVVMReactiveCocoa.png"></img>

With MVVM your Views should be very simple, doing little more than just reflecting the current UI state.

ReactiveCocoa holds a special role in implementing MVVM applications, providing a simple mechanism for synchronising Views and their associated ViewModels.

## ReactiveCocoa and Swift

Swift was designed to interop with Objective-C, and as a result, you should be able to use ReactiveCocoa directly within a Swift application. 

However the Swift compiler borks on the 'and', 'or' and 'not' methods (despite the fact that they are not reserved keywords). To side-step this issue, I am using [Yusef Napora's fork of ReactiveCocoa](https://github.com/yusefnapora/ReactiveCocoa/tree/de3c9a76666b1bf847f3f50df6a3791035defd9a) which simply changes these methods to AND, OR and NOT.

With that out of the way, let's get onto the good stuff ...

## Signals in Swift

The block-based ReactiveCocoa API becomes a closure-based API when bridged to Swift. As a quick example, the following Objective-C example subscribes to the `rac_textSignal` on a text field, logging the current length:

{% highlight objective-c %}
[self.searchTextField.rac_textSignal subscribeNext:^(id x) {
  NSString *text = (NSString *)x;
  NSLog(text);
}];
{% endhighlight %}

Which logs the following as you type:

<pre>
0
1
2
3
4
5
</pre>

However, with Objective-C you can change the block signature from `id` to `NSString` removing the need to cast explicitly:

{% highlight objective-c %}
[self.searchTextField.rac_textSignal subscribeNext:^(NSString *text) {
  NSLog(text);
}];
{% endhighlight %}

A Swift equivalent for the code above is the following:

{% highlight csharp %}
searchTextField.rac_textSignal().subscribeNext {
  (next:AnyObject!) -> () in
  if let text = next as? String {
    println(countElements(text))
  }
}
{% endhighlight %}

Unfortunately, as is often the case, when Swift and Objective-C meet, things get a little messy! We cannot perform an implicit cast, Swift sticks rigidly to the required function type of `(AnyObject!) -> ()`.

You can simplify a little by dropping the `if-let` condition. You know that this `AnyObject` is always going to be a string:

{% highlight csharp %}
searchTextField.rac_textSignal().subscribeNext {
  (next:AnyObject!) -> () in
  let text = next as String
  println(countElements(text))
}
{% endhighlight %}

However, this is still more complicated than the Objective-C counterpart.

Fortunately there is a solution to this problem! An extension method can be added to `RACSignal` that uses the type information from a passed function in order to perform the required cast:

{% highlight csharp %}
extension RACSignal {  
  func subscribeNextAs<T>(nextClosure:(T) -> ()) -> () {
    self.subscribeNext {
      (next: AnyObject!) -> () in
      let nextAsT = next as T
      nextClosure(nextAsT)
    }
  }
}
{% endhighlight %}

The above uses generics to determine the required cast, where the generic type parameter `T` is inferred from the supplied function / closure. This leads to a much more pleasing syntax:

{% highlight csharp %}
searchTextField.rac_textSignal().subscribeNextAs {
  (text:String) -> () in
  println(countElements(text))
}
{% endhighlight %}

Much better!

Interestingly because this extension method infers the type parameter from the supplied function, you cannot use a more shorthand closure syntax where the parameter type is excluded:

{% highlight csharp %}
searchTextField.rac_textSignal().subscribeNextAs {
  (text) -> () in
  println(countElements(text))
}
{% endhighlight %}

In the code above, the compiler cannot infer that the `text` parameter is a `String`.

The project on GitHub includes extension methods for `filter`, `map`, `doNext` and a number of other common operations.

## Bindings

ReactiveCocoa has a few macros that allow you to bind properties to signals. As an example, the following binds the `searchText` property of the ViewModel to a text field's `rac_textSignal` property. Whenever the text field is updated, this signal emits the current text, with the binding updating the ViewModel property.

{% highlight csharp %}
RAC(self.viewModel, searchText) = self.searchTextField.rac_textSignal;
{% endhighlight %}

Swift does not support macros, so you cannot use `RAC` from ReactiveCocoa.

Once again [Yusef Napora comes to the rescue](http://napora.org/a-swift-reaction/)! The following struct and operator are a direct replacement of the RAC macro:

{% highlight csharp %}
// a struct that replaces the RAC macro
struct RAC  {
  var target : NSObject!
  var keyPath : String!
  var nilValue : AnyObject!
  
  init(_ target: NSObject!, _ keyPath: String, nilValue: AnyObject? = nil) {
    self.target = target
    self.keyPath = keyPath
    self.nilValue = nilValue
  }
  
  func assignSignal(signal : RACSignal) {
    signal.setKeyPath(self.keyPath, onObject: self.target, nilValue: self.nilValue)
  }
}

operator infix ~> {}
@infix func ~> (signal: RACSignal, rac: RAC) {
  rac.assignSignal(signal)
}
{% endhighlight %}

This has the added advantage that it puts the property, which is the 'target' on the right-hand side, which feels more logical:

{% highlight csharp %}
searchTextField.rac_textSignal() ~> RAC(viewModel, "searchText")
{% endhighlight %}

Fortunately the `RACObserve` macro is much easier to replicate:

{% highlight csharp %}
func RACObserve(target: NSObject!, keyPath: String) -> RACSignal  {
  return target.rac_valuesForKeyPath(keyPath, observer: target)
}
{% endhighlight %}

## Cool stuff!

One of my favourite parts of the original app looks even better in Swift. In order to reduce load on the Flickr API the app only fetches the photo metadata (comment & favourite count) when a photo has been visible for one second. This is achieved within the ViewModel that backs each image with the following code:

{% highlight csharp %}
// a signal that emits events when visibility changes
let visibleStateChanged = RACObserve(self, "isVisible").skip(1)

// filtered into visible and hidden signals
let visibleSignal = visibleStateChanged.filter { $0.boolValue }
let hiddenSignal = visibleStateChanged.filter { !$0.boolValue }

// a signal that emits when an item has been visible for 1 second
let fetchMetadata = visibleSignal.delay(1).takeUntil(hiddenSignal)

fetchMetadata.subscribeNext {
  // fetch the meta-data
}
{% endhighlight %}

The above code observes the `isVisible` property of the ViewModel then generates signals that emit events when the photo is shown or hidden. These are then combined, with a one second delay in order to produce the final signal that is used to fetch the metadata.

Awesome!

## Conclusions

Migrating this app from Objective-C to Swift was basically a fun exercise that I wanted to share. There are a few areas that cause some friction, but in general I think Swift will make this style of programming so much easier in future.

Keep your eye on [this branch](https://github.com/ReactiveCocoa/ReactiveCocoa/pull/1382) for the official re-implementation of ReactiveCocoa with Swift, this will no doubt tidy up some of the firction that necessarily exists when bridging Objective-C APIs into Swift.

Regards, Colin E.



