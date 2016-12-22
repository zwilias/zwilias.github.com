---
title: "Unit-test output formats - a state of affairs"
layout: post
date: 2016-12-09 21:02
tag:
- unit testing
- tap
- test anything protocol
- generic test output protocol
- junit
headerImage: false
blog: true
description: "Unit-test output formats - a state of affairs"
author: ilias
externalLink: false
---

We're living in modern times. We have an incredible wealth of tooling for our development needs, many of it "language agnostic". We have IDE's capable of managing source files in many different languages, build-tools that may compile and run code written in many different languages, and so much interoperability it'll make your head float. Many of our test-runners have pluggable assertion frameworks, and our IDE's are fairly capable of running our tests and rendering a nice test-tree with pretty results.

{% include image.html
    img="![Pretty test tree](/assets/images/posts/2016-12-09-unit-test-output-formats-state-of-affairs/v15_tests_vertical_layout.png)"
    caption="A pretty test-tree! Source: [JetBrains](https://blog.jetbrains.com/idea/2015/07/intellij-idea-15-eap-introduces-new-ui-for-testing/)"
%}

What's odd, however, is the degree of coupling in our test-runners. More specifically, the degree of coupling between the runner itself, and the output format. Running most test-runners on the commandline, will output some pretty, human readable format, and -- after completion -- optionally dump an JUnit XML report somewhere, so your CI environment can pick it up and keep track of your test results.

As an aside, I should probably mention that the JUnit XML format isn't properly specified anywhere. There have been some attempts at creating an XSD for it, but the only real source of truth is the code of the JUnit ANT task. [Read on](http://help.catchsoftware.com/display/ET/JUnit+Format) for more information.
{:.aside}

This means, however, that -- in order to get a pretty test tree -- your IDE has to either:

- **parse the custom CLI output** and somewhat magically convert it to something it can render in a pretty tree
- **wait** for the run to finish, take the XML *report* and interpret it
- do **deep, custom integration** with your test framework so it can keep informed of progress

Not only that, but this means that, if you upgrade your testing framework, you may lose the ability to run tests in your IDE directly, or have some corner-case situation where half your test results go missing.

*Note*: In all of the above, I'm using "your IDE" as an example. The main point I'm making is that we're dealing with a huge degree of coupling between a test framework and its output format.
{:.aside}

So the closest thing we have to a universal test output format is the JUnit XML format, which is an excellent report format, but not a streaming output format.

As a developer, this smells of *tight coupling* and a violation of the *separation of concerns* design principle.

---

## Enter TAP.

TAP -- the [Test Anything Protocol](https://testanything.org/) -- is a sort-of human-readable, plaintext output format which introduces this separation of concerns in the simplest way possible:

> TAP Producer (i.e. your test harness) produces TAP.  
> TAP Consumer (i.e. your test result reporter) consumer TAP.

It started life as part of the test harness for Perl but now has implementations in C, C++, Python, PHP, Perl, Java, JavaScript, and others. And it's great. Got a TAP producer? Install a random TAP consumer, and get your test-results in nyan-cat style.

{% include image.html
    img="![Nyan cat. Yay!](/assets/images/posts/2016-12-09-unit-test-output-formats-state-of-affairs/nyan.png)"
    caption="Nyan cat. Nyan cat. Cat cat cat. Cat! Source: [calvinmetcalf/tap-nyan](https://github.com/calvinmetcalf/tap-nyan)"
%}

However, for all of its nice additions to the unit testing world, TAP ain't perfect. It has a specification, but most producers take some creative liberty. Furthermore, although it looks as if it's a line-based format; its inclusion of inline YAML documents means that any parser needs to buffer and take care of state. Furthermore, there are some practical shortcomings.

### TAP doesn't allow declaring tests

Trying to generate any kind of test tree before actually reporting on their success or failure is, as such, impossible. Out of the box, TAP supports 2 statuses and has 2 directives that change the meaning of such a test:

- `ok` means.. Well, the test has passed.
- `not ok` signifies failure. Obviously.
- `# SKIP` can be applied either to a test or a plan; meaning the tests' status should not be recorded.
- `# TODO` can be applied to a `not ok` test and signifies that it's currently failing and still needs implementation.

Lacking a way to declare a test means it is impossible to create a tree before starting your tests or tracking runtime. Furthermore, de default Perl test-harness, when using subtests, will (by necessity) output the results of the subtests before output the result of the parent test. Naively, one could just link subtest results to the first test that follows. But even that isn't sufficient, as subtests of different parent tests may arrive in an out of order fashion.

### TAP isn't a line based protocol

Although TAP is a test-based protocol, it is no longer a line-based protocol since the introduction of inline YAML documents. By its very nature, a YAML document requires buffering the entire document before parsing it and attaching that result as a diagnostic to whichever test preceded it. This means a lot of context tracking. Worse, this means a TAP parser cannot report a test result before the next test result arrives, which serves as a trigger that no diagnostics are attached.

Besides, as subtest results may be reported before the parent test is reported, a TAP parser needs to keep track of its preceding context.

### TAP is kind of awkward

Highly subjective but I feel that this merits mentioning either way.

It was meant to be human readable, but it's not very clear at all. Especially when subtests and YAMLish diagnostics are used, parsing TAP output - as a human - is, well, awkward. At some point, someone agreed and tried to introduce an alternative syntax: [UTO](https://github.com/universal-test-output/uto-specification). However, that failed to gain much (if any) traction.

### TAP suffers from keeping backwards compatibility

This is a rather sticky subject with many, widely differing opinions, but it needs to be mentioned either way. Introducing better syntax for subtests, for example, is very hard to do without breaking implementations that can only parse output in an older version of the spec. Of course, this is kind of to be expected; but as outlined in [the TAP Philosophy](https://testanything.org/philosophy.html):

> TAP should be forward compatible.

As their format is a rather non-extensible one, there are limited possibilities for introducing new syntax:

- pragmas - which were introduced under the radar, and are intended to influence how the TAP stream is parsed
- directives - special comments, basically
- break older parsers

### TAP doesn't really support structuring tests

That is to say - you may structure your tests whichever which way you like, but the output is currently limited to a single level or -- when using a parser compatible with subtests, which isn't officially part of the specification -- two levels at most. In today's day and age where projects often have a couple of thousand tests, this is a rather limiting factor.

### TAP requires its tests to output in order

Test output in TAP can (optionally) contain a testnumber. When this testnumber is present, the numbers need to be strictly incrementing. Basically, this is a case of "abusing your identifier". It severely hampers running tests in parallel, too, as you can't just interleave the output of many threads.

---

In conclusion, TAP is rather broken. Many sad.

{% include image.html
    img="![broken tap](/assets/images/posts/2016-12-09-unit-test-output-formats-state-of-affairs/broken-tap.jpg)"
    caption="A broken tap. I've seen worse. Courtesy of [yourrepair](http://www.yourrepair.co.uk/broken-tap-what-to-do-before-emergency-plumber-arrives/)"
%}

But can we fix it? I think TAP as a format is not the way to go. Rather, a *new and improved* **generic test output protocol** (or **GTOP** for short) would be required.

Let's start with a number of use cases that should, ultimately, be supported when using this *GTOP*.

Some first use case.
{:.aside}

Some other use case.
{:.aside}

Yet another use case.
{:.aside}

Now let's extract the requirements from this. Having a list of requirements to work with is always nice..

- **Must** haves:

  - **R1.** Use a simple, yet **extensible format**
  - **R2.** Create a simple, **machine parsable specification** for this format
  - **R3.** Allow **nesting** structures
  - **R4.** Provide a way to **declare** tests and test suites
  - **R5.** **Unordered** messages should be possible

- **Nice** to haves:

  - **R6.** Unified way of **encoding** the **location** of tests
  - **R7.** Codify how to provide **encoded diagnostics** (expected vs actual for assertions, failure message, stack traces, whatever you have)
  - **R8.** Be open for extension, closed for modification. In other words, **best effort forward compatibility**.

Requirements:

[] do I have checkboxes? -> nope.

- simple, well known format.
- human readability is ok
- support for declaring tests -> build (partial) tree before finishing it
- unordered messages to support concurrency

Proposal:

- ...
