---
id: 731
title: 'The fundamentals of unit testing : Tests shouldn&rsquo;t ape the production code'
date: 2012-11-12T02:09:50+00:00
author: Mark Simpson
layout: single
guid: https://defragdev.com/blog/?p=731
#permalink: /?p=731
tags:
  - fundamentals of unit testing
  - testing
  - tips
---
Aping the logic of the class under test is an bad idea when writing tests, but it&#8217;s an easy mistake to make. 

## Aping&#8230; Bueller?

"Aping" simply means imitating/repeating the logic used in the class under test. In its simplest form, the problem can be illustrated like so: 

Production code:

> public class Calculator  
> {  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; public int Add(int a, int b)  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; {  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; return **a + b**; // actual logic of the class under test  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; }  
> }

A test for the above class:

> [Test]  
> public void adding\_two\_numbers\_returns\_the_sum()  
> {  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; // Arrange  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; var calculator = CreateCalculator();
> 
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; // Act  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; int result = calculator.Add(1, 2);
> 
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; // Assert  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; Assert.That(result, Is.EqualTo(**1 + 2**)); // ape alert, ooh ooh ooh! >:O  
> }

Although this doesn&#8217;t look like the end of the world, we can see that the test method line "Assert.That(result, Is.EqualTo(**1 + 2**))" is directly replicating the logic in the Add method.&#160; This is “aping”.

## Why is this a bad thing?

This is bad for a few reasons. Firstly, if the class under test is bugged and the test apes its logic to get an expected answer, we&#8217;ve just reproduced the bug in our test code and erroneously passed the test. So, instead of having working production code, we have two bits of bugged code and a false sense of security.

Secondly, when writing tests, if state can be tested, it should be done using as declarative a style as possible &#8212; we don&#8217;t want to say **how** we get to the answer, just **what** it should be. 

I don&#8217;t want to see the test code literally add 1 and 2 to get 3. Instead, I want to see 3 in the test.&#160; Here’s a fixed-up version of the test that does not ape the production code:

> [Test]  
> public void adding\_two\_numbers\_returns\_the_sum()  
> {  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; // Arrange  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; var calculator = CreateCalculator();
> 
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; // Act  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; int result = calculator.Add(1, 2);
> 
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; // Assert  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; Assert.That(result, Is.EqualTo(3));&#160;&#160;&#160;  
> }

You should always scan your test code looking for this anti-pattern.&#160; The example I’ve cited is very simple, but I’ve seen a lot of real world code that has succumbed to aping the production code resulting in bugs going undiscovered. 

## Validating your answers

You may be thinking, “OK genius, so you skipped adding 1 and 2 in the test code, but you must’ve known to follow the addition algorithm in your head to be able to come up with the answer of 3.&#160; How do you know your brain didn’t get it wrong and the test is bugged?”&#160; 

It’s a good question and the truth is, it’s often the case that it’s not clear-cut.&#160; However, there are steps you can take to validate your answers (or at least be a little more sure of them).

  * Don’t just assume you’ve got it right first time.&#160; Double check it for a few minutes more. 
  * Write out some notes on paper (I sometimes do the testing version of truth tables). 
  * Maybe you have a specification for this particular part.&#160; Use it to check the answer&#160; (stop laughing at the back, there! – it occasionally does happen; e.g. conventions for a maths library). 
  * Use an alternative method to come up with the same answer (it’s often the case that the same value can be calculated via multiple algorithms / schemes). 
  * Use an existing implementation to corroborate your findings.&#160; E.g. if you’re writing a test to make sure that JSON is validated correctly, then check that [JSONLint.com](http://jsonlint.com/) agrees with your test results. 
  * If you’re not sure how to corroborate an exact answer, then check any invariants you’re aware of as a smoke test.&#160; If any of the invariants don’t hold, your answer is probably wrong.