---
layout: post
title: "Learning Foundation.framework"
date: 2012-12-24 19:20
comments: true
categories: [Cocoa, Foundation.framework, Objective-C]
---

*[Last update: 2012-12-24]*

`Foundation.framework` is the most basic framework we use in almost all the iOS/Mac projects. If you can understand this framework well, it can save you a lot of time. Starting from today, I will try to start a new serial of posts about learning `Foundation.framework`.

**Just bear in mind that as the learning journey goes on, this post will be updated from time to time.**

In [the bottom of this post](#post_organization), I organized my posts in different groups, just as you can see in [the figure](#class_figure)

## Basic Understanding of `Foundation.framework`

Before we start to study the whole framework, a basic understanding of `Foundation.framework` is necessary. `Foundation.framework` is usually referred as `Foundation`.

1. `Foundation` is part of Cocoa framework(in OSX), and Cocoa Touch Framework(in iOS). To see what Cocoa is, Apple has a good documentation on [What is Cocoa](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/CocoaFundamentals/WhatIsCocoa/WhatIsCocoa.html#//apple_ref/doc/uid/TP40002974-CH3-SW16). 
  * In OSX, Cocoa includes 2 major frameworks: `Foundation` and `AppKit`.
  * In iOS, Cocoa Touch includes 2 major frameworks: `Foundation` and `UIKit`.
2. There are little differences between the `Foundation` in OSX and iOS, but most of the classes are identical in both platform.
3. `Foundation` defines a base layer of classes that can be used for any kind of Cocoa program. Thus `Foundation` has no relationship to user interface(`AppKit` or `UIKit`)

### Foundation Design Goal

> * Define basic object behavior and introduce consistent conventions for such things as memory management, object mutability, and notifications.
> * Support internationalization and localization with (among other things) bundle technology and Unicode strings.
> * Support object persistence.
> * Support object distribution.
> * Provide some measure of operating-system independence to support portability.
> * Provide object wrappers or equivalents for programmatic primitives, such as numeric values, strings, and collections. It also provides utility classes for accessing underlying system entities and services, such as ports, threads, and file systems.
> 
> Cocoa applications, which by definition link either against the AppKit framework or UIKit framework, invariably must link against the Foundation framework as well. The class hierarchies share the same root class, NSObject, and many if not most of the AppKit and UIKit methods and functions have Foundation objects as parameters or return values. Some Foundation classes may seem designed for applications—NSUndoManager and NSUserDefaults, to name two—but they are included in Foundation because there can be uses for them that do not involve a user interface.

### Foundation Paradigms & Policies

> Foundation introduces several paradigms and policies to Cocoa programming to ensure consistent behavior and expectations among the objects of a program in certain situations.:
> 
> * **Object retention and object disposal**. The Objective-C runtime and Foundation give Cocoa programs two ways to ensure that objects persist when they’re needed and are freed when they are no longer needed.     
>    **NOTE**: Garbage collection was introduced in Objective-C 2.0 and deprecated since ARC is released. For newly written applications, ARC is recommended. Even the old applications with explicit retain/release paradigm can be upgraded to use ARC.
> * **Mutable class variants**. Many value and container classes in Foundation have a mutable variant of an immutable class, with the mutable class always being a subclass of the immutable one. If you need to dynamically change the encapsulated value or membership of such an object, you create an instance of the mutable class. Because it inherits from the immutable class, you can pass the mutable instance in methods that take the immutable type. For more on object mutability, see “[Object Mutability](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/CocoaFundamentals/CocoaObjects/CocoaObjects.html#//apple_ref/doc/uid/TP40002974-CH4-SW33).”
> * **Class clusters**. A class cluster is an abstract class and a set of private concrete subclasses for which the abstract class acts as an umbrella interface. Depending on the context (particularly the method you use to create an object), an instance of the appropriate optimized class is returned to you. `NSString` and `NSMutableString`, for example, act as brokers for instances of various private subclasses optimized for different kinds of storage needs. Over the years the set of concrete classes has changed several times without breaking applications. For more on class clusters, see “[Class Clusters](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/CocoaFundamentals/CocoaObjects/CocoaObjects.html#//apple_ref/doc/uid/TP40002974-CH4-SW34).”
> * **Notifications**. Notification is a major design pattern in Cocoa. It is based on a broadcast mechanism that allows objects (called observers) to be kept informed of what another object is doing or is encountering in the way of user or system events. The object originating the notification can be unaware of the existence or identity of the observers of the notification. There are several types of notifications: synchronous, asynchronous, and distributed. The Foundation notification mechanism is implemented by the `NSNotification`, `NSNotificationCenter`, `NSNotificationQueue`, and `NSDistributedNotificationCenter` classes. For more on notifications, see “[Notifications](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/CocoaFundamentals/CommunicatingWithObjects/CommunicateWithObjects.html#//apple_ref/doc/uid/TP40002974-CH7-SW7).”

## Know the `Foundation` Classes

There are many classes and functions defined in `Foundation`. Understanding those classes one by one is key to accelerate your development of OSX/iOS apps.

Apple's development document already have some very good diagrams, which divided those classes into different groups by functionality.

### <a id="class_figure"></a>Figure: The Foundation class hierarchy

![](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CocoaFundamentals/Art/objc_foundation_A.jpg)
![](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CocoaFundamentals/Art/objc_foundation2_A.jpg)
![](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CocoaFundamentals/Art/objc_foundation3_A.jpg)

### Read the Release Note of Each Version

Apple is keeping the document correct and up-to-date, but, to be frank, *most of the time*, the document can not reflect the most  recent changes of each version, so reading the release note for each version is very important.

Here are the related release notes:

* [Foundation Release Note](https://developer.apple.com/library/mac/#releasenotes/Cocoa/FoundationOlder.html#//apple_ref/doc/uid/TP30000742)
* [OSX 10.8 Release Note:Foundation](https://developer.apple.com/library/mac/#releasenotes/MacOSX/WhatsNewInOSX/Articles/MacOSX10_8.html%23//apple_ref/doc/uid/TP40011634-SW24)
* [OSX 10.7 Release Note:Foundation](https://developer.apple.com/library/mac/#releasenotes/MacOSX/WhatsNewInOSX/Articles/MacOSX10_7.html%23//apple_ref/doc/uid/TP40010355-SW31)
* [OSX 10.6 Release Note:Foundation](https://developer.apple.com/library/mac/#releasenotes/MacOSX/WhatsNewInOSX/Articles/MacOSX10_6.html%23//apple_ref/doc/uid/TP40008898-SW32)

## <a id="post_organization"></a>The Organization of the Posts

Since there are so many different classes in the `Foundation`, it's necessary to group my posts as [the figure you see above](#class_figure).

1. Value Objects
2. XML
3. Strings
4. Collections
5. Predicates
6. OS Services
  1. File System
  2. URL
  3. Interprocess Communication
  4. Locking/Threading
7. Notifications
8. Archiving & Serialization
9. ObjC Language Services
10. Scripting
11. Distributed Objects.

### Asking for help

Since such kind of study is really huge and will cost a long time to archive the goal. I'm here to ask you to join the journey with me.

#### How can you help

1. Review the post I write, give me feedback.
2. Write posts with me, allow me to post in this site.
3. Write sample code with me.

#### How to Join the Journey

1. Twitter: [@TonnyXu](http://twitter.com/TonnyXu)
2. Facebook: [Tonny Xu](http://facebook.com/Xu.Tonny)
3. Email: tonny.xu[AT]gmail.com

----
## License - Creative Common 3.0

<a rel="license" href="http://creativecommons.org/licenses/by-nc/3.0/deed.en_US"><img alt="Creative Commons License" style="border-width:0" src="http://i.creativecommons.org/l/by-nc/3.0/88x31.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" property="dct:title">Loving Cocoa</span> by <a xmlns:cc="http://creativecommons.org/ns#" href="http://tonnyxu.github.com" property="cc:attributionName" rel="cc:attributionURL">Tonny Xu</a> is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc/3.0/deed.en_US">Creative Commons Attribution-NonCommercial 3.0 Unported License</a>.<br />Based on a work at <a xmlns:dct="http://purl.org/dc/terms/" href="http://tonnyxu.github.com" rel="dct:source">http://tonnyxu.github.com</a>.