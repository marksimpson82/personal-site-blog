---
id: 633
title: 'Unity3d &#8211; Profiling calls to Destroy'
date: 2011-08-31T16:39:01+00:00
author: Mark Simpson
layout: single
guid: https://defragdev.com/blog/?p=633
#permalink: /?p=633
categories:
  - 'c#'
  - debugging
  - software
  - tips
  - Unity3d
---
## Destroy All … Things

I’ve recently been investigating some performance problems in our Unity3d app.&#160; In one specific instance (running full screen on a rubbish laptop), there were numerous large performance spikes caused by the nebulous-sounding “Destroy”.&#160; After prodding a little bit, Destroy is related to calling [GameObject.Destroy()](http://unity3d.com/support/documentation/ScriptReference/Object.Destroy.html).

Unfortunately, Unity3d’s profiler won’t give you any information related to what types of objects are being destroyed, and wrapping your GameObject.Destroy calls in profiler sections doesn’t help, as Unity3d defers the work for later in the frame (so the methods return nigh-on immediately).&#160; As such, you get told “Destroy is taking x ms this frame”, and that’s about it.

## Finding the culprits with the profiler

I managed to work around this limitation by (temporarily) changing all GameObject.Destroy() method calls to [GameObject.DestroyImmediate()](http://unity3d.com/support/documentation/ScriptReference/Object.DestroyImmediate.html) then wrapping the calls in [Profiler.BeginSample()](http://unity3d.com/support/documentation/ScriptReference/Profiler.BeginSample.html) / Profiler.EndSample() pairs.

**Note:** If you access your unity instances in the same frame after calling destroy, this probably won’t work for you.

It was then possible to see which resources were doing the damage on the laptop.&#160; All of our resource cleanup code was in one place, so it was trivial to do.

The temporarily instrumented code ended up looking something like this, and immediately let us know the culprits.&#160; Note this code is just a simplified mockup, but it should give you the gist of the idea:

> <font size="2" face="Consolas">// centralised resource cleanup makes profiling simple <br />private void CleanupResources<TResource>() <br />{ <br />&#160;&#160;&#160; Profiler.BeginSample("Destroy: " + typeof(TResource).Name); <br />&#160;&#160;&#160; IEnumerable<TResource> resources = FindResourceOfType(typeof(TResource)); <br />&#160;&#160;&#160; foreach(var resource in resources) <br />&#160;&#160;&#160; { <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160; resource.Dispose(); <br />&#160;&#160;&#160; } <br />&#160;&#160;&#160; Profiler.EndSample();&#160;&#160;&#160; <br />}</font>
> 
> <font size="2" face="Consolas">//&#8230; and each Resource type inherits from a common base class, implementing IDisposable. <br />public abstract class Resource : IDisposable <br />{ <br />&#160;&#160;&#160; protected abstract void CleanupUnityResources(); <br />&#160;&#160;&#160; <br />&#160;&#160;&#160; public void Dispose() <br />&#160;&#160;&#160; { <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160; CleanupUnityResources(); <br />&#160;&#160;&#160; } <br />}</font>
> 
> <font size="2" face="Consolas">public class SomeResource : Resource <br />{ <br />&#160;&#160;&#160; private Mesh m_unityMesh; // gets set when resource is locked in <br />&#160;&#160;&#160; <br />&#160;&#160;&#160; protected override void CleanupUnityResources() <br />&#160;&#160;&#160; { <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160; // GameObject.Destroy(m_unityMesh); <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160; GameObject.DestroyImmediately(m_unityMesh); <br />&#160;&#160;&#160; } <br />}</font>
> 
> <span style="font-family: consolas"></span>