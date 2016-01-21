---
layout: post
title: "road to iOS 0.1"
date: 2014-08-31 09:16:07 +1000
comments: true
categories: [ios]
---

## Attributed String & UITextView in Objective-C

### What is Attributed String
Determine how text will render on the screen. You can think of `NSAttributedString` as an NSString where each character has an NSDictionary of "attributes".

``` objective-c example of creating attributed strings
// this will create a dictionary for holding diifferent standard attributes
// the name of attributes are global NSString constants
NSDictionary *attrsDictionary =
	// specify font attributes
	@{ NSFontAttributeName :
	     [UIFont preferredFontWithTextStyle:UIFontTextStyleHeadline],
	// specify foreground attribute
	  NSForegroundColorAttributeName : [UIColor greenColor],
	  NSStrokeWidthAttributeName : @-5,
	  NSStrokeColorAttributeName : [UIColor redColor] }
// creating attributed string using attribute dictionary
NSAttributedString *attrString =
	[[NSAttributedString alloc] initWithString:@"string" attributes:attrsDictionary];
```

<!--more-->

### What is NSRange
To specify subranges inside strings and arrays. It's a **c structure** defined as below:

``` objective-c definition for NSRange
typedef struct _NSRange {
	  // The start index (0 is the first, as in C arrays).
      NSUInteger location;
      // The number of items in the range (can be 0).
      NSUInteger length;
} NSRange;
```

### NSNotFound
A value, or a constant that indicates that an item requested couldn’t be found or doesn’t exist.

``` objective-c illustrate NSRange + NSNotFound
NSString *heystack = @"heystack";
NSString *needle = @"hi";
// find the range of needle inside heystack
NSRange r = [heystack rangeOfString:needle];
// if fail to find, do something
if (r.location == NSNotFound) {...}
```

### NSRangePointer
Just a `NSRange *` used as an method parameter


<br>
-----------------

### Access attributes of NSAttributedString
- get the value for an attribute

``` objective-c get the value for an attribute from NSAttributedString
- (id)attribute:(NSString *)attributeName atIndex:(NSUInteger)index
								   effectiveRange:(NSRangePointer)aRange
// attributeName: the name of standard attribute
// aRange: If you don't need this value, pass NULL
// return value: The value for the attribute named attributeName of the character at index index, or nil if there is no such attribute.
```

- get all attributes as NSDictionary

``` objective-c return all attributes as NSDictionary
- (NSDictionary *)attributesAtIndex:(NSUInteger)index effectiveRange:(NSRangePointer)aRange
// return value: The attributes for the character at index.
// aRange: If you don't need this value, pass NULL
```

### Set attributes of NSAttributedString
Since NSAttributedString is **immutable**, we need `NSMutableAttributedString` to do all the settings...

``` objective-c setting attributes on characters
// add an attribute to a range of characters
- (void)addAttributes:(NSDictionary *)attributes range:(NSRange)range;
// set attributes in a range
- (void)setAttributes:(NSDictionary *)attributes range:(NSRange)range;
// remove attribute in a range
- (void)removeAttribute:(NSString *)attributeName range:(NSRange)range;
```

<br>
------------

### NSAttributedString is not an NSString
It's not a subclass of `NSString`, methods of `NSString` don't apply for `NSAttributedString`. However, you can convert it to `NSString` using **- (NSString *)string**.

``` objective-c NSAttributedString to NSString
NSAttributedString *attributedString = ...;
NSString *substring = ...;
NSRange r = [[attributedString string] rangeOfString:substring];
```

<br>
-------------

### UITextView
- Display multiline text using custom style
- selectable, editable, scrollable

#### Set text and attributes in UITextView using textStorage
- NSTextStorage is a subclass of NSMutableAttributedString.
- simply modify it and the UITextView will **automatically update**

``` objective-c use textStorage to modify content in UITextView
@interface ViewController ()
@property (weak, nonatomic) IBOutlet UITextView *body;
@end
// Prototype: @property (nonatomic, readonly) NSTextStorage *textStorage
[self.body.textStorage addAttributes:@{ NSStrokeWidthAttributeName : @3,
                                        NSStrokeColorAttributeName :[UIColor purpleColor]}
                               range:self.body.selectedRange];
```

#### Set font in UITextView
- one thing to keep in mind, the `@property (nonatomic, strong) UIFont *font;` will apply a font to the entire UITextView

<br>
-------------

### Mindmap for NSAttributedString
![ NSAttributedString Mindmap ](/images/ios/NSAttributedString.png)

<br>
-------------

### Reference
1. Stanford iOS 7 development
2. [Mac Developer Library](https://developer.apple.com/library/mac/navigation/)
3. [Beginning iOS 7 Development](http://www.apress.com/9781430260226)
