---
layout: post
title: "Road To iOS Series 3"
date: 2014-09-01 19:28:13 +1000
comments: true
header-img: "img/post-bg-01.jpg"
categories: [ios]
---

## NSNotification

### What is Notification
- It's a message **sent to one or more observing objects** to **inform** them of **an event** in a program
- a way for an object that initiates or handles a program event to communicate with any number of objects that want to know about that event.
- A blind structured way of communication (**notification doesn’t have to know what those observers are**)
- A broadcast radio station model in MVC
- A very similar idea like the **event listener model** in Java

<!--more-->

### How does Notification work
- an object post a notification to notification center
- notification center broadcast the event
- event goes to the observer who registers itself at notification center

![ UIViewController Lifecycle ](/images/ios/notificationcenter.png)

### Benefit of Broadcast Model in Notification

> The object sending (or posting) the notification doesn’t have to know what those observers are.
> Notification is thus a powerful mechanism for attaining coordination and cohesion in a program. It reduces
> the need for strong dependencies between objects in a program (such dependencies would reduce the
> reusability of those objects).

<br>
-----------

### Get default notification
Each running Cocoa program has a default notification center. You typically don’t create your own.

{% highlight objective-c %}
[NSNotificationCenter defaultCenter]
{% endhighlight %}

### Register at notification center
Register observer(yourself) at notification center if you want to listen to broadcast, In this example, we are listening to system broadcast(UIContentSizeCategoryDidChangeNotification)

{% highlight objective-c %}
// the oberserver here is controller itself
// object : whos change you're interested in. Nil, meaning anyone can send notification to controller
// name is the name of the station
// selector: once you receive notification, what you will do
[[NSNotificationCenter defaultCenter] addObserver:self
                                         selector:@selector(preferredFontsChanged:)
                                             name:UIContentSizeCategoryDidChangeNotification
                                           object:nil];

// the method which you create to handle the recieved notification
-(void)preferredFontsChanged:(NSNotification *)notification
{
    [self usePreferredFonts];
}

-(void)usePreferredFonts
{
...
}
{% endhighlight %}

### Remove observer at notification center
Failure to remove yourself can sometimes result in crashers.

{% highlight objective-c %}
[[NSNotificationCenter defaultCenter] removeObserver:someObserver];
// or
// use this for best practise, since you may listen to several stations at same time
// simply remove yourself as observer will result in failure of listening to other stations.
[[NSNotificationCenter defaultCenter] removeObserver:self name:UIContentSizeCategoryDidChangeNotification object:nil];
// or put the dealloc in viewWillDisappear
- (void)dealloc
{
// be careful in this method! can’t access properties! you are almost gone from heap!
    [[NSNotificationCenter defaultCenter] removeObserver:self];
}
{% endhighlight %}

<br>
----------

### Reference
1. Stanford iOS 7 development
2. [Mac Developer Library](https://developer.apple.com/library/mac/navigation/)
3. [Cocoa Core Competencies](https://developer.apple.com/library/mac/documentation/General/Conceptual/DevPedia-CocoaCore/Notification.html#//apple_ref/doc/uid/TP40008195-CH35)
