---
author: ceberhardt
title: Simulating Accelerometer and Location data for iOS
categories: 
summary: This blog post looks at how to simulate accelerometer and location data so that you can test iOS apps without the need for a physical device. The simulated data is provided by an interactive UI which allows you to rotate the phone and mark paths on a map which can then be replayed.
layout: default_post
---


This blog post looks at how to simulate accelerometer and location data so that you can test iOS apps without the need for a physical device. The simulated data is provided by an interactive UI which allows you to rotate the phone and mark paths on a map which can then be replayed.

<img src="{{ site.baseurl }}/ceberhardt/assets/SimulatorEnhancements.jpg"/>

The drives for each of my computers are littered with half-finished concepts and experiments, moments of inspiration that I either tire of or decide are of no real value. However, more often than not, these ideas are interrupted by 'real' work and are never returned to. This is why I quite like travelling - the hours spent in airports and on aeroplanes can be some of my most productive. The lack of internet and other distractions means that I tend to finish a project rather than start ten new ones! 

One problem I have been thinking of tackling for a while is how to simulate hardware sensor input. A recent trip gave me the perfect excuse (and time) to try and tackle this problem.

If you have written an app that uses accelerometer input, you will no doubt have encountered the issue that the iOS simulator reports a reading of `(0.0, 0.0, 0.0)`, no matter which angle you hold your MacBook! Your only route to testing this type of app is to use a physical device.

With location, Xcode does allow you to specify the location via a menu option:

<img src="{{ site.baseurl }}/ceberhardt/assets/xcodeLocationSelector.png"/>

This is useful for apps that perform spacial searches, e.g. find restaurants in my ares, but if you are writing an app that relies on location changes, such as a running app, this isn't much good.

## Faked Location Data

Applications receive notifications of location change via the `CLLocationManager`. This involves creating an instance of the manager class, and assigning a delegate to receive updates:

{% highlight objc %}
- (id)init {
  self.locationManager = [[CLLocationManager alloc] init];
  self.locationManager.delegate = self;
}

- (void)locationManager:(CLLocationManager *)manager 
                didUpdateToLocation:(CLLocation *)newLocation
                       fromLocation:(CLLocation *)oldLocation {
                          
  // do something with the new location information
}
{% endhighlight %}

In order to simulate location data we have to find a way to invoke the `locationManager:didUpdateToLocation:fromLocation:` delegate method with simulated data. The `CLLocationManager` is a class supplied by the Cocoa framework, so how do we inject new behaviours into it?

Fortunately the Objective-C runtime allows you to dynamically change the behaviour of *any* class, via the functions `class_addMethod` and `class_replaceMethod` which are defined within `objc/runtime.h`. The process of changing the behaviour of an existing class by modifying its methods has been given the rather funky name 'swizzling'. 

I've wrapped up this concept into a simple utility class:

{% highlight objc %}
＠interface CESwizzleUtils : NSObject

/**
 Swizzle the given method so that it is replaced with a method
 named 'override_methodName'.
 */
+ (void)swizzleClass:(Class)class method:(NSString*)methodName;

＠end

＠implementation CESwizzleUtils

+ (void)swizzleClass:(Class)class method:(NSString*)methodName {
  
  SEL originalSelector = NSSelectorFromString(methodName);
  SEL newSelector = NSSelectorFromString(
       [NSString stringWithFormat:＠"%＠%＠", ＠"override_", methodName]);
  
  Method originalMethod = class_getInstanceMethod(class, originalSelector);
  Method newMethod = class_getInstanceMethod(class, newSelector);
  
  method_exchangeImplementations(originalMethod, newMethod);
}

＠end
{% endhighlight %}

Using `swizzleClass:method:` you can swap out the implementation of a method `foo` with a method `override_foo` which can be implemented in a category.

Putting this to use, we can swizzle the setter for the location managers delegate:

{% highlight objc %}
[CESwizzleUtils swizzleClass:[CLLocationManager class]
                      method:＠"setDelegate:"];
{% endhighlight %}

Then we can define a category on `CLLocationManager` that implements the swizzled method:

{% highlight objc %}
＠implementation CLLocationManager (Enhancements)


-(void)override_setDelegate:(id<CLLocationManagerDelegate>)delegate {
  // do something with the reference to self or delegate
  [self override_setDelegate:delegate];
}

＠end
{% endhighlight %}

The above effectively intercepts the setter for the delegate property, and as a result, as soon as a delegate is assigned to *any* location manager instance, the above method is called. The call to `[self override_setDelegate:delegate]` ensures that `setDelegate:` is called on the location manager, providing the default behaviour.

To make use of the above I created a simple singleton that holds references to location manager instances.

So, now that a reference for the location manager has been obtained, how do we simulate location changes? Within the same category I added the following method:

{% highlight objc %}
- (void)simx_didUpdateLocation:(CLLocation *)location {
  id delegate = self.delegate;
  
  if ([delegate respondsToSelector:
                   ＠selector(locationManager:didUpdateLocations:)]) {
    [delegate locationManager:self
           didUpdateLocations:＠[location]];
  }
  
  if ([delegate respondsToSelector:
      ＠selector(locationManager:didUpdateToLocation:fromLocation:)]) {
    [delegate locationManager:self
          didUpdateToLocation:location
                 fromLocation:nil];
  }
}
{% endhighlight %}

With the above, we can send location updates to a location manager's delegate as follows:

{% highlight objc %}
[locationManager simx_didUpdateLocation:location];
{% endhighlight %}

I'll look at where the simulated location data comes from shortly. In the meantime, the final thing I discovered was that the while I was able to simulate my own location 'events', the delegate was still being called with the 'real' locations.

Inspecting the stack trace I spotted that an internal method,  `onClientEventLocation` was being called just before the delegate method:

<img src="{{ site.baseurl }}/ceberhardt/assets/StackTraceLocation.png"/>

Suppressing this behaviour is as simple as swizzling the method, but without calling the original implementation:

{% highlight objc %}
- (void)override_onClientEventLocation:(id)foo {
  // no-op - this suppresses location change events that are raised
  // by CLLocationManager
}
{% endhighlight %}

This stops the real location updates reaching the delegate.

## Faked Motion Data

The `CMMotionManager` provides access to accelerometer and gyro data, however, it's API differs from the `CLLocationManger`. Rather than using a delegate to provide change notifications, you can obtain the current data via the `deviceMotion` property, or have updates delivered to a queue:

{% highlight objc %}
[self.motionManager startAccelerometerUpdatesToQueue:self.queue withHandler:
 ^(CMAccelerometerData *accelerometerData, NSError *error) {
     // do something with the new data …
 }];
{% endhighlight %}

This requires a slightly different approach! 

Again, swizzling is used to intercept the `startAccelerometerUpdatesToQueue:withHandler` method, but this time a reference to the handler is stored so that it can be invoked directly with the simulated data:

{% highlight objc %}
#define HANDLER_IDENTIFIER ＠"accelerometerHandler"

＠implementation CMMotionManager (Enhancements)

- (void)simx_accelerometerUpdate:(CMAccelerometerData *)accelerometerData {
  CMAccelerometerHandler handler =
                     objc_getAssociatedObject(self, HANDLER_IDENTIFIER);
  handler(accelerometerData, nil);
}

-(void)override_startAccelerometerUpdatesToQueue:(NSOperationQueue *)queue
                                withHandler:(CMAccelerometerHandler)handler {

  objc_setAssociatedObject(self, HANDLER_IDENTIFIER,
                                      handler, OBJC_ASSOCIATION_RETAIN);
  
  [[CEMotionEnhancements instance] addManager:self];
}

＠end
{% endhighlight %}

**NOTE:** The use of `objc_getAssociatedObject` and the corresponding set function are due to the fact that you cannot add instance variables to a class via categories.

