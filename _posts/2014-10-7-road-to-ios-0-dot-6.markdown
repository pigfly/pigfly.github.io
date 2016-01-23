---
layout: post
title: "Road To iOS Series 6"
date: 2014-10-7 4:14:19 +1000
comments: true
categories: [ios]
header-img: "img/post-bg-11.jpg"
---

The bits about view in iOS:

- View is an instance of UIView or one of its subclass (e.g. UIScrollView UILabel...)
- View knows how to draw itself (e.g. drawInRect)
- View handles events (e.g. touches, value changes)
- View exists within a hierarchy of views. (root view is app's window)

<!--more-->

Views have their own hierarchy:

- Each view in the hierarchy, renders itself to its *layer*, an instance of CALayer
- The layers of all the views are composited together to form a complete view
- When you add a view as subview of another view, the superview and subviews  properties are automatically established
- Classes like `UIButton`, `UILabel` already know how to draw themselves to their layers

![view hierarchy diagram ](/images/ios/view_hierarchy_2.png)

------------

So how to create view programmatically:

- You create your own view class inherent from UIView
- initWithFrame: is designated initializer
- drawRect: is for custom drawing

{% highlight objective-c %}
// designated initializer
- (id)initWithFrame:(CGRect)frame
{
    self = [super initWithFrame:frame];
    if (self) {
        // do init setup for your view

    }
    return self;
}

// Only override drawRect: if you perform custom drawing.
- (void)drawRect:(CGRect)rect
{
	...
}
{% endhighlight %}

- To create a view, you need to get its *frame*
- A *frame* specifies the view's size and its position relative to its superview, and it's always in a rectangle model.
- Steps for creating a view :
	- Specify the view's frame
	- Init an instance of view
	- configure the view instance
	- add view instance as subview

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

------------

How to do customized view:

- UIView subclasses override drawRect: to perform custom drawing. e.g. the drawRect: for UILabel draw text on screen
- To override drawRect, you need to get the *bounds*
	- bounds is a view's rectangle in its **own** coordinate system
	- frame is the same rectangle in its **superview's** coordinate system
- Steps for creating a custom view in drawRect:
	- Specify the bounds of view
	- Specify necessary geometry variables for building UIBezierPath
	- Init an instance of UIBezierPath
	- Call appropriate drawing API in UIBezierPath
	- Draw the line


{% highlight objective-c %}
- (void)loadView
{
	// 1. Specify the bounds of view
	CGRect bounds = self.bounds;

	// 2. Specify necessary geometry variables for building UIBezierPath
	CGPoint center;
	center.x = bounds.origin.x + bounds.size.width / 2.0;
	center.y = bounds.origin.y + bounds.size.height / 2.0;

	// The largest circle will circumstribe the view
	float maxRadius = hypot(bounds.size.width, bounds.size.height) / 2.0;

	// 3. Init an instace of UIBezierPath
	UIBezierPath *path = [[UIBezierPath alloc] init];

	// 4. Call appropriate drawing API in UIBezierPath
	[path addArcWithCenter:center
	                radius:maxRadius
	            startAngle:0.0
	              endAngle:M_PI * 2.0
	             clockwise:YES];


	// Configure the drawing color to light gray
	[[UIColor lightGrayColor] setStroke];

	// 5. Draw the line!
	[path stroke];
}
{% endhighlight %}

------------

Event Handler & Redrawing:

- When user touches a view, the view is sent the message touchesBegan:withEvent:
- It's a view's event handler in the run loop

> When an app is launched, it starts a run loop, or run lifecycle. Its job is to listen for events, such as
> touch. When an event occurs, it finds the appropriate handler methods for event. Once finished, control
> returns to the run loop.

- When run loop regains control, it checks a list of "dirty views" - views that need to be re-rendered
- Then run loop then sends the drawRect: message to those "dirty views"
- To get a view on the re-rendered lists, you **must** send view the message setNeedDisplay

{% highlight objective-c %}
// When a finger touches the screen
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
{
    // Get 3 random numbers between 0 and 1
    float red = (arc4random() % 100) / 100.0;
    float green = (arc4random() % 100) / 100.0;
    float blue = (arc4random() % 100) / 100.0;

    UIColor *randomColor = [UIColor colorWithRed:red
                                           green:green
                                            blue:blue
                                           alpha:1.0];

    self.circleColor = randomColor;
}
{% endhighlight %}

{% highlight objective-c %}
- (void)setCircleColor:(UIColor *)circleColor
{
	self.circleColor = circleColor;

	// send the view message to re-render
	[self setNeedsDisplay];
}
{% endhighlight %}

------------

Reference

1. Stanford iOS 7 development
2. [Mac Developer Library](https://developer.apple.com/library/mac/navigation/)
3. [Big Nerd Ranch](http://www.bignerdranch.com/)
