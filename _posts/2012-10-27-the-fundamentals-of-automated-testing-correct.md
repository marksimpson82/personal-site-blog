---
id: 704
title: 'The fundamentals of unit testing : Correct'
date: 2012-10-27T19:50:56+00:00
author: Mark Simpson
layout: single
guid: http://defragdev.com/blog/?p=704
permalink: /?p=704
categories:
  - fundamentals of unit testing
  - testing
  - tips
---
Once you’ve decided what you’re going to test, chosen a good name and started to write the code, the next step is to make sure that the test is purposeful and correct.&#160; This is a simple tip and largely common sense.

## Does the test code’s functionality match its name?

It is simple to accidentally write a test that purports to test one thing, but actually tests another. E.g. 

> [Test]  
> public void model\_is\_loaded\_correctly\_from_stream()  
> {  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; // Arrange  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; var resourceLoader = CreateResourceLoader();  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; var modelDataStream = CreateModelDataStream();
> 
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; // Act  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; var loadedResource = resourceLoader.LoadFromStream(modelDataStream); // fine so far
> 
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; // Assert  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; Assert.That(resourceLoader.LoadedResources, Is.EqualTo(1)); // buh?!  
> }

In this test, we can see that it claims to test that loading a model from a stream returns a valid resource. However, the test is then asserting that the loaded resources count has increased. It is not checking the returned resource for correctness.&#160; Either the test name is out of date, or it is not testing the correct thing.&#160; 

If it is the former case, we should rename the test to “loading a model increments the loaded resources count”.&#160; If it is the latter case, we should change the test such that we are asserting on the model’s state.&#160; How do you know the model has been loaded correctly if you’re not inspecting its vertices, indices and UVs?

If we are misled by our test names, it is irritating because it looks like something is covered, but it’s not.&#160; If we’re asserting on the wrong stuff, then our test won’t save us when the unit of code malfunctions.&#160; Maybe it’s already bugged.

## Is the Assert meaningful?

All tests should assert against some meaningful result. Put simply, if a test does not prove much, then it is not worth writing or maintaining.

Here is an example of a test that looks useful but, on closer inspection, is doesn&#8217;t really prove much at all. 

> [Test]  
> public void game\_properties\_are\_correctly\_serialized()  
> {  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; // Arrange  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; var currentLevel = 4;  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; var playerName = "Steve";  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; var game = CreateGame(playerName, currentLevel);&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; var serializer = CreateGamePersistence();  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; using(Stream stream = new MemoryStream())  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; {  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; // Act  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; serializer.Save(game, stream);  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; }  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; Assert.That(true, Is.True) // WHAT?!?!?  
> }

I’ve seen the “Assert true == true” thing numerous times.&#160; If the assert is missing or tells us nothing useful, the chances are the test isn’t going to prove much, so why are some developers compelled to do it? My guess is that people think, “hmm, I’ve written a test with no asserts, there we go”.&#160; If you ever get tempted to do this, don’t!&#160; It’s a sign that something is wrong.

**Note**: it’s sometimes useful to have methods like this to act as surrogate Main() methods so we can invoke them via a test runner for easy debugging but, in general, it’s not a terribly good idea and you wouldn’t want to run these tests as a part of your normal development workflow.