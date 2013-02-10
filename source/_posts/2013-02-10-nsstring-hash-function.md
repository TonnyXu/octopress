---
layout: post
title: "NSString - Hash function"
date: 2013-02-10 18:40
comments: true
categories: [NSString, algorithm, hash]
---

## The origin of `-[NSString hash]` function

The `-[NSString hash]` function is actually defined by NSObject protocol(not the class), it defines `-[NSObject hash]` as required. You can see it from `NSObject.h`

```objective-c the define of `-hash` function
@protocol NSObject

- (BOOL)isEqual:(id)object;
- (NSUInteger)hash;

// others ignored.
@end
```

And since NSObject class adopts `NSObject` protocol, hence every object definitely has a function called `-hash`. While different classes have different types of data, writing a universal hash function for all the classes is not possible, so the classes in `Foundation.framework` implemented different hash function by override NSObject class's implementation.

<a id="note64"></a>
One thing to be **NOTED**: the return type of `-[NSObject hash]` function is `NSUInteger`. Because `NSUInteger` has different sizes on x86 and x86_64 and ARM systems, when implement your own class's hash function, don't forget that.

## When this function will be used?

When we write our code, we rarely use hash function directly. Why? Because it is usually used by container class `NSDictionary` which is wildly called **hash table**. 

