---
id: 767
title: 'Integer to binary string in c#'
date: 2014-04-17T23:43:22+00:00
author: Mark Simpson
layout: single
guid: http://defragdev.com/blog/?p=767
permalink: /?p=767
categories:
  - 'c#'
  - python
  - tips
tags:
  - binary
  - bitwise
  - 'c#'
  - decimal
---
Python has the terribly useful functionality of [bin()](https://docs.python.org/2/library/functions.html#bin) to do this, but what about C#?

On the rare occasion I have to monkey around with bitwise operators, I always keep this handy. It has the added bonus of printing the leading zeros – I find this very useful when writing/debugging bitwise code, as it makes it easy to line up bitmasks etc. in the immediate window / console.

> public static class BinaryPrinter  
> {  
> &#160;&#160;&#160; public static string ToBinaryString(long _long)  
> &#160;&#160;&#160; {  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; // have to do this or it won&#8217;t print the leading 0 values&#160;&#160;&#160;&#160;&#160;&#160;&#160;  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; return Convert.ToString(_long, 2).PadLeft(sizeof(long) * 8, &#8216;0&#8217;);&#160;  
> &#160;&#160;&#160; }  
> }

Similarly, you could write a few of these (for all of the various inbuilt types) and chuck them in extension methods.