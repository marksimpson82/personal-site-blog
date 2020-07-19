---
id: 675
title: 'Functional-style programming &ndash; Easy debugging!'
date: 2012-07-07T21:34:15+00:00
author: Mark Simpson
layout: single
guid: http://defragdev.com/blog/?p=675
permalink: /?p=675
categories:
  - 'c#'
  - debugging
  - MyWorld
  - patterns
  - software
  - tips
---
Ever since LINQ arrived in C# land, a lot of people have been using a functional style of programming without even perhaps being conscious of it.&#160; 

I always chuckle when I read the words “functional style”, because I picture grim marches across a barren landscape, brutalist architecture and people with beards.&#160; 

One particular facet of programming in a functional style is where, rather than mutating an existing piece of data, input data is transformed / projected into something new via a [pure function](http://en.wikipedia.org/wiki/Pure_function).&#160; Since this style of programming limits the number of side-effects, it makes debugging a comparatively wonderful experience.&#160; 

<!--more-->

## The status quo

First, let’s consider business-as-usual.&#160; Imagine you’re debugging a part of your program that is doing something complicated.&#160; The program is written in the usual imperative, OO style; that is to say, when method calls occur, state is mutated.

It’s often the case that an object with state either mutates itself, or is mutated by other objects.&#160; Once that code has been executed, the old object state is gone.&#160; It’s done one.&#160; It ain’t comin’ back, Billy.&#160; When you observe the result and see a problem, the data trail goes cold.&#160; All you can see is the state of the object _after_ it has been mutated.&#160; Sometimes you can work your way backwards, but in other cases the mental gymnastics are too complicated.&#160; Sometimes you weep hot salty tears.

To reproduce the bug, you need to get the program back into its original state and observe proceedings as they happen.&#160; This can be tricky, especially when the reproduction steps are unknown.

## Step forward, functional style

If you’re simply transforming some data D by some function F, you get something like F(D) = D\`.&#160; D’ is a new object or structure.&#160; The original input data D remains untouched.&#160; If your program is incorrect after the transformation, all of the original state still exists.&#160; 

You can simply break in the VS debugger then use the “Set Next Statement” to rewind the execution to an arbitrary point.&#160; As long as the operations have no side effects, you can do this as many times as you want or to any depth.&#160; It’s almost like you recorded the drama on tape and rewound it.

## 

## Stuff wot I’m doing

The current program I’m working on is developed using a series of stages.&#160; It goes something like this:

“Take this raw json input, build a graph.&#160; Take the graph, build a higher level representation with bells on.&#160; Take the higher level representation with bells on, build a triangle mesh with texture coordinates.”

At pretty much every stage, I am transforming input data into something new rather than mutating an existing structure.&#160; There is precisely one part where I mutate data.&#160; Guess where I spent the most time debugging it when unexpected input data caused it to crash?&#160; Yup, the mutation bit.&#160; 

We have terabytes of data to crunch.&#160; It is vital to be able to reproduce crashes in a deterministic fashion.

I’m not claiming this style of programming is a silver bullet or applicable everywhere, but it does have its charms.