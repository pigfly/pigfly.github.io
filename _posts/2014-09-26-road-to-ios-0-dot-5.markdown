---
layout: post
title: "road to iOS 0.5"
date: 2014-09-26 1:14:19 +1000
comments: true
categories: [ios]
---

##  Interface builder, Connections, Initializers

### Interface builder
- **StoryBoard**
	- easy way, less flexible, hard to do programmatically
- **XIB**
	- create a view only, easy to do programmatically

<!--more-->

#### What is XIB
- XIB stands for XML Interface Builder
- **Object editor**, you create and configure objects like buttons and controllers, save them into archive called XIB

#### What is NIB
- When you build app, the XIB file is compiled into NIB file for efficiency
- Then NIB file is copied into app's *bundle*
- **Bundle**, a directory contains app's executable and any resources

<br>
------------

### Connections
- Let one object know where another object is in memory, so that two objects can **communicate**
- More precisely,  often between controller and view communication
	- **outlets**, as property, no need to know who is sender
	- **actions**, as implementation, sometimes needed to know who is sender, so that you can perform desired action

#### Connect your controller to appDelegate
- To get your interface on screen, you have to connect your view controller to app's controller: **appDelegate**
- That means, you need to make your controller as *root controller* of this window.
- **appDelegate**, manages a single top-level **UIWindow** for the application.

``` objective-c make your controller as root controller 
- (BOOL)application:(UIApplication *)application 
didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
        // init window
        ....

        // init your controller, set as root
        YourViewController *vc = [[YourViewController alloc] init];
        self.window.rootViewController = vc;

        return YES;
}
```

<br>
------------

### Initializers
- begins with word **init**, name convention only
- **designated initializer**,  no matter how many init methods are,  one method is chosen as 
designated, it makes sure that every instance variable of an object is valid.

``` objective-c pattern for designated init
- (instancetype)initWithxxx:(yyy)zzz
		       	 xxx:(yyy)zzz...
{
	self = [super init];

	if (self)
	{
		...
	}

	return self;
}
```

#### Controller Initializer
When an instance of controller is created, it is sent the message **initWithNibName:bundle**

``` objective-c controller init
- (instancetype)initWithNibName:(NSString *)nibNameOrNil
			    bundle:(NSBundle *)nibBundleOrNil
{
	self = [super initWithNibName:nibNameOrNil bundle:nibBundleOrNil];

	if (self) 
	{
		....
	}

	return self;
}
```

#### Instancetype
- Why not return specific type ?  all about polymorphism
- Deal with the problem if the class was subclassed but sending this init message to subclass :(
- Return the type of the caller

<br>
------------

### Reference
1. Stanford iOS 7 development
2. [Mac Developer Library](https://developer.apple.com/library/mac/navigation/)
3. [Big Nerd Ranch](http://www.bignerdranch.com/)