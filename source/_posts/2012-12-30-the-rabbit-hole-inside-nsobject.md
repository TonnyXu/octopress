---
layout: post
title: The rabbit hole inside NSObject
date: 2012-12-30 14:45
comments: true
categories: [Objective-C, Cocoa, Foundation.framework, NSObject]
---

`NSObject` and `NSProxy` is the only two root classes in Cocoa. You can create your own root class, but that is a tough task and it's not a kind of work that most of us can do. Although `NSObject` is used everywhere, but it's not understood by most of us. Understanding `NSObject` and Objective-C Runtime system could help us a lot.

Meanwhile, there are 2 `NSObject`s, one stands for root class, another stands for protocol. 

----
# NSObject as a root class

## <a id=""></a>A peak of the `NSObject` header file

### `isa` pointer

#### the meta-class

### Opaque type `Class` since Objective-C 2.0

### Test it with LLDB

## The implementation of `NSObject`

See ref[1] Object-C Runtime

### Memory layout

#### View Memory in Xcode

----
# NSObject as a protocol

## The definition of `NSObject` protocol

## The relationship between `NSObject` class and `NSObject` protocol




----
Nice source: 

1. [Runtime](http://opensource.apple.com/source/objc4/)
1. [CoreFoundation](http://opensource.apple.com/source/CF/)
2. http://cocoadev.com
3. [Objective-C Internals by Andre Pang](http://algorithm.com.au/downloads/talks/objective-c-internals/objective-c-internals.pdf)
4. [Objective-C Runtime Programming Guide](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtVersionsPlatforms.html#//apple_ref/doc/uid/TP40008048-CH106-SW1), old, not updated since 2009
5. [Clang Documentation](http://clang.llvm.org/docs/index.html), recently updated, will move to reStructuredText soon.(Dec 30, 2012)
6. [Cocoa Fundamentals Guide](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/CocoaFundamentals/Introduction/Introduction.html#//apple_ref/doc/uid/TP40002974-CH1-SW1)

Source: // ~/Download/objc4-532.2