The `simx_accelerometerUpdate` method is used to provide simulated data to the handler. However, there is one more obstacle - the `acceleration` property of `CMAccelerometerData` is readonly, hence there is no way to provide faked data.

The solution to this problem? swizzling! (of course)

The code below adds a `simx_setAcceleration` method to `CMAccelerometerData` which allows us to inject the fake data. The swizzled `override_acceleration` method replaces the getter implementation of this readonly property to return the fake data:

{% highlight objc %}
#define ACCELERATION_IDENTIFIER ＠"accelerometerHandler"

＠implementation CMAccelerometerData (Enhancements)

- (void) simx_setAcceleration:(CMAcceleration)acceleration {
  NSValue *value = [NSValue value:&acceleration
                     withObjCType:＠encode(CMAcceleration)];
  objc_setAssociatedObject(self, ACCELERATION_IDENTIFIER,
                            value, OBJC_ASSOCIATION_RETAIN);
}

- (CMAcceleration)override_acceleration {
  NSValue *value = objc_getAssociatedObject(self, ACCELERATION_IDENTIFIER);
  CMAcceleration acc;
  [value getValue:&acc];
  return acc;
}

＠end
{% endhighlight %}

That's the readonly property foiled!

## Simulating Data

So far this blog post has shown that a liberal dose of swizzling can be used to inject faked motion and location data into an iOS app. But how to create this data in the first place?

I decided that the most flexible approach was to provide data in JSON format via a local web-server. The iOS app polls a localhost address requesting data 10 times a second. When the data is received it is parsed and the methods described above are used to inject the results.

In order to create a quick, prototypes UI for simulating data, I opted for a simple web-app using node as the server. Practically this would probably be better as a Mac app, but personally I find myself able to prototype more quickly with web technologies.

I'll not cover the web-app in detail, the code is all on [GitHub](https://github.com/ColinEberhardt/SimulatorEnhancements-Server), so you can poke around at your leisure. Instead I'll give a very brief overview of the app structure.

The server code is written using [node](http://nodejs.org/), with [express](http://expressjs.com/) providing basic web-server functionality. The app has one URL, `localhost/update` which is used by the web UI to update the current simulated data, the second, `localhost/data` is used by the iOS code to fetch the current state. 

The web UI uses [Knockout](http://knockoutjs.com/) to create a model of the simulator data. The Knockout 'observable' concept is used to detect changes in this data (with a bit of throttling thrown in), and the resultant JSON sent to the  `localhost/update` URL via HTTP POST.

The accelerometer view is a very simple rendering of a squashed cube using [three.js](http://threejs.org/):

<img src="{{ site.baseurl }}/ceberhardt/assets/AccelerometerSimulation.png"/>

I must admit, 3D graphics is not my strong-point! The code is a pretty simple adaptation of one of the examples provided by the [awesome mrdoob](http://mrdoob.github.io/three.js/examples/canvas_geometry_cube.html).

The location view uses a Google map, with the JavaScript API being used to record and replay paths:

<img src="{{ site.baseurl }}/ceberhardt/assets/LocationSimulation.png"/>

Both of these views update the underlying model, with the updated data being sent to the node back-end. The iOS simulator, which is polling the same node app, picks up these changes. Somewhat brute-force, but simple and effective!

## Conclusion

I embarked on this little project out of my own frustration at not being able to develop iOS apps without physical hardware. In its current state it is very much a proof-of-concept, only a small number of the `CMMotionManager` and `CLLocationManager` methods are intercepted. 

I'm very interested to hear people's thoughts and opinions on this approach, is this something you could see yourself using?

The code is all on GitHub, where you can find the [iOS simulation code](https://github.com/ColinEberhardt/SimulatorEnhancements), and the [server](https://github.com/ColinEberhardt/SimulatorEnhancements-Server) that simulates the input data. Feel free to dig around!

Regards, Colin E.
