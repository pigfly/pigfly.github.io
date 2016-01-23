---
layout: post
title: "Road To iOS Series 7"
date: 2014-10-12 12:14:19 +1000
comments: true
header-img: "img/post-bg-05.jpg"
categories: [ios]
---

View Controller Essentials :

- An instance of a subclass of UIViewController
- It manages a view hierarchy
- It's responsible for creating view objects, handling events

<!--more-->

View in View Controller :

- It's the *root* of view controller's view hierarchy
- Two ways of creating its view hierarchy :
	- Programmatically, override the UIViewController method loadView
	- Interface Builder, loading a NIB file

Create view programmatically :

In your own view controller class(which can be created by inheriting UIViewController), override the loadview:

{% highlight objective-c %}
- (void)loadView
{
	// 1. specify the view's frame
	// could use other Rect like full screen
	// [UIScreen mainScreen].bounds
	CGRect firstFrame = CGRectMake(160, 200, 100, 105);

	// 2. init an instance of view
	YourOwnView *firstView = [[YourOwnView alloc] initWithFrame:firstFrame];

	// 3. configure the  view
	firstView.backgroundColor = [UIColor greenColor];

	// 4. set the view in view controller to this view
	// sometimes you use addSubView if you have multiple views
	// e.g. [firstView addSubview: secondView];
	self.view = firstView;
}
{% endhighlight %}

However, the code above is for creating view hierarchy in view controller only,
it has nothing to do with the actual view appeared on the screen. To connect a
view in view controller with screen, you need to use UIwindow's setRootViewController.

{% highlight objective-c %}
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
	self.window = [[UIWindow alloc] initWithFrame:[[UIScreen mainScreen] bounds]];

	// init your view controller
	YourOwnViewController *vc = [YourOwnViewController alloc] init];

	// set the window's root controller
	self.window.rootViewController = vc;

	self.window.backgroundColor = [UIColor whiteColor];
	[self.window makeKeyAndVisible];
	return YES;
}
{% endhighlight %}

Create view use XIB :

- Steps for creating view for view controller using XIB:
	- create a view using interface builder(i.e. create xib file)
	- create your view controller for this xib
	- make sure your xib's name is named the same as your view controller (xcode uses this to automatically load nib file i.e. init the view)
	- connect your controller to your xib(e.g. File's owner, IBOutlet, IBAction)

{% highlight objective-c %}
@interface YourOwnViewController ()

// IBOutlet is typedef as void, it's only for xcode to identify the specific element in xib
// Pay attention to the weak decorator, follow parent-child rule
@property (nonatomic, weak) IBOutlet UIDatePicker *datePicker;

@end

// IBAction is typedef as void, it's only for xcode to identify the specific action in xib
- (IBAction)addReminder:(id)sender
{
    NSDate *date = self.datePicker.date;
    NSLog(@"Setting a reminder for %@", date);
}
{% endhighlight %}

After you get your xib and view controller done, you may now **connect these two together**.

- Connect to File's Owner in xib
	- use identity inspector to change the view controller class to your own class
- Connect outlet and actions
	- right click on File's Owner, drag-and-drop those connections between your controller and xib

------------

UIViewController Initializer :

- Designated Initializer: `initWithName:bundle:`
- Simply call alloc and init
- Sending init to a view controller calls initWithName:bundle: and passes nil for both arguments
- When a view controller is initialized with *nil* as its NIB name, it searches for a NIB file with the name of the class

------------

UIViewController Lifecycle :

See previous post for more details: [View Controller Lifecycle](http://pigfly.github.io/blog/2014/09/01/road-to-ios-0-dot-2/)

------------

### Reference
1. Stanford iOS 7 development
2. [Mac Developer Library](https://developer.apple.com/library/mac/navigation/)
3. [Big Nerd Ranch](http://www.bignerdranch.com/)
