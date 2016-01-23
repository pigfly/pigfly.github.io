---
layout: post
title: "Road to iOS Series 9"
date: 2014-11-11 02:14:19 +1000
comments: true
header-img: "img/post-bg-09.jpg"
categories: [ios]
---

ImageIO - could not find ColorSync function [Solved]

Recently I found an undocumented bug in iOS 8 when I tried to generate pdf under UIkit framework. The bug blocks me from generating pdf in a preferred way using UIPrint, so I had to use a more primitve method to generate the pdf in iOS8.

For better understanding this issue, first I will introduce ways of generating PDF in iOS, and why it may become a problem in iOS8 or above.

<!--more-->

- The first way of generating PDF is using UiKit and core graphics. It provides a set of functions for generating PDF content using native drawing code. However, there are some classes like *UIBezierPath* provide more high level API for drawing. Truth is, those drawing classes wrap Core Graphics code into their methods to ease drawing for the programmer.
- Notice, the core graphics is a 2D drawing C API, it deals with lower level of drawing compared to the latter one. And you need to provide frame,  margins, translate matrix parameters,  number of pages etc in order to generate a user-friendly pdf.
- For a more detailed examples of generating PDF, visit apple doc at [generating PDF content](https://developer.apple.com/library/ios/documentation/2DDrawing/Conceptual/DrawingPrintingiOS/GeneratingPDF/GeneratingPDF.html)

- The task I involved is to generate PDF from html string. So I decide to use *UIWebView*'s viewPrintFormatter and other UIPrint* classes to achieve the goal.
- However, there is a key method in the UIPrintPageRenderer class causes drawing thread hang up, and never gets returned, here is the code and error message:

{% highlight objective-c %}
// the sixth line causes the problem
for ( int i = 0 ; i < self.numberOfPages ; i++ )
    {
        UIGraphicsBeginPDFPage();

        [self drawPageAtIndex: i inRect: bounds];
    }

// the console output
// *** ImageIO - could not find ColorSync function 'ColorSyncProfileCreateSanitizedCopy'
{% endhighlight %}

- When you start to trace the source of the method call, you will find it's something worked behind the scence, you can't even see the source code. So what I end up with doing is using a core graphics approach, avoiding using *drawPageAtIndex:* call, and figure out a way how to draw multiple pdf pages in pdf.


{% highlight objective-c %}
for (int i = 0 ;i < pages ;i++)
    {
        if (maxHeight * (i + 1) > height)
        {
            // Check to see if page draws more than the height of the UIWebView
            CGRect scrollViewFrame = [scrollView frame];
            scrollViewFrame.size.height -= (((i + 1) * maxHeight) - height);
            [scrollView setFrame:scrollViewFrame];
        }
        // Specify the size of the pdf page
        UIGraphicsBeginPDFPageWithInfo(CGRectMake(0, 0, kPaperWidthA4, kPaperHeightA4), nil);
        CGContextRef currentContext = UIGraphicsGetCurrentContext();

        CGContextTranslateCTM(currentContext, kMargin, -(maxHeight * i) + kMargin);

        [scrollView setContentOffset:CGPointMake(0, maxHeight * i) animated:NO];

        [scrollView.layer renderInContext:currentContext];
    }
{% endhighlight %}


------------

### Reference
1. Stanford iOS 7 development
2. [Mac Developer Library](https://developer.apple.com/library/mac/navigation/)
