---
id: 789
title: 'The fundamentals of unit testing : draw attention to &lsquo;interesting&rsquo; values'
date: 2014-08-07T21:51:25+00:00
author: Mark Simpson
layout: single
guid: http://defragdev.com/blog/?p=789
permalink: /?p=789
categories:
  - fundamentals of unit testing
  - testing
  - tips
---
3 is the magic number? No. No, it&#8217;s not. Magic numbers/strings/other values in all walks of programming are a readability and maintenance liability. It’s quite common to see all kinds of literals being littered through test code. It’s quite easy to simply switch off and treat test code as a second class citizen.

To illustrate this point, please tell me what this does

> [Test]  
> public void TestInvalidFuelAmount()  
> {  
> &#160;&#160;&#160; // Arrange  
> &#160;&#160;&#160; var car = CreateLittleCar(); 
> 
> &#160;&#160;&#160; // Act & Assert  
> &#160;&#160;&#160; var exception = Assert.Throws<ArgumentException>(() =>&#160; car.AddFuel(500));  
> }

It&#8217;s not exactly crystal clear, is it? Although the naming of this test can and should be improved, bear in mind that it&#8217;s an extremely simple case (one method call with one argument being passed). Despite this, the intent is still obscured because of a literal.

<a name="Replace_magic_values_with_named_consts"></a> 

## Replace literals with named consts 

Let&#8217;s change 500 to a named constant. Let’s also change the test name while we’re at it.

> [Test]  
> public void when\_the\_car\_fuel\_tank\_capacity\_is\_too\_low\_and\_fuel\_is\_added\_an\_exception\_is\_thrown()  
> {  
> &#160;&#160;&#160; // Arrange  
> &#160;&#160;&#160; var car = CreateLittleCar();  
> &#160;&#160;&#160; const int LitresOfFuelThatIsTooLargeForTankCapacity = 500; 
> 
> &#160;&#160;&#160; // Act & Assert  
> &#160;&#160;&#160; var exception = Assert.Throws<ArgumentException>(() =>&#160;  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; car.AddFuel(LitresOfFuelThatIsTooMuchForTankCapacity));  
> }

Now that&#8217;s much clearer. We can see what the value represents, the units it&#8217;s measured in and its place in the test. Some people shit the bed when they see really long story-like variable names in tests, but they’re generally a help, not a hindrance.

While the previous example is simple, this tip makes much more of a difference in situations where a function/method/constructor has numerous parameters, but the test is largely focused on only one of them. How do you know which one to pay attention to?

> [Test]  
> public void when\_account\_balance\_is\_low\_money\_transfers\_fail\_when\_would\_be_overdrawn()  
> {  
> &#160;&#160;&#160; // Arrange  
> &#160;&#160;&#160; var currentAccount = GetCurrentAccountWithBalanceInUSD(10); 
> 
> &#160;&#160;&#160; // Act&#160;&#160;&#160;&#160;  
> &#160;&#160;&#160; bool transferSucceeded = currentAccount.TryTransfer(1000, 42, 602402); 
> 
> &#160;&#160;&#160; // Assert  
> &#160;&#160;&#160; Assert.That(transferSucceeded, Is.False);  
> }

Fine, this is a crufty-looking API, but in certain circumstances, it’s quite common to see functions/constructors that take multiple arguments of a similar type. At a glance, how can you tell which of the arguments is the one of interest to the test? Is it 1000, 42 or 602402? What is the significance of any of them? Which one is causing the transfer to fail?

Let’s use a named const to draw attention to the interesting argument.

> [Test]  
> public void when\_account\_balance\_is\_low\_money\_transfers\_fail\_when\_would\_be_overdrawn()  
> {  
> &#160;&#160;&#160; // Arrange  
> &#160;&#160;&#160; var currentAccount = GetCurrentAccountWithBalanceInUSD(10);  
> &#160;&#160;&#160; const int AmountInUSDThatWillForceAccountIntoOverdraft = 1000;&#160;&#160;&#160;&#160; 
> 
> &#160;&#160;&#160; // Act&#160;&#160;&#160;&#160;  
> &#160;&#160;&#160; bool transferSucceeded = currentAccount.TryTransfer(AmountInUSDThatWillForceAccountIntoOverdraft , 42, 602402); 
> 
> &#160;&#160;&#160; // Assert  
> &#160;&#160;&#160; Assert.That(transferSucceeded, Is.False);  
> }

Turns out the 42 is the number of cents in the transfer, and that 602402 is just a dummy customer number we’re passing through. 

We can go further than this to clean things up still further by doing things such as:

  * Extract a wrapper test method that calls TryTransfer, but only has one parameter: the amount of USD to withdraw (the other parameters can be satisfied with constants). This draws attention to the important part of the test (varying the value of the USD transfer amount) and also helps insulate the tests from changes to the TryTransfer method. 
  * In the production code, using types that are more appropriate to reduce ambiguity (e.g. in the case of a customer account number, an AccountCredentials class would be more suitable than an int!) 
  * Finally, if lots of things are changed in the tests, but in slightly different ways each time, consider the use of a [Test Data Builder](?p=147). Test data builders are very useful, as the provided defaults are sensible for most cases. As a result, when arranging your test, the only things that are changed in the builder are the things that actually matter. Test data builders can also insulate the tests from breaking changes.