> If you want to have a deep understanding of **hash table**, please see the [wikipedia page](http://en.wikipedia.org/wiki/Hash_table).

In short, hash table is a data structure that support O(1) **data access** by searching the **key**. Do reach that goal:

1. The keys a hash table contains must be **unique**.
2. A particular key can always be **hashed** to the same value.

One basic idea is that we can generate a integer hash value for each key so that each value is **unique**, and use this unique key to insert/delete/search the value. If the key is a decimal, that it would be easier, but numbers don't have meanings. Using a string to represent a key will make our code(and our life) much more easier. 

As a result, modern programing languages implemented hash table all support using a string as a key. That's when the `NSString`'s hash function is used.

## How `NSString` implements the hash function?

We talked about the key must be an **unique decimal number**. So the problem will be **how to turn a string into a unique number without collisions**. Actually this doesn't turn out to be an easy task. Many computer scientists had researched this topic and see [Introduction to algorithms, chapter 11](http://www.amazon.com/gp/product/0262533057/ref=as_li_ss_tl?ie=UTF8&camp=1789&creative=390957&creativeASIN=0262533057&linkCode=as2&tag=totonet-20)(a.k.a. CLRS) for details.

To understand how `NSString` implements the hash function, the best way is reading the source code, and Apple released the source code of `CoreFunction` which includes `CFString`, a toll-free bridge object to `NSString`, we can see the [source code of CFString](http://opensource.apple.com/source/CF/CF-744.12/CFString.c) now.

### Step 1: Grab the source code

To grab the source code, just access [http://opensource.apple.com/](http://opensource.apple.com/), navigate to a particular Mac OS X release, search for **CF-**, then you will find a link on the right to download the source code.

### Step 2: Open the source tree and navigate to `CFString.c`

Using vim/emacs or any text editor you like. There are 4 _public_ functions defined:

```c 4 hash functions defined in CFString
CFStringHashCString(const uint8_t *bytes, CFIndex len)
CFStringHashCharacters(const UniChar *characters, CFIndex len)
CFStringHashISOLatin1CString(const uint8_t *bytes, CFIndex len)
CFStringHashNSString(CFStringRef str)
```

Here is the start point of our journey.

### Step 3: Let the journey begin

Let's take a deep look into the `CFStringHashNSString(CFStringRef str)` because from what the function name tells, it's the function we are looking for.

<a id="list1"></a>
```c List 1
/* This is meant to be called from NSString or subclassers only. It is an error for this to be called without the ObjC runtime or an argument which is not an NSString or subclass. It can be called with NSCFString, although that would be inefficient (causing indirection) and won't normally happen anyway, as NSCFString overrides hash.
*/
CFHashCode CFStringHashNSString(CFStringRef str) {
    UniChar buffer[HashEverythingLimit];
    CFIndex bufLen;		// Number of characters in the buffer for hashing
    CFIndex len = 0;	// Actual length of the string
    
    len = CF_OBJC_CALLV((NSString *)str, length);
    if (len <= HashEverythingLimit) {
        (void)CF_OBJC_CALLV((NSString *)str, getCharacters:buffer range:NSMakeRange(0, len));
        bufLen = len;
    } else {
        (void)CF_OBJC_CALLV((NSString *)str, getCharacters:buffer range:NSMakeRange(0, 32));
        (void)CF_OBJC_CALLV((NSString *)str, getCharacters:buffer+32 range:NSMakeRange((len >> 1) - 16, 32));
        (void)CF_OBJC_CALLV((NSString *)str, getCharacters:buffer+64 range:NSMakeRange(len - 32, 32));
        bufLen = HashEverythingLimit;
    }
    return __CFStrHashCharacters(buffer, bufLen, len);
}
```

Note the comment at the beginning of this function. Because `NSCFString` is completely private, we don't have a chance to see how `NSCFString` to implement this. But, this function also clearly demonstrates how to implement a hash function, so let's continue our journey with this function.

#### The define of `HashEverythingLimit`

Basically, the `if` clause divides this function into 2 parts. The length of the string is more than `HashEverythingLimit` or not. 
`HashEverythingLimit` is defined as:

<a id="list2"></a>
```c List 2
/* String hashing: Should give the same results whatever the encoding; so we hash UniChars.
If the length is less than or equal to 96, then the hash function is simply the 
following (n is the nth UniChar character, starting from 0):
   
  hash(-1) = length
  hash(n) = hash(n-1) * 257 + unichar(n);
  Hash = hash(length-1) * ((length & 31) + 1)

If the length is greater than 96, then the above algorithm applies to 
characters 0..31, (length/2)-16..(length/2)+15, and length-32..length-1, inclusive;
thus the first, middle, and last 32 characters.

Note that the loops below are unrolled; and: 257^2 = 66049; 257^3 = 16974593; 257^4 = 4362470401;  67503105 is 257^4 - 256^4
If hashcode is changed from UInt32 to something else, this last piece needs to be readjusted.  
!!! We haven't updated for LP64 yet

NOTE: The hash algorithm used to be duplicated in CF and Foundation; but now it should only be in the four functions below.

Hash function was changed between Panther and Tiger, and Tiger and Leopard.
*/
#define HashEverythingLimit 96

#define HashNextFourUniChars(accessStart, accessEnd, pointer) \
    {result = result * 67503105 + (accessStart 0 accessEnd) * 16974593  + (accessStart 1 accessEnd) * 66049  + (accessStart 2 accessEnd) * 257 + (accessStart 3 accessEnd); pointer += 4;}

#define HashNextUniChar(accessStart, accessEnd, pointer) \
    {result = result * 257 + (accessStart 0 accessEnd); pointer++;}
```

The reason I post other context is because the comment before `HashEverythingLimit` well explained how apple's guys were thinking.

1. If the string is less than 96 characters, hash it directly.
2. If the string has more than 96 characters, get the first, middle, last 32 characters to form a 96 characters string, then hash the 96 characters string.

You can see the code in the [gist](#gist) below.

This explains what the `if` clause is doing in [List 1](#list1). The function described in [List 2](#list2)'s comment is called [TBD](). For details please see [CLRS, page xxx].

#### This inline function does the real work

Back to [List 1](#list1), we can see the `if` clause is actually forming a temp string buffer to be used to calculate the hash value. The real work is done in `return` line.

```c 
    return __CFStrHashCharacters(buffer, bufLen, len);
```

OK, let's see what this line function `__CFStrHashCharacters` really does.

<a id="list3"></a>
```c List 3: __CFStrHashCharacters
/* In this function, actualLen is the length of the original string; but len is the number of characters in buffer. The buffer is expected to contain the parts of the string relevant to hashing.
*/
CF_INLINE CFHashCode __CFStrHashCharacters(const UniChar *uContents, CFIndex len, CFIndex actualLen) {
    CFHashCode result = actualLen;
    if (len <= HashEverythingLimit) {
        const UniChar *end4 = uContents + (len & ~3);
        const UniChar *end = uContents + len;
        while (uContents < end4) HashNextFourUniChars(uContents[, ], uContents); 	// First count in fours
        while (uContents < end) HashNextUniChar(uContents[, ], uContents);		// Then for the last <4 chars, count in ones...
    } else {
        const UniChar *contents, *end;
	contents = uContents;
        end = contents + 32;
        while (contents < end) HashNextFourUniChars(contents[, ], contents);
	contents = uContents + (len >> 1) - 16;
        end = contents + 32;
        while (contents < end) HashNextFourUniChars(contents[, ], contents);
	end = uContents + len;
        contents = end - 32;
        while (contents < end) HashNextFourUniChars(contents[, ], contents);
    }
    return result + (result << (actualLen & 31));
}
```

Actually, this function is implemented right below the [List 2](#list2). 

Based on the length of the string buff to be hashed, it calculate the hash value every 4 characters, then add the modulo value and return. 

**That's it**.

## Verify the hash function

So far, we understand how the hash value of a NSString is generated. Since the root function is `CFStringHashNSString` we can place a symbolic breakpoint on this function and `__CFStringHash` in Xcode and see what will happen. Note that the core function to calculate the hash value `__CFStrHashCharacters` in [List 3](#list3) is marked `CF_INLINE` we could not put a breakpoint on this function.

![image of putting a symbolic breakpoint](/images/Screen Shot 2013-02-10 at 8.28.54 PM.png)

I wrote a simple function to see the differences of 2 functions: `-[NSString hash]` and `CFStringHashNSString`.

<a id="gist"></a>
{% gist 4749305 %}

{% gist 4749305 [output.txt] %}
If see the output, did you remember I mentioned to differences between 32bit system and 64bit system [before](#note64)?

The hash value returned by `CFStringHashNSString` is actually a 32bit unsigned integer, while `-[NSString hash]` does return a 64bit unsigned integer in 64bit system. If we look at [List 2](#list2) again, we will see that Apple's engineer noted that they haven't implemented `LP64` yet.

And the last demo code shows if you calculate the hash value for 2 strings longer than 96 characters but have the same characters in first, middle, last 32 characters, we did get 2 hash codes **exactly the same**.

## Problems left

But if we put a breakpoint on `__CFStringHash` function, we will see it does get called when `-[NSString hash]` is called. And if we call `__CFStringHash` directly we got the same result as `CFStringHashNSString`. Looks like the way we pass in the parameter to `__CFStringHash` is wrong. 

Now I'm not so clear about how get a 64bit long hash value when we call `__CFStringHash` or `CFStringHashNSString` directly. If anyone knows, please leave a message.