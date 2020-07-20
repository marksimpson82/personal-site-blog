---
id: 694
title: 'The fundamentals of unit testing : It&rsquo;s a skill'
date: 2012-10-24T02:00:04+00:00
author: Mark Simpson
layout: single
guid: https://defragdev.com/blog/?p=694
#permalink: /?p=694
categories:
  - fundamentals of unit testing
  - testing
  - tips
---
The first thing I’m writing about is probably the most important.&#160; This is a bit of a meandering tale, but it is crucial to understanding the pitfalls of automated testing.

## A Cautionary Tale

Back in the day at Realtime Worlds (the best part of 5 years ago), when it came to automated testing we were trying to pull ourselves up by our bootstraps.&#160; Most people (including myself) didn’t have much of a clue about automated testing.&#160; It was a pretty bold move for a games company to try it on a large scale, but it took a very long time for it to pay any benefits.&#160; In fact, given the lackadaisical approach and lack of buy-in, it probably had an overall negative effect in the majority of areas. 

In my first job fresh out of university, I had the nominal title of Junior Test Engineer.&#160; At first, it was our job to try and write extra tests that developers missed, debug failing tests and generally take care of the build.&#160; I soon learned to be very cynical.

As you can imagine, it wasn’t very effective (hint: the larger the distance between a person’s actions and the consequences of said actions, the less of an interest they will take).&#160; I would characterise it as a baptism of fire.&#160; Thanks to spending all day, every day, on automated testing, I acquired a large amount of knowledge by trial and error and learning with my colleagues.&#160; That knowledge was eventually ploughed back into the team in a supporting role.

<!--more-->

Anyway, when I first joined, it was common to see stuff like this:

> [Test]  
> public void TestInverse()  
> {  
> &#160;&#160;&#160; var m = Matrix.Identity;  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;  
> &#160;&#160;&#160; Assert.That(m.Inverse(), Is.EqualTo(m));  
> }

.. or worse, this:

> [Test]  
> public void Test1()  
> {  
> &#160;&#160;&#160; var m1 = Matrix.Identity;&#160;&#160;&#160;&#160;&#160;&#160;&#160;  
> &#160;&#160;&#160; Assert.That(m, Is.Not.Null);&#160;&#160;&#160;  
> &#160;&#160;&#160; var m2 = Matrix.Identity;&#160;&#160;&#160;  
> &#160;&#160;&#160; var m3 = Matrix.Multiply(m1, m2);  
> &#160;&#160;&#160; Assert.That(m3, Is.EqualTo(Matrix.Identity));  
> &#160;&#160;&#160; var m4 = m2 + m3;  
> &#160;&#160;&#160; Assert.That(m4, Is.EqualTo(new Matrix(&#160;&#160;&#160; 2, 0, 0, 0, 0, 2, 0, 0, 0, 0, 2, 0, 0, 0, 0, 0));  
> &#160;&#160;&#160; var m5 = m1 &#8211; m2;  
> &#160;&#160;&#160; Assert.That(m5, Is.EqualTo(Matrix.Zero));  
> }

These tests aren’t good.&#160; But they were _everywhere_.

This is not intended as a slight to whoever wrote tests like these because _they were all like that.&#160;_ Developers were told to write tests without sufficient guidance, support and – crucially – there were few consequences for writing bad test code, as the testers would scuttle out from their partition, produce a dustpan and broom and proceed to clear up after the developers.

The result was that we had a huge amount of test cases that provided very little value – probably negative value, in fact.&#160; The maintenance cost alone was huge.&#160; Furthermore, running the main test suite took the best part of 10 minutes with ~6000 tests.&#160; It should’ve taken 6 seconds.&#160; 20 if I’m being very generous.

## Templates of Failure

How did this state of affairs occur?&#160; Well, people were asked to write tests, but nobody knew much about it and few people took an active interest.

Person A wrote their feature and attempted to write some tests.&#160; After a touch of cursory reading, Person A wrote some tests and bathed in the green glow of the NUnit GUI’s progress bar.&#160; However, they didn’t really fully grasp what they were doing and, as time was pressing on, were keen to finish the task and move on to the next one.&#160; 

Person B – a programmer of similar experience when it came to testing &#8212; looked at what Person A had done.&#160; “That looks like the done thing!”, they cried while blithely repeating all of Person A’s mistakes and probably throwing in a couple of new ones, too.

This attitude is very common in companies.&#160; The first rule of code is that, once it’s checked in, it instantly becomes an order of magnitude harder to modify, correct and improve.&#160; Code hardens like concrete.&#160; The second rule of code is that other programmers will often use your code for reference, no matter the quality.&#160; Time is finite.&#160; If a programmer sees code that _seems_ to do the job and it is relevant to the problem they’re trying to solve, it’s fairly common that your shitty code becomes their shitty code.&#160; Anti-patterns propagate quickly.&#160; One by one, _they just started copying it_.&#160; 

What I’m trying to say is that, if shitty code is checked in, it can quickly become the de facto standard – a rigorously applied template of failure.&#160; The more alien the topic is, the more likely a developer is to engage in such behaviour.

## Slow down and think!

The reason I am saying this is that, at Realtime Worlds, the worst style of test code was the most prevalent, oft-aped and blindly repeated.&#160; People wanted to get their features in and understood neither the value nor the pitfalls of automated testing, as they received insufficient training or had no interest in it.&#160; Developers didn’t stop to think about what automated testing _is_ or what it’s _for_.&#160; We had thousands upon thousands of lines of largely worthless code that had to be maintained.&#160; It took a long time to change things.&#160; 

In the examples I gave, neither of those tests is useful.&#160; They prove very little, they’re hard to understand and difficult to maintain.&#160; The class under test is a mathematical one, too!&#160; Matrices have no dependencies.&#160; It doesn’t get much simpler than testing classes that have no dependencies.

## Automated Testing is a Skill

The key is to change how you think about testing.&#160; Testing is just like any other aspect of programming – if you want to reap the reward, you need to put in the effort to build up your skills.&#160; It’s a gradual ramp from “derp, I know nothing” up to the finer points of test data builders, different testing styles and whatnot.

It’s not just a case of including a framework, clattering out a few test methods and learning to click the buttons on the test runner GUI.&#160; If you don’t have the time to take it seriously and treat your test code as a first-class citizen, it’s probably not going to go particularly well.&#160; 

## The Bad News

If you do it badly on a large scale, here’s a list of just _some_ of the issues you can encounter:

  * The tests do nothing to prove that your software functions correctly 
  * The tests only prove that the ‘happy path’ works – unexpected data causes failures in production 
  * Some tests are not deterministic (they will periodically fail) 
  * Developers do not trust the tests.&#160; Test failures are routinely ignored 
  * Tests can have side-effects (so running test 3 after test 10 could cause a failure, but not vice-versa) 
  * Developers don’t understand what certain tests are trying to prove 
  * Even if developers understand the purpose of a test, it’s time-consuming to debug a failure as the test code is too convoluted 
  * The tests run extremely slowly, retarding productivity 
  * The tests routinely break (fail to compile or don’t pass when executed) when minor changes are made to production code.&#160; Developers throw their hands in the air and delete the tests rather than fix them. 
  * Developers cannot test swathes of production code, as it has intractable, unbreakable dependencies 
  * Developers tend to repeat huge amounts of code when writing tests. 
  * Tests pass when the production code is broken, as the test code apes the class under test. 
  * Tests pass regardless of the implementation being correct.&#160; I.e. the test is bugged, usually due to (incorrectly) performing calculations in the test code itself. 

## The Good News

The vast majority of these problems can be tamed by applying a handful of principles, which I will outline in further posts.