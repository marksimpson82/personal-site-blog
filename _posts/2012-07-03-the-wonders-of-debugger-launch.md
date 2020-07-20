---
id: 668
title: The wonders of Debugger.Launch()
date: 2012-07-03T22:46:16+00:00
author: Mark Simpson
layout: single
guid: https://defragdev.com/blog/?p=668
#permalink: /?p=668
categories:
  - 'c#'
  - debugging
  - software
  - tips
---
Ever worked on a project that involved spawning new .NET processes? (as in, one [arbitrary] program launches another .NET executable)&#160; I’ve had to do this on quite a few occasions over the years and the one thing that always saves my bacon when it comes to understanding and fixing bugs is [Debugger.Launch()](http://msdn.microsoft.com/en-us/library/system.diagnostics.debugger.launch.aspx).

A common scenario is as follows:

  * Program / Script A is launched
  * Program / Script A makes a call to launch .NET Program B (e.g. via [Process.Start()](http://msdn.microsoft.com/en-us/library/system.diagnostics.process.start.aspx))
  * .NET Program B throws an exception and it’s not immediately clear why.
  * The world ends.&#160; 

If you’re the author of program B, simply insert a call to Debugger.Launch() inside Main().&#160; The program will halt execution and prompt you to attach the debugger.&#160; You can then examine the conditions and fix the bug.&#160; 

Also, another thing to try is to create a handler that wraps the program’s invocation in a try/catch block, complete with an #if wrapped call to Debugger.Launch().&#160; This allows you to connect to the crashing process without requiring lots of boilerplate code.

I wouldn’t recommend using this for production code in case you forget to remove it (add conditional #if guards and whatnot), but it’s nice to have in the toolbag nonetheless.

E.g.:

> public static class AttachableRunner  
> {&#160;&#160;&#160;  
> &#160;&#160;&#160; public static void RunWithDebugger(Action _action)  
> &#160;&#160;&#160; {  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; try  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; {  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; _action();  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; }  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; catch(Exception e)  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; {  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; Debugger.Launch();  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; throw;  
> &#160;&#160;&#160;&#160;&#160;&#160;&#160; }  
> &#160;&#160;&#160; }  
> }

This does just fine for debugging most problems.&#160; 

**Note**: It won’t attach properly if your program causes an unhandled exception to be thrown on a a thread other than the main thread. I’ll leave that as an exercise for the reader (\*cough\* [AppDomain.UnhandledException](http://msdn.microsoft.com/en-us/library/system.appdomain.unhandledexception.aspx) can help, but won’t give you as much useful information beyond standard callstack info).