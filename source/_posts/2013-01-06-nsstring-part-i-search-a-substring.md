---
layout: post
title: NSString - Part I: Search/Find a substring
date: 2013-01-06 18:18
comments: true
categories: [Objective-C, Cocoa, Foundation.framework, NSString, search, algorithm]
---

## The most common task for string object: find a substring

Finding a substring `T` whether in the source string `S` or not is definitely the most common task(You can argue that ;)) for a string object. NSString already provides a bunch of functions for that. Let's take a look at [NSString Class Reference, section "Finding Characters and Substrings"](https://developer.apple.com/library/mac/#documentation/Cocoa/Reference/Foundation/Classes/NSString_Class/Reference/NSString.html%23//apple_ref/doc/uid/20000154-SW23)

There are 9 methods listed there:

1. `– rangeOfCharacterFromSet:`
2. `– rangeOfCharacterFromSet:options:`
3. `– rangeOfCharacterFromSet:options:range:`
4. `– rangeOfString:`
5. `– rangeOfString:options:`
6. `– rangeOfString:options:range:`
7. `– rangeOfString:options:range:locale:`
8. `– enumerateLinesUsingBlock:`
9. `– enumerateSubstringsInRange:options:usingBlock:`

Basically, the first 3 methods are different variants for finding a character, and 4-7 is another set of variant methods for string. 8 and 9 are for paragraphs.

Since `NSString` is `toll-free bridged` with `CFStringRef`, and `CoreFoundation` is an open source framework that you can always check the source at [http://opensource.apple.com](http://opensource.apple.com/source/CF/CF-744.12/). Current available version is **OS X 10.8.2**, and the CF(=CoreFoundation) has a version number: **744.12**. This means we can find corresponding C methods in [CFString.h](http://opensource.apple.com/source/CF/CF-744.12/CFString.h):

```c
CF_EXPORT
CFComparisonResult CFStringCompareWithOptionsAndLocale(CFStringRef theString1, CFStringRef theString2, CFRange rangeToCompare, CFStringCompareFlags compareOptions, CFLocaleRef locale) CF_AVAILABLE(10_5, 2_0);

CF_EXPORT
CFComparisonResult CFStringCompareWithOptions(CFStringRef theString1, CFStringRef theString2, CFRange rangeToCompare, CFStringCompareFlags compareOptions);

CF_EXPORT
CFComparisonResult CFStringCompare(CFStringRef theString1, CFStringRef theString2, CFStringCompareFlags compareOptions);

CF_EXPORT
Boolean CFStringFindWithOptionsAndLocale(CFStringRef theString, CFStringRef stringToFind, CFRange rangeToSearch, CFStringCompareFlags searchOptions, CFLocaleRef locale, CFRange *result) CF_AVAILABLE(10_5, 2_0);

CF_EXPORT
Boolean CFStringFindWithOptions(CFStringRef theString, CFStringRef stringToFind, CFRange rangeToSearch, CFStringCompareFlags searchOptions, CFRange *result);

CF_EXPORT
CFArrayRef CFStringCreateArrayWithFindResults(CFAllocatorRef alloc, CFStringRef theString, CFStringRef stringToFind, CFRange rangeToSearch, CFStringCompareFlags compareOptions);

CF_EXPORT
CFRange CFStringFind(CFStringRef theString, CFStringRef stringToFind, CFStringCompareFlags compareOptions);

CF_EXPORT
Boolean CFStringHasPrefix(CFStringRef theString, CFStringRef prefix);

CF_EXPORT
Boolean CFStringHasSuffix(CFStringRef theString, CFStringRef suffix);

CF_EXPORT CFRange CFStringGetRangeOfComposedCharactersAtIndex(CFStringRef theString, CFIndex theIndex);
```

To make it easy to read, all the comments are deleted, you can read them from the web site.

Basically, for string operations, the 

----
### Reference
1. [Searching cocoa string](http://hecticant.wordpress.com/2010/01/25/searching-cocoa-strings/), shown that finding a substring in cocoa is not so efficient.
2. [Cocoa Performance Guidelines: String Management](http://developer.apple.com/library/Mac/#documentation/Cocoa/Conceptual/CocoaPerformance/Articles/StringDrawing.html#//apple_ref/doc/uid/TP40001445-BCICGHBC), some basic rules for using string in Cocoa.
3. [String Programming Guide](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/Strings/introStrings.html#//apple_ref/doc/uid/10000035i), **very important** guide line that any Cocoa developer definitely need to read throughout.
4. [String Programming Guide for Core Foundation](https://developer.apple.com/library/mac/#documentation/CoreFoundation/Conceptual/CFStrings/introCFStrings.html#//apple_ref/doc/uid/10000131i), from the Core Foundation's perspective. Much more from a low level perspective.
5. [CFString.h](http://opensource.apple.com/source/CF/CF-744.12/CFString.h)
6. [CFString.c](http://opensource.apple.com/source/CF/CF-744.12/CFString.c)
7. [CFData.h](http://opensource.apple.com/source/CF/CF-744.12/CFData.h)
8. [CFData.c](http://opensource.apple.com/source/CF/CF-744.12/CFData.c)
10. [KMP algorithm from Matrix67.com](http://www.matrix67.com/blog/archives/115)
11. [KMP algorithm from en.wiki](http://en.wikipedia.org/wiki/Knuth–Morris–Pratt_algorithm)
12. [Extra String matching algorithms](http://www-igm.univ-mlv.fr/%7Elecroq/string/node1.html)
13. [Boyer-Moore Algorithm from en.wiki](http://en.wikipedia.org/wiki/Boyer–Moore_string_search_algorithm)
14. [From KMP to BM algorithm](http://blog.csdn.net/v_JULY_v/article/details/6545192)
15. [Design and Analysis of Algorithms: KMP](http://www.ics.uci.edu/~eppstein/161/960227.html)

What?