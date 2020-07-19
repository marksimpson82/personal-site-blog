---
id: 723
title: 'The fundamentals of unit testing : KISS'
date: 2012-11-04T18:32:49+00:00
author: Mark Simpson
layout: single
guid: http://defragdev.com/blog/?p=723
permalink: /?p=723
categories:
  - fundamentals of unit testing
  - testing
  - tips
---
KISS, ([Keep it Simple, Stupid](http://en.wikipedia.org/wiki/KISS_principle)) is a guiding principle in software engineering and it&#8217;s of vital importance when writing tests.&#160; Tests should be straightforward and contain as little logic as possible.&#160; 

I know from painful experience that trying to do something overly &#8216;clever&#8217; in a test is a recipe for disaster.

## Control Flow / Extra Logic in Tests

The moment you start changing the flow of a test based on some condition, you are setting yourself up for a fall.&#160; Think about it:&#160; we test a particular unit of production code because its logic may be flawed.&#160; If the test that validates the production code tends towards the same level of complexity, there is a distinct and real risk that the test code will also contain errors.

Here&#8217;s a real example of a bad test that contains error-prone logic (please ignore the potential floating point comparison issues… I’ll touch on that later):

> [Test]  
> public void IdentityTest()  
> {  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; var m = Mtx4f.Identity;
> 
> **&#160;&#160;&#160;&#160;&#160;&#160;&#160; for(int r = 0; r < 4; ++r) // potential error; looping**  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; {  
> **&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; for(int c = 0; c < 4; ++c) // potential error; looping**  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; {  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; **&#160;&#160;&#160;&#160;&#160;&#160;&#160; if(r != c) // potential error; logical test / branching**  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; Assert.AreEqual(m[r, c], 0.0f);  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; **else&#160; // potential error; logical test / branching**  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; {  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; Assert.AreEqual(m[r, c], 1.0f);  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; }  
> }

<!--more-->

Off the top of&#160; my head, here’s a list of gripes I have: The test loops _and_ branches in a totally unnecessary fashion.&#160; It’s easy to get an r or a c mixed up, just as it’s simple to make an if/else mistake.&#160; The assert branching is particularly concerning.&#160; There’s a few extra variables in the mix too, so there’s more balls in the air, so to speak.&#160; 

In addition to the potential for logical errors, I can’t just look at the code and say, “row 3, column 2 of the matrix should have <x> value”.&#160; As a result, the test has been obfuscated, because I have to understand & walk through the execution of an algorithm to determine what the result should be. In this case, the test code is actually _much more_ complicated than the production code it is testing – a red flag if I’ve ever seen one.

Would it really have hurt so much to validate the 16 elements manually? We&#8217;re talking about 16 lines of code versus 12 or 13 lines of _complicated_ code&#8211; it&#8217;s not the end of the world, and it&#8217;s not often you have to write such verbose code.

Here&#8217;s two alternative tests that do the same thing in a simpler way:

> [Test]  
> public void IdentityTest()  
> {  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; var i = Mtx4f.Identity;
> 
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; // Row 0  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; Assert.That(i[0,0], Is.EqualTo(1f));  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; Assert.That(i[0,1], Is.EqualTo(0f));  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; Assert.That(i[0,2], Is.EqualTo(0f));  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; Assert.That(i[0,3], Is.EqualTo(0f));
> 
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; // Row 1  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; Assert.That(i[1,0], Is.EqualTo(0f));  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; Assert.That(i[1,1], Is.EqualTo(1f));  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; Assert.That(i[1,2], Is.EqualTo(0f));  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; Assert.That(i[1,3], Is.EqualTo(0f));
> 
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; // Row 2  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; Assert.That(i[2,0], Is.EqualTo(0f));  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; Assert.That(i[2,1], Is.EqualTo(0f));  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; Assert.That(i[2,2], Is.EqualTo(1f));  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; Assert.That(i[2,3], Is.EqualTo(0f));
> 
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; // Row 3  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; Assert.That(i[3,0], Is.EqualTo(0f));  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; Assert.That(i[3,1], Is.EqualTo(0f));  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; Assert.That(i[3,2], Is.EqualTo(0f));  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; Assert.That(i[3,3], Is.EqualTo(1f));  
> }

This one is fairly verbose, has some scope for errors (row & column accessors are very easy to typo and/or transpose) and will soon get cumbersome if you have to write multiple tests.&#160; To work around the verbosity, I would probably go with something more like this:

> [Test]  
> public void IdentityTest()  
> {  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; // Arrange  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; var identity = Mtx4f.Identity;&#160;  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; var expected = Matrix(1, 0, 0, 0,  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; 0, 1, 0, 0,  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; 0, 0, 1, 0,  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; 0, 0, 0, 1);
> 
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; // Act & Assert  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; Assert. That(identity, Is.EqualTo(expected));  
> }

**Note**: the second test assumes that there is an equals implementation for Mtx4f and it is well-tested.&#160; In an ideal world we’d only ever use the unit of code we’re testing, but I’ll often compromise.&#160; 

Depending on whether you think it’s a good idea to ever do a direct floating point comparison without a specified tolerance value (hint: it’s generally a bad idea) _and_ whether you think having an equals implementation for matrices is good design, either of these choices may be deemed unacceptable.&#160; 

I’d personally go for a third way™: write a testing extension that checks two matrices with a specified tolerance value.&#160; Something like this would do it:

> static void AssertEqualsWithinTolerance(Mtx4f a, Mtx4f b, float tolerance)

Add it to a testing helpers namespace and write a few tests for it.&#160; You can also extend some test runners to make it read more idiomatically.&#160; It’s important to write tests for utility / helper test code, as anything that uses it is making the assumption that it is correct and trustworthy.

## Be as Declarative as Possible

Leading on from this, I also advocate being as declarative as possible.&#160; Basically, in test code, we should endeavour to say _what_ the result should be, not _how_ to calculate it.&#160; Do your best to stick to this rule of thumb and it’ll save your life a thousand times over.&#160; I’ll talk about this more when I cover “[Don’t ape production code in tests](?p=731)” in a future post.

## Test Code is Simple Code

I sometimes hear folk griping about how much test code they have to write.&#160; However, writing code is generally not the bottleneck in productivity.&#160; We spend more time thinking, testing and debugging than we do actually _writing_ code. Furthermore, the simpler the code is in terms of its content, the more of it we can write and the easier it is to maintain. 

I say this because, for me, test code falls into the “simple” category.&#160; I know for a fact that I can bang out test code in a fraction of the time it takes to write production code.&#160; Why is that?&#160; Well, it’s because the test code is doing comparatively little of note.&#160; I don’t have to tweak loop boundary conditions, fix break/continue bugs, remedy branching errors and the like.&#160; 

That’s not to say that test code doesn’t have its own nuances and pitfalls, but once you internalise the rules to writing good tests, things go much more quickly.&#160; Most of the time sunk into tests tends to be invested in making them maintainable &#8212; comparatively little time is spent in fixing logical errors.&#160; 

## KISS on a Larger Scale

Sometimes it seems like a good idea to do something &#8216;clever&#8217; to reduce the amount of testing work we have to do. However, any time you hit upon such a scheme, take a step back and think about it carefully. I remember the early days of my time at RTW where I attempted to write a clone test that would find all types that subclassed a type T, clone them, then loop through their properties, checking for equality. It worked first time.&#160; I thought I must be some kind of genius. 

It backfired horribly, though. As new types were added, the test broke repeatedly. Due to all of the hoop jumping I was doing to discover and map the types, the test code was barely understandable and, due to its brittle nature, represented a maintenance risk.&#160; Debugging a failure was as clear as mud.

Ultimately, I deleted it and replaced it with simple, individual tests. They worked fine and were written inside a couple of hours &#8212; far less time than I spent dicking around trying to make a &#8216;clever&#8217; solution work.&#160; Oh and the simple solution never once broke.