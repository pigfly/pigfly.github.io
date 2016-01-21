---
layout: post
title: "Road To iOS Series 2"
date: 2014-09-01 12:36:03 +1000
comments: true
header-img: "img/post-bg-05.jpg"
categories: [ios]
---

## View Controller Lifecycle (UIViewController_Class)

### Overview
1. `awakeFromNib`, nib here means storyboard
2. outlets get set
3. `viewDidLoad`
4. `viewWillLayoutSubviews:` and `viewDidLayoutSubviews:`
5. `viewWillAppear:` and `viewDidAppear:` etc. (both repeatedly)
6. `didReceiveMemoryWarning`

<!--more-->

### awakeFromNib
- `- (void)awakeFromNib`
- Sent to all objects that come out of a storyboard (including your Controller)
- Happens **before** outlets are set! (i.e. before the MVC is “loaded”)
- **After** an object receives an awakeFromNib message, it is guaranteed to have all its outlet instance variables set
- Anything that would go in your Controller’s init method would have to go in awakeFromNib

{% highlight objective-c %}
- (void)setup{}; //dosomething which can’t wait until viewDidLoad
- (void)awakeFromNib { [self setup]; }
{% endhighlight %}

### viewDidLoad
- Only get called **once**
- **After** instantiation and outlet-setting, viewDidLoad is called
- **Before** the actual screen shows up
- Geometry of your view (its bounds) is **not** set yet!
- A fantistic place for init. Better than your controller init. for your outlet now is set!

{% highlight objective-c %}
@interface ViewController ()
@property (weak, nonatomic) IBOutlet UIButton *outlineButton;
@end

-(void)viewDidLoad
{
	// always call super for letting superclass init.
    [super viewDidLoad];
	// Do any additional setup after loading the view, typically from a nib.

	// because we are not textview, we have to convert title to mutablestring
    // then set the title
    NSMutableAttributedString *title = [[NSMutableAttributedString alloc] initWithString:self.outlineButton.currentTitle];
    [title setAttributes:@{ NSStrokeWidthAttributeName : @5,
                            NSStrokeColorAttributeName : self.outlineButton.tintColor}
                   range:NSMakeRange(0, title.length)];

    [self.outlineButton setAttributedTitle:title forState:UIControlStateNormal];
}
{% endhighlight %}

### viewWillLayoutSubviews: and viewDidLayoutSubviews:
- Called any time a view’s frame changed and its subviews were thus re-layed out. e.g. autorotation
- Put Geometry codes here

### viewWillAppear: and viewDidAppear: (both repeatedly)
- Repeatedly: view will only get “loaded”(viewDidLoad) once, but it might appear and disappear a lot.
- `- (void)viewWillAppear:(BOOL)animated;` a place to do something if things you display are changing while your MVC is off-screen.
- `- (void)viewWillDisappear:(BOOL)animated` a place to put “remember what’s going on” and cleanup code.

{% highlight objective-c %}
- (void)viewWillDisappear:(BOOL)animated
{
[super viewWillDisappear:animated]; // call super in all the viewWill/Did... methods
// let’s be nice to the user and remember the scroll position they were at ...
[self rememberScrollPosition]; // we’ll have to implement this
// do some other clean up now that we’ve been removed from the screen
[self saveDataToPermanentStore]; // maybe do in did instead?
// but be careful not to do anything time-consuming here, or app will be sluggish
// maybe even kick off a thread to do what needs doing here
}
{% endhighlight %}

### didReceiveMemoryWarning
- iOS gets its right to kill your app if you are a memory eater...
- put clean up  and dealloc codes here if necessary
- set `strong` pointer to `nil`

<br>
------------

### Picture of View Controller Lifecycle
![ UIViewController Lifecycle ](/images/ios/uiviewcontroller_lifecycle.png)

[image source](http://rdkw.wordpress.com/2013/02/24/ios-uiviewcontroller-lifecycle/)

<br>
-------------

### Reference
1. Stanford iOS 7 development
2. [Mac Developer Library](https://developer.apple.com/library/mac/navigation/)
3. [UIViewController_Class Reference](https://developer.apple.com/library/ios/documentation/uikit/reference/UIViewController_Class/Reference/Reference.html)
4. [ViewController Programming](https://developer.apple.com/library/ios/featuredarticles/viewcontrollerpgforiphoneos/ViewLoadingandUnloading/ViewLoadingandUnloading.html)
5. [Stackoverflow](http://stackoverflow.com/questions/5562938/looking-to-understand-the-ios-uiviewcontroller-lifecycle)
