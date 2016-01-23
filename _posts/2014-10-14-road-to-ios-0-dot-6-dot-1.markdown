---
layout: post
title: "Road to iOS Series 8"
date: 2014-10-14 09:14:19 +1000
comments: true
header-img: "img/post-bg-07.jpg"
categories: [ios]
---

The Nib file

- User interface objects(Visual elements)
- File Owner, placeholder, a controller object that is responsible for contents of nib file.
	- Usually you will want to connect a controller to the nib file, so that controller's **view** can be connected with nib's top-level object
	- However, if you want to break a view into different nib components,  then assemble them together in one controller, you can leave this File Owner blank, and load all nibs in your controller.
- First Responder, placeholder, the first object in your app's responder chain.
	- The UIKit framework automatically set first responder for you

<!--more-->

Nib Object Life Cycle :

The nib-loading code instantiates the objects, configures them, and reestablishes any inter-object connections that you created in your nib file.
When you use methods of **NSBundle** to load and instantiate the objects in nib file, the nib-loading code does following:

1. Load the archived nib file into memory

2. Unarchive the nib file, instantiates the objects
	- In iOS, any object that conforms to the NSCoding protocol is initialized using the **initWithCoder:**. This includes all subclasses of UIView and UIViewController whether they are part of the default Xcode library or custom classes you define.
	- The reason using `initWithCoder` is that they are available in Xcode library (System knows how to instantiate)
	- Custom objects other than those described in the preceding steps receive an **init** message. (Which may lead to init chain)


3. It reestablishes all connections (actions, outlets, and bindings) between objects in the nib file.
	- Outlet connections
		- The nib-loading code uses the **setValue:forKey:** to reconnect each outlet
		- Setting an outlet also generates a **key-value observing (KVO)** notification for any registered observers.
			- That's why if sometimes you forget to connect oulet in your code to nib file, it will prompt key-value pair not matched !
	- **Action connections**
		- In iOS, the nib-loading code uses the **addTarget:action:forControlEvents:** method of the UIControl object to configure the action. If the target is nil, the action is handled by the responder chain.


4. It sends an awakeFromNib message to the appropriate objects in the nib file
	- In iOS, this message is sent only to the interface objects that were instantiated by the nib-loading code. It is not sent to File’s Owner, First Responder, or any other placeholder objects.
	- That means, you can also put view init code in awakeFromNib in your custom view, since interface objects are already instantiated.

{% highlight objective-c %}
- (void)awakeFromNib
{
	if (self)
	{
		// do view init
		self.button.layer setBorderWidth:2.0f];
		...
	}
}
{% endhighlight %}

<br>

Be mindful that if you need to configure the objects in your nib file further at load time, the most appropriate time to do so is **after your nib-loading call returns**. At that point, all of the objects are created, initialized, and ready for use.

{% highlight objective-c %}
- (void)loadView
{
	// load nib file
	NSArray *nibs = [[NSBundle mainBundle] loadNibNamed:@"yourNib"
							       owner:self
							     options:nil];
	YourOwnView *first = [nibs firstObject];

	// configure the view
	...

	self.view = first;
}
{% endhighlight %}

------------

Load custom view (nib file + UIView file) :

- Steps for loading custom view
	- create a view using interface builder(i.e. create xib file)
	- create your view controller for this xib (if don't leave File Owner empty)
	- create subclass of **UIView**, e.g. YourOwnView
	- **change the custom class view attribute for xib to YourOwnView**
	- implement **awakeFromNib** to custom initialise your elements in nib
	- **call loadNibNamed: at appropriate time in your controller to let nib-loading cycle go through**

A common mistake is to call **initWithFrame** in controller, which only affects those view created pure programmatically. It has nothing to do with Your nib file. You need to call **loadNibNamed** in controller at least once to give your nib file a chance to finish nib-loading life cycle !

![ 805 585 Nib Anatomy ](/images/ios/nib_anatomy.png)

------------

### Reference
1. Stanford iOS 7 development
2. [Mac Developer Library](https://developer.apple.com/library/mac/navigation/)
3. [Manage Resources in OSX](https://developer.apple.com/library/ios/documentation/cocoa/conceptual/loadingresources/cocoanibs/cocoanibs.html)
