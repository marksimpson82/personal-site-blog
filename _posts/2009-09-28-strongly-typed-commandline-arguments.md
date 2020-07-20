---
id: 393
title: Strongly typed commandline arguments
date: 2009-09-28T01:19:24+00:00
author: Mark Simpson
layout: single
guid: https://defragdev.com/blog/?p=393
#permalink: /?p=393
categories:
  - 'c#'
---
I&#8217;ve read quite a bit about [Static Reflection](http://blog.jagregory.com/2009/01/26/introduction-to-static-reflection/ "Static Reflection") and found it to be very appealing, but I hadn&#8217;t used it&#8230; until now!  Please have a quick look at the article, as I&#8217;m not going to parrot its key points, I&#8217;m going to write something that is horrendously over-engineered to solve a trivial problem, instead!  P.s. I apologise for interchanging arguments/parameters throughout this post.  My attention span is akin to that of a hey did anyone play Batman yet?

A bit of background on something I&#8217;m working on:  I have a c# app that is responsible for starting other processes.  Every now and then, the arguments/parameters go out of sync &#8212; I change the parameter list in the callee process and the caller, with its piddly weakly-typed guesses, causes the callee to bomb out as the arguments supplied do not match the parameters required.

E.g. I&#8217;d write something like:

<pre>public class ProcessFactory
{
    public Process CreateProcess()
    {
        // lots of stuff happens, then I try to make the arguments string
        // to set as part of the process start info
        string arguments = string.Format(
            "-IceCream:{0}", numberOfScoops);
    }
}</pre>

But oh, oh no!  I&#8217;d renamed &#8220;IceCream&#8221; to &#8220;JimmyNeedsSomeIceCream&#8221; or removed it.  Since there is no communication between the two processes, it&#8217;s hard to write an integration test that proves all is well and, besides, most of the time it&#8217;s just down to me renaming or removing an argument.  When it bombs out, I then have to go setting breaks points in the callee or trawling through log files.  Not ideal.  I&#8217;d rather the compiler told me when there was an obvious problem.  So, my challenge was to make the command line arguments more robust.

### You need a target to hit

Firstly, I&#8217;ll say that if you have non-trivial arguments to parse, the first piece of the puzzle [is to grab a good parser](http://stackoverflow.com/questions/491595/best-way-to-parse-command-line-arguments-in-c).  I use one written by [Peter Hallam](http://blogs.msdn.com/peterhal/archive/2004/10/23/246731.aspx) (the link on his blog forwards you to a defunct site, but you can find the source in loads of open source projects) which works really nicely. I think I&#8217;d rather stop a bus with my face than write another crap command line parser, so it&#8217;s always nice to drop an existing, proven one in.

Here&#8217;s an example arguments class.  Notice that the types are not strings, they can be any simple type that can be parsed:

<pre>public class TargetProcessArguments
{
    /// &lt;summary&gt;Allow multiple instances?&lt;/summary&gt;
    [Argument(ArgumentType.AtMostOnce, HelpText = "I'm a knife.  Knifin' around.")]
    public int NumberOfScoops;

    // etc. and so forth
}</pre>

Anyway, now you have the parameter names, half of the battle is won.  Move the arguments class into a common assembly that both the caller and the callee can access.  The next step is using those parameter names in a strongly typed fashion.

### Latch on to the target

For this, static reflection fits the bill.  Using a slightly modified version of the static reflection code, you can already write code like this:

<pre>[Test]
public void GetName_WithValidField_ReturnsFieldName()
{
    var fieldName = StaticReflection.GetName&lt;TestClass&gt;(x =&gt; x.TestField);
    Assert.That(fieldName, Is.EqualTo("TestField"));
}</pre>

In the above snippet, I&#8217;m getting the name of a field or property of a type using static reflection.  If the field or property disappears, the compiler will tell me.  If it is renamed, the code will update as with normal refactoring.

Here&#8217;s my barely modified example based on the first link in this post:

<pre>/// &lt;summary&gt;
/// http://www.lostechies.com/blogs/gabrielschenker/archive/
/// 2009/02/03/dynamic-reflection-versus-static-reflection.aspx
/// &lt;/summary&gt;
public static class StaticReflection
{
    /// &lt;summary&gt;
    /// Works with either a property or
    /// a field (simplifies use)
    /// &lt;/summary&gt;
    public static string GetName&lt;TEntity&gt;(
            Expression&lt;Func&lt;TEntity, object&gt;&gt; expression)
    {
        var memberExpression = GetMemberExpression(expression);
        return memberExpression.Member.Name;
    }

    private static MemberExpression GetMemberExpression&lt;T&gt;(
         Expression&lt;Func&lt;T, object&gt;&gt; expression)
    {
        MemberExpression memberExpression = null;
        if (expression.Body.NodeType == ExpressionType.Convert)
        {
            var body = (UnaryExpression)expression.Body;
            memberExpression = body.Operand as MemberExpression;
        }
        else if (expression.Body.NodeType == ExpressionType.MemberAccess)
        {
            memberExpression = expression.Body as MemberExpression;
        }

        if (memberExpression == null)
        {
            throw new ArgumentException(
                "Not a member access", "expression");
        }

        return memberExpression;
    }
}</pre>

Let&#8217;s take a bit further, Jimmy.  It doesn&#8217;t take much effort at all to write a command line argument builder based on the same principles.  The existing Static Reflection code is operating on an expression tree.  All we need to do is receive an expression tree from the user and (if applicable, a value to go with the param name) and add it to our argument string.

### 5 minutes later

I wrote a 5 minute job that contained a stringbuilder and added some formatting and a nice fluid interface.  Behold!

<pre>[Test]
public void TestParamPair_WithFieldExpression_ParamPairIsWrittenCorrectly()
{
    string result = new CommandLineBuilder&lt;TestArguments&gt;()
        .ParamPair(x =&gt; x.NumberOfScoops, "3")
        .Build();

    Assert.That(result, Is.EqualTo("-NumberOfScoops:3"));
}</pre>

So there we go.  I can now modify my command line arguments in one project and all callers will be brought up to speed at compile time (or the compiler will tell you that they&#8217;ve gone out of synch).  Of course, they might still be semantically incorrect, but hey, you can&#8217;t have everything.

You could probably extend this further by adding extra validation to the values of the arguments or by setting it on fire.