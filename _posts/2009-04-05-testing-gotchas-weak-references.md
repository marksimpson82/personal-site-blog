---
id: 129
title: 'Testing gotchas &#8211; c# Weak References'
date: 2009-04-05T02:55:49+00:00
author: Mark Simpson
layout: single
guid: https://defragdev.com/blog/?p=129
#permalink: /?p=129
tags:
  - 'c#'
  - gotchas
  - testing
---
If you ever have to test a class that uses a [WeakReference](http://msdn.microsoft.com/en-us/library/system.weakreference.aspx), or even just have to _use_ Weak References, be very careful.  Numerous strange-looking things can occur when [Weak References](http://msdn.microsoft.com/en-us/library/system.weakreference.aspx) are involved.

If you have even a cursory understanding of the [.NET Garbage Collector](http://msdn.microsoft.com/en-us/library/0xy59wtx.aspx) (GC), you will know that it keeps track of objects.  When an object is no longer strongly referenced, the GC will potentially collect it, freeing up its resources.  This causes the object to &#8216;disappear&#8217;. So, if you have a strong reference to an object in your program, you are generally safe in the assumption that the object will stick around.  The GC won&#8217;t pull the carpet from under you while you&#8217;re using that object.

Weak References, on the other hand, do **not** stop the GC from collecting the object they refer to.  In certain circumstances, it can be advantageous to use Weak References because you do want to use/observe/whatever an object, but you don&#8217;t want to stop it from being collected.  So far so good.  All obvious stuff.

### The GC moves in mysterious ways

OK you say, what&#8217;s the point in this article?  The point is that GC is extremely clever.  Almost a little too clever.  So clever it may aggressively collect objects to the point where it can mess with your head, and your tests.  This &#8216;problem&#8217; can manifest itself in subtle ways &#8212; certain types of tests involving Weak References will usually pass in debug mode, but may sporadically fail in release mode.  Heads will be scratched.  Bemused gurning will commence.

### Obligatory Contrived Example

<pre>public class GuyWhoUsesWeakRef
{
    private WeakReference m_weakBuilder;

    public GuyWhoUsesWeakRef(StringBuilder _builder)
    {
        m_weakBuilder = new WeakReference(_builder);
    }

    public bool IsWeakRefValid { get { return m_weakBuilder.Target != null; } }
}

[Test]
public void IsWeakRefValid_WhenValidRef_ReturnsTrue_Test()
{
    var builder     = new StringBuilder();
    var usesWeakRef = new GuyWhoUsesWeakRef(builder);

    Assert.That(usesWeakRef.IsWeakRefValid, Is.True,
        "Operation relying on the weak reference should've succeeded");
}</pre>

### Why does this occasionally fail?

If you look carefully, you may spot the problem.  Recall that I said that objects can be collected **while still in scope**.  After the builder reference has been passed to the GuyWhoUsesWeakRef constructor, it is no longer used anywhere.  The GuyWhoUsesWeakRef class doesn&#8217;t take a strong reference, so the moment the parameter is no longer used, that reference also gets discarded.

As a result, immediately after the new GuyWhoUsesWeakRef(builder) call, the GC figures out that the StringBuilder object we&#8217;ve created will never be used again.  After all, if the object is never used again, why not collect it as soon as possible?

In debug mode, this won&#8217;t throw a spanner in the works.  The test will pass because the GC is not aggressively collecting.  However, in release mode, the GC may well collect the StringBuilder when we fully expect it to still be alive for our Assert.That() call.

The main problem is that it won&#8217;t happen every time.  The GC is _non-deterministic_, so this test will pass and fail intermittently; it depends on the timing of the collection.  Coming from C++ where objects are destroyed as they exit scope, I found this somewhat bemusing.  In the context of the GC, it makes sense, though.  You just have to be careful.

### The solution

The good folks at Microsoft they provided a very simple static method call to solve this particular problem; enter [GC.KeepAlive](http://msdn.microsoft.com/en-us/library/system.gc.keepalive.aspx).  Placing a call to GC.KeepAlive(builder) at the end of this test method will ensure that the object we&#8217;re referring to will not be collected until after the GC.KeepAlive call has been made.  Problem solved.

<pre>[Test]
public void IsWeakRefValid_WhenValidRef_ReturnsTrue_Test()
{
    var builder     = new StringBuilder();
    var usesWeakRef = new GuyWhoUsesWeakRef(builder);

    Assert.That(usesWeakRef.IsWeakRefValid, Is.True,
        "Operation relying on the weak reference should've succeeded");

    GC.KeepAlive(builder); // weak ref is now valid up to this point
}</pre>