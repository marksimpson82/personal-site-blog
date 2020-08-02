---
id: 467
title: Debug.Assert vs. Exceptions
date: 2010-01-09T18:53:30+00:00
author: Mark Simpson
layout: single
guid: https://defragdev.com/blog/?p=467
#permalink: /?p=467
tags:
  - Uncategorized
---
<div>
  <p>
    <strong> </strong>
  </p>
  
  <p>
    &#8220;When should I use Debug.Assert and when should I use exceptions?&#8221; &#8212; It&#8217;s a fairly sensible question to ask, but you&#8217;ve got to sift through a lot of articles to get anything resembling solid guidance on it (<a href="http://stackoverflow.com/questions/1467568/debug-assert-vs-exception-throwing/1468385#1468385">Eric Lippert&#8217;s stack overflow post is particularly enlightening</a>).  I&#8217;ve wrestled with it quite a bit as a programmer and test engineer, so here&#8217;s my 2 pence.
  </p>
  
  <p>
    Good rules of thumb I&#8217;ve arrived at:
  </p>
  
  <ol>
    <li>
      Asserts are not a replacement for robust code that functions correctly independent of configuration. They are complementary debugging aids.
    </li>
    <li>
      Asserts should never be tripped during a unit test run, even when feeding in invalid values or testing error conditions. The code should anticipate and handle these conditions without an assert occurring!
    </li>
    <li>
      If an assert trips (either in a unit test or during normal application usage), the class containing the assert is the prime suspect, as it has somehow managed to get into an invalid state (i.e. it&#8217;s bugged).
    </li>
  </ol>
  
  <p>
    For all other errors &#8212; typically down to environment (network connection lost) or misuse (caller passed a null value) &#8212; it&#8217;s much nicer and more understandable to use hard checks & exceptions. If an exception occurs, the caller knows it&#8217;s likely their fault.  This is what makes the .NET base class libraries a joy to develop with &#8212; it usually clear when you are misusing an API, resulting in fewer <a href="http://www.pragprog.com/the-pragmatic-programmer/extracts/tips">&#8220;select is broken&#8221;</a> moments.  It fails early and clearly communicates the reason for failure.
  </p>
  
  <p>
    You should be able to test and use your class with erroneous input, bad state, invalid order of operations and any other conceivable error condition and an assert should <strong>never</strong> trip expectedly.  Each assert is checking something should <strong>always</strong> be true regardless of the inputs or computations performed.  If something should always be true, then the number of asserts used shouldn&#8217;t be a barrier to thorough unit testing.  If an assert occurs, the caller knows it&#8217;s likely a bug in the code where the assert is located.
  </p>
  
  <p>
    If you stick to this level of separation, things are a bit easier.
  </p>
</div>