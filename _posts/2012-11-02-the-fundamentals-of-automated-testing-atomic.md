---
id: 714
title: "The fundamentals of unit testing: Atomic"
date: 2012-11-02T05:47:01+00:00
author: Mark Simpson
layout: single
guid: https://defragdev.com/blog/?p=714
#permalink: /?p=714
tags:
  - fundamentals of unit testing
  - testing
  - tips
---
Every single unit test you write should be atomic.&#160; 

## A definition

Atomic tests are order-independent, relying on and causing no side effects.&#160; They neither rely on nor alter shared state; they do not mutate their environment. If a test run involves 2, 10, 100 or 1,000,000 tests, the tests should be able to be executed in any order.&#160; If one legitimately fails because a bug has been introduced in the production code, it shouldn’t have a domino effect. 

For example, if we have 3 tests named A, B and C, we should be able to run these tests in _any_ order.&#160; For a tiny set of 3 tests, there are already 6 ways of arranging the order of execution: ABC, ACB, BAC, BCA, CAB, CBA.&#160; If the order ABC is fine, but the order BAC causes test C to fail, then one of the tests is non-atomic.&#160; 

Put short, a test should be built up and torn down cleanly, leaving no trace of the fact that it has been executed.&#160; 

## Why is this important?

A good automated test has a descriptive name, is implemented using straightforward code and leads us to the cause of the failure without much hassle.&#160; When dealing with well written atomic tests, at the very worst, diagnosing the cause of a failure should be a simple debugging job.

However, if non-atomic tests create knock-on effects it introduces a significant maintenance risk, as to diagnose failures, we must understand the whole picture.&#160; Instead of just inspecting the local state that exists prior to executing the failing test, we have to start delving into environmental differences (machine configuration, file systems) and [AppDomain](http://blogs.msdn.com/b/cbrumme/archive/2003/04/15/51317.aspx) issues (typically global / static state).

<!--more-->

While most test failures caused by a lack of atomicity are localised (e.g. tests confined to one fixture start to fail), I&#8217;ve personally witnessed a system under test causing hundreds of test failures spanning multiple (seemingly unrelated) fixtures.

Furthermore, if failures cannot be reliably reproduced, it undermines the trust in those tests. People won&#8217;t pay attention to them, even if the failures are legitimate.&#160; The moment developers see a red build and, without even thinking about it, click the “rebuild” button, you’re entering the world of [NOMFUP](http://www.urbandictionary.com/define.php?term=NOMFup).&#160; 

## Tell Tale Signs

How can you tell that you have non-atomic tests lurking in your codebase? 

#1: Test runners don&#8217;t always find and run tests in the same order &#8212; they are often loaded in an arbitrary order. A good example of this phenomenon is the disparity between ReSharper and NUnit. They both execute tests in different orders (or they did last time I checked with a large sample set).&#160; If different results can be obtained by running the same tests using different runners, it’s probably caused by non-atomic tests.

#2: Running your entire test suite or a subset of it with the **same instance** of the test runner may cause failures the Nth time around.&#160; E.g. in .NET land: open the NUnit GUI, load your tests and then repeatedly run the tests.

**Note**: There is a combinatorial explosion involved with order dependence, so these two signs aren’t guaranteed to present themselves..&#160; The best defence is to write production & test code bearing a few things in mind.

## Cause: Static State in Production Code

The biggest offender by far is static state mutated / read by the _production_ code.&#160; For most intents and purposes, static is just “global” by another name.

If we have a class under test like the following (please note: do not write code like this!):

> public class ModelLoader  
> {  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; public Model Load(string xml)  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; {  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &#8230; // some processing occurs then…  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; var handle = **MemoryManager.Allocate**(numVertices * vertexSize); // a **static call**  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; }  
> }

&#8230; and the static dependency:

> public **static** class MemoryManager  
> {  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; private static MemoryPool m_memoryPool;
> 
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; public **static** void Init()  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; {  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; // does initialisation stuff.  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; m_memoryPool = new MemoryPool();  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
> 
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; public **static** MemoryHandle Allocate(long numBytes)  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; {  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; // this will crash if Init wasn&#8217;t called&#160;  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; return m_memoryPool.DoSomething();  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
> 
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; public **static** void Quit()  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; {&#160;  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; // poor man’s Dispose()&#160;  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; }

&#8230; then ModelLoader has an _implicit_ dependency on the MemoryManager.&#160; It&#8217;s implicit because there is nothing in the public API to say that this static class MemoryManager is used, let alone that it must be initialised if we want to use the ModelLoader. As Misko says, [Singletons are pathological liars](http://misko.hevery.com/2008/08/17/singletons-are-pathological-liars/).&#160; 

As a result, if we forget to call the static Init() or Quit() methods, the state mutations caused by one test will bleed into the next.&#160; 

Here’s the (horribly non-atomic) test code:

> [TestFixture]  
> public class StaticStateHurtsMyBrainTests  
> {&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; [Test]  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; public void loading\_simple\_model\_xml\_returns\_valid\_model()  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; {  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; // Arrange  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; **MemoryManager.Init(); // static state initialised here**!  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; var modelLoader = new ModelLoader();
> 
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; // Act. Our class under test depends on implicit static state contained in MemoryManager  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; var loadedModel = modelLoader.Load("<Model Name = &#8230; etc ");
> 
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; // Assert  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; Assert.That(&#160; // **Note: we forgot to call MemoryManager.Quit()**  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
> 
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; [Test]  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; public void loading\_model\_with\_positions\_and\_uvs\_returns\_valid\_model()  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; {  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; // Arrange.&#160; **Uh oh, we forgot to call MemoryManager.Init()&#160;  
>** &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; var modelLoader = new ModelLoader();
> 
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; // Act. MemoryManager isn&#8217;t initialised; test will fail here. Or will it?!  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; var loadedModel = modelLoader.Load("<Model Name = &#8230;&#160; etc");
> 
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; // Assert  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; Assert.That(&#160; // etc.  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; }  
> } 

We’re now in a world of pain.&#160; Depending on the order the tests are&#160; run _and_ whether individual tests pass or fail, we may obtain different results.&#160; Can you see why? The author of the first test has correctly ascertained that the nasty (hidden) MemoryManager dependency must be initialised, but doesn’t shut it down correctly.&#160; Since the MemoryManager is a static class, its state remains in the AppDomain from test to test.&#160; As a result, the second test will possibly pass, but only if it runs after the first.&#160; If the order is switched, we’ll definitely have a minimum of a single failure.&#160; These tests are non-atomic.

## Fix: Remove Static State

To solve this problem, the static state should be replaced with instance-based construction and any functionality that needs access to it should explicitly ask for it via the constructor or other means, like so:

> public class ModelLoader  
> {  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; private MemoryManager m_memoryManager;  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; public ModelLoader(MemoryManager memoryManager)  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; {  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; m_memoryManager = memoryManager;  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; }  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; public Model Load(string xmlString)  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; {  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; &#8230; // some processing occurs .. then  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; m_memoryManager.Allocate(numVertices * vertexSize);  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; }  
> }
> 
> public class MemoryManager // probably implement IDisposable too, but it’s just an example so…  
> {  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; private MemoryPool m_memoryPool;
> 
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; public MemoryManager()  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; {&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; m_memoryPool = new MemoryPool();  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
> 
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; public MemoryHandle Allocate(long numBytes)  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; {&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; return m_memoryPool.DoSomething(numBytes);  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
> 
> }

This greatly simplifies the complexity by making dependencies explicit, limiting access and removing the carried over state.&#160; We have ensured that the ModelLoader has a properly constructed MemoryManager dependency and that it can be constructed and thrown away without causing side effects.&#160; 

Do not simply refactor the test code such that it calls static methods before/after each test is run.&#160; This is just a bandage over a horrible API.&#160; Static state is a bit like playing whackamole – you can thrash most of it into shape, but there’s always a point where it becomes too much to handle.&#160;&#160;&#160; 

Tests are microcosms of real usage &#8212; if something is hard to use in a test, it is hard to use, period. Unless you _really_ have a need for a static type and you are well-versed in the risks, avoid it.

## Cause: Test Depends on Filesystem

Any test that is dependent on its environment is an integration test. The most common way the environment manifests itself is when dealing with the file system.&#160; The filesystem is the filesystem.&#160; It persists before, during and after a test run, so it stands to reason that modifying it can cause knock-on effects.

> [TestFixture]  
> public class OrderDependentTests  
> {  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; private const string Filename = "myfile.txt";
> 
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; [Test]  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; public void when\_the\_file\_exists\_constructor\_loads\_the\_file\_contents()  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; {  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; // Arrange  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; File.Create(Filename);  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; AddThreeElementsToFile(Filename); // imagine this opened the file and wrote something in
> 
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; // Act  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; var systemUnderTest = new TypeLoader(Filename);
> 
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; // Assert  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; Assert.That(systemUnderTest.LoadedTypes, Has.Count.EqualTo(3));  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
> 
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; [Test]  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; public void when\_file\_exists\_that\_contains\_only\_comments\_constructor\_doesnt\_load\_anything()  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; {  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; // Arrange  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; AddCommentToFile(Filename); // this requires the file to exist! Does it? Maybe.
> 
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; // Act  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; var systemUnderTest = new TypeLoader(Filename);
> 
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; // Assert  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; Assert.That(systemUnderTest.LoadedTypes, Has.Count.EqualTo(0));  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
> 
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; [Test]  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; public void when\_file\_does\_not\_exist\_constructor\_doesnt\_load\_anything()  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; {  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; // Arrange  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; File.Delete(Filename); // what happens here if the previous test(s) didn’t run yet?
> 
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; // Act  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; var systemUnderTest = new TypeLoader(Filename);
> 
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; // Assert  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; Assert.That(systemUnderTest.LoadedTypes, Has.Count.EqualTo(0));  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; }  
> }

Firstly, you should note that these tests touch the file system and are really integration tests. Secondly, if these tests run in the order they&#8217;re defined, they&#8217;ll pass. However, if the order is mixed up, or the test that relies on the file existing is run in isolation, the test(s) will fail! 

## Fix: Don’t Rely on the Filesystem

For unit tests, abstract the production code’s file IO by at least one level.&#160; By this, I mean separate the loading & processing of files and their contents.&#160; You can still present file loading APIs to the user, but the guts of the implementation uses streams, allowing fast, in-memory unit testing for the more complicated ‘inner’ code.&#160; 

I wrote a post about this a while back, so [take a look](?p=438).