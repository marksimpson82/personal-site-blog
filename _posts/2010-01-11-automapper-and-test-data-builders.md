---
id: 474
title: AutoMapper and Test Data Builders
date: 2010-01-11T23:07:19+00:00
author: Mark Simpson
layout: single
guid: http://defragdev.com/blog/?p=474
permalink: /?p=474
categories:
  - 'c#'
  - patterns
  - testing
  - tips
---
I&#8217;ve recently been tinkering with WCF and, as many people already know, writing data transfer objects is a pain in the balls.  Nobody likes writing repetitive, duplicate and tedious code, so I was delighted when I read about [AutoMapper](http://www.codeplex.com/AutoMapper).  It works really nicely;  with convention over configuration, you can bang out the entity => data transfer code in no time, the conversions are less error prone, the tests stay in sync and you&#8217;re left to concentrate on more important things.

Anyway, I immediately realised that I&#8217;ve used the same pattern in testing &#8212; with property initializers & test data builders.  I&#8217;ve [posted before](?p=147) about Test Data Builders and I&#8217;d recommend you read that post first.

For small test data builder classes, it&#8217;s really not that big a deal.  For larger classes, using AutoMapper is quite useful.  For example, for testing purposes we&#8217;ve got an exception details class that is sent over to an exception logging service.

Every time the app dies, we create an exception data transfer object, fill it out and then send it over the wire.  When unit testing the service, I use a Test Data Builder to create the exception report so that I can vary its properties easily.  Guess what?  The test data builder&#8217;s properties map 1:1 with the exception report &#8212; hmm!

So, rather than create the same boilerplate code to map the 10+ properties on the exception builder => data transfer object, I just used AutoMapper to handle the mapping for me :)

<pre>public class ExceptionReportDto
{
    public string ExceptionType { get; set; }
    public string StackTrace { get; set; }
    public string AssemblyName { get; set; }
    public string EntryPoint { get; set; }
    public string UserName { get; set; }
    public string MachineName { get; set; }
    // etc
}</pre>

<pre>public class ExceptionReportBuilder
{
   public string ExceptionType { get; set; }
   public string StackTrace { get; set; }
   public string AssemblyName { get; set; }
   public string EntryPoint { get; set; }
   public string UserName { get; set; }
   public string MachineName { get; set; }
   // etc

// create the mapping when the static ctor is invoked
static ExceptionReportBuilder()
}
   Mapper.CreateMap&lt;ExceptionReportBuilder, ExceptionReportDto&gt;();
}

public void ExceptionReportDto()
{
    // set up defaults
    ExceptionType = "System.ArgumentException";
    StackTrace = "Oh no I am a stack trace";
    //etc.
}

 public ExceptionReportDto Build()
 {
     // go go automagic!
     return Mapper.Map&lt;ExceptionReportBuilder, ExceptionReportDto&gt;(this);
 }
}</pre>

I&#8217;ve had good results with this approach.  The only bit I&#8217;m remotely concerned about is creating the mapping in the static constructor.  Any AutoMapper gurus out there who can say whether there&#8217;s any reason I shouldn&#8217;t do that?