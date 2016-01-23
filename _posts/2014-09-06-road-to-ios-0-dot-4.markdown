---
layout: post
title: "Road To iOS Series 4"
date: 2014-09-06 12:14:19 +1000
comments: true
header-img: "img/post-bg-03.jpg"
categories: [ios]
---

## Multiple MVCs

### Two multiple MVC controllers
- **UINavigationController**
	- often used for a more detailed view (e.g. calendar in iOS)
- **UITabBarController**
	- often used for views have no logical connections (e.g. timer in iOS)

<!--more-->

### What is UINavigationController

> Whenever an iOS app displays a user interface, the displayed content is managed by a view controller or a
> group of view controllers coordinating with each other. Therefore, view controllers provide the skeletal
> framework on which you build your apps.


- It's a View Controller **manages stacks of other view controllers**
- An MVC's view is another MVC

![ 600 480 navigation interface overview ](/images/ios/navigation_interface.png)

[image source](https://developer.apple.com/library/ios/featuredarticles/ViewControllerPGforiPhoneOS/Art/navigation_interface_2x.png)

If user wants to see more detail information about current view, navigation controller will push the view from root view->list view->detail view.

<br>
------------

### How does UINavigationController work

#### Overview
<script type="text/javascript" src="/javascripts/libs/jssor.core.js"></script>
<script type="text/javascript" src="/javascripts/libs/jssor.utils.js"></script>
<script type="text/javascript" src="/javascripts/libs/jssor.slider.min.js"></script>
<script type="text/javascript" src="/javascripts/slides.js"></script>
<div id="slider1_container" style="position: relative; top: 0px; left: 0px; width: 600px;
        height: 300px;">

        <!-- Loading Screen -->
        <div u="loading" style="position: absolute; top: 0px; left: 0px;">
            <div style="filter: alpha(opacity=70); opacity:0.7; position: absolute; display: block;
                background-color: #000000; top: 0px; left: 0px;width: 100%;height:100%;">
            </div>
            <div style="position: absolute; display: block; background: url(/images/loading.gif) no-repeat center center;
                top: 0px; left: 0px;width: 100%;height:100%;">
            </div>
        </div>

        <!-- Slides Container -->
        <div u="slides" style="cursor: move; position: absolute; left: 0px; top: 0px; width: 600px; height: 300px;
            overflow: hidden;">
            <div><img u="image" src="/images/ios/mvc_working_together1.png" /></div>
            <div><img u="image" src="/images/ios/mvc_working_together2.png" /></div>
            <div><img u="image" src="/images/ios/mvc_working_together3.png" /></div>
            <div><img u="image" src="/images/ios/mvc_working_together4.png" /></div>
        </div>

        <style>
            .jssorb03 div, .jssorb03 div:hover, .jssorb03 .av
            {
                background: url(/images/b03.png) no-repeat;
                overflow:hidden;
                cursor: pointer;
            }
            .jssorb03 div { background-position: -5px -4px; }
            .jssorb03 div:hover, .jssorb03 .av:hover { background-position: -35px -4px; }
            .jssorb03 .av { background-position: -65px -4px; }
            .jssorb03 .dn, .jssorb03 .dn:hover { background-position: -95px -4px; }
        </style>
        <!-- bullet navigator container -->
        <div u="navigator" class="jssorb03" style="position: absolute; bottom: 4px; left: 6px;">
            <!-- bullet navigator item prototype -->
            <div u="prototype" style="position: absolute; width: 21px; height: 21px; text-align:center; line-height:21px; color:white; font-size:12px;"><NumberTemplate></NumberTemplate></div>
        </div>

</div>


- **Segue**: one MVC goes to the other MVCs, or a visual transition from one scene to another
- **Independent MVC**: each MVC is independent, it's encapsulated in its own view controller
- **No memory between MVCs**: everytime we push(segue) to another MVC, we create a new one, **old one gets dealloc from the heap**. You may need variables to store information if you want persistent data between segues
- When you create variables to store information, outlets in new view is not set yet.

#### Detail
A navigation controller is a container view controller—that is, **it embeds the content of other view controllers inside of itself**. You access a navigation controller’s view from its view property. This view incorporates:

- **navigation bar**
	- an NSArray of UIBarButtonItems
	- the back button will be generated automatically
- **optional toolbar**
	- an NSArray of UIBarButtonItems
	- optional, not necessary for every content view controller
- **content view**
	- create your own
	- by **subclassing either the UIViewController** class or the UITableViewController class

![UINavigationController view components ](/images/ios/UINavigationController_view_hierarchy.jpg)

<br>
------------

### How does UINavigationController work (Cont.)

#### Push the view to screen
- often, in Xcode, we use push segue and along with its identifier to connect different MVCs
- however, you can do this programmatically with `- (void)pushViewControllerAnimated:(BOOL)animated;`

![ 600 400 segue ](/images/ios/segue.png)

#### Take the view off screen
- often, user hits the back button on the top left of the screen to get back to the parent view
- however, you can do this programmatically with `- (void)popViewControllerAnimated:(BOOL)animated;`

#### Prepare for the segues
The segue offers the source VC the opportunity to “prepare” the new VC to come on screen. This method is sent to the VC that contains the button that initiated the segue:

{% highlight objective-c %}
- (void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender {
	// check the segue id we set in storyboard
    if ([segue.identifier isEqualToString:@“DoSomething”]) {
    	// check if the destination is correct by checking class reference
        if ([segue.destinationViewController isKindOfClass:[DoSomethingVC class]]) {
            DoSomethingVC *doVC = (DoSomethingVC *)segue.destinationViewController;
            	// e.g. doVC.date = getDateFromSomewhere(...);
				doVC.neededInfo = ...;
		}
	}
}
{% endhighlight %}

#### Push, Pop lifecycle

![push pop lifecycle](/images/ios/uinavigation_lifecycle.png)

<br>
------------

### Conclusion

- UINavigationController is container view controller to manage other view controllers(MVCs)
- Each MVC in UINavigationController is well encapsulated and independent
- UINavigationController mainly consists of three parts: navigation bar, navigation toolbar, content views(other MVCs)
- UINavigationController uses push segue and pop segue to make transition between content views(other MVCs)
- Segues use prepareForSegue to prepare data between different pushes

<br>
------------

### Reference
1. Stanford iOS 7 development
2. [Mac Developer Library](https://developer.apple.com/library/mac/navigation/)
3. [How a Navigation-based App Fits Together](http://adoptioncurve.net/archives/2013/04/how-a-navigation-based-app-fits-together)
4. [UINavigationController Class Reference](https://developer.apple.com/library/ios/documentation/uikit/reference/UINavigationController_Class/Reference/Reference.html#//apple_ref/occ/instp/UINavigationController/navigationBar)
5. [View Controller Programming Guide for iOS](https://developer.apple.com/library/ios/featuredarticles/ViewControllerPGforiPhoneOS/Introduction/Introduction.html#//apple_ref/doc/uid/TP40007457)
