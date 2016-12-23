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
{:.note}

This means, however, that -- in order to get a pretty test tree -- your IDE has to either:

- **parse the custom CLI output** and somewhat magically convert it to something it can render in a pretty tree
- **wait** for the run to finish, take the XML *report* and interpret it
- do **deep, custom integration** with your test framework so it can keep informed of progress

Not only that, but this means that, if you upgrade your testing framework, you may lose the ability to run tests in your IDE directly, or have some corner-case situation where half your test results go missing.

*Note*: In all of the above, I'm using "your IDE" as an example. The main point I'm making is that we're dealing with a huge degree of coupling between a test framework and its output format.
{:.note}

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

### TAP does not have a clear diagnostics format

> Currently (2007/03/17) the format of the data structure represented by a YAML block has not been standardized. It is likely that whatever schema emerges will be able to capture the kind of forensic information about a test’s execution seen in the example above.

Almost 10 years later, a standard schema has not yet emerged. So yeah.

---

In conclusion, TAP is rather broken. Many sad.

{% include image.html
    img="![broken tap](/assets/images/posts/2016-12-09-unit-test-output-formats-state-of-affairs/broken-tap.jpg)"
    caption="A broken tap. I've seen worse. Courtesy of [yourrepair](http://www.yourrepair.co.uk/broken-tap-what-to-do-before-emergency-plumber-arrives/)"
%}

## Exit TAP

But can we fix it? I think TAP as a format is not the way to go. Rather, a *new and improved* **generic test output protocol** (or **GTOP** for short) would be required.

Let's start with a number of use cases that should, ultimately, be supported when using this *GTOP*.

<div markdown="1" class="note">
A note on wording.

- **GTOP message**: A message, formatted according to the imaginary GTOP spec.
- **GTOP stream**: A line-separated list of GTOP messages, with new messages appearing as they are created.
- **GTOP producer**: A program capable of generating a GTOP stream. Usually, a test-runner or pluggable reporter for a test-runner.
- **GTOP consumer**: A program capable of receiving a GTOP stream and handling its messages. This will usually entail visualizing the results described by the GTOP stream.
- **GTOP parser**: A library or module that is leveraged by a GTOP consumer to convert the messages on a GTOP stream into objects or events or whatever you have that can be handled by the consumer.
</div>

**UC1**: As an end user, I can run my unit tests, written using a test-runner capable of generating GTOP output, and visualize the results in my IDE, my CI environment, or -- by piping the output into a command line GTOP consumer -- on the command line.
{:.jumpout}

**UC2**: As GTOP parser, I am perfectly content handling the output on a line by line basis; and do not need to keep track of previous output in order to verify the validity of the received GTOP output. Ideally, I can use widely available tooling to parse and verify GTOP output, without having to write custom parser logic or keeping it to the bare minimum.
{:.jumpout}

**UC3**: As a GTOP consumer, I am able to deterministically create and update my tree, using the GTOP messages provided to me by the parser. Even though these messages may spawn from different threads or even different processes altogether, this does not influence my ability to keep track of the state of my tree.
{:.jumpout}

**UC4**: As an end user, looking at the visualization of my test results; I should be able to locate the definition of the test file, if supported by the used GOP consumer.
{:.jumpout}

**UC5**: As an end user, I can easily identify *why* my test failed, if supported by the used GOP producer and consumer.
{:.jumpout}

**UC6**: As an end user, I can see that PHPUnit reported risky tests and eslint gave a warning.
{:.jumpout}

Now let's extract some requirements. Having requirements will help us in creating a format, by checking that all requirements can be fulfilled.

- **Must** haves:

  - **R1.** Use a simple, yet **extensible format**. [Covers: *UC2*]
  - **R2.** Create a simple, **machine parsable specification** for this format. [Covers: *UC2*]
  - **R3.** Allow **nesting** structures. [Covers: *UC3*]
  - **R4.** Provide a way to **declare** tests and test suites. [Covers: *UC3*]
  - **R5.** **Unordered** messages should be possible. [Covers: *UC3*]
  - **R6.** Allow multiple types of test failures and passes. [Covers: *UC6*]

- **Nice** to haves:

  - **R6.** Unified way of **encoding** the **location** of tests. [Covers: *UC4*]
  - **R7.** Codify how to provide **encoded diagnostics** (expected vs actual for assertions, failure message, stack traces, whatever you have). [Covers: *UC5*]
  - **R8.** Be open for extension, closed for modification. In other words, **best effort forward compatibility**.

**[UC1]** is covered by the separation of concerns between producer and consumer. **[R8]** is to ensure that the format does not prevent other use cases from arising and other requirements to be created and handled.

---

## Enter GTOP

GTOP, short for Generic Test Output Protocol, is a proposed protocol to deal with the above requirements. In short, each GTOP message would be a [JSON](http://www.json.org/) object, with each message appearing on its own line. Since JSON is supported in a multitude of languages, this would make it extremely easy to generate the messages using a simple interface.

Using [http://json-schema.org/](JSON schema), we can ensure that there's a proper specification for the messages, which developers of GTOP producers can use to validate their output against, and developers of GTOP consumers can use to validate incoming messages.

Other than simply having the protocol, guidelines should also be created (and maintained!), helping the developer community maintain consistent results across the board.

### Message format

Let's start with the basics. What should a GTOP message look like? In order to somewhat help parsers, I'll suppose its first level should quite simply show the type of message. Now, keeping in mind the GTOP is not supposed to be a format for saving reports, but rather reporting running tests, let's go wild and say that each message should only contain a single entry.

So, let's start with the simplest message I can come up with - 1 single test, no grouping whatsoever.

```json
{
    "testResult": {
        "name": "My test",
        "status": "PASSED"
    }
}
```

Note that the whitespace I've included here, means that this is not actually a valid GTOP message. A GTOP message should not include line breaks, since the GTOP stream itself is already supposed to be line delimited!
{:.note}

Naturally, that's not going to cover a whole lot of cases. So let's add some more.


```json
{
    "testStarted": {
        "name": "My test suite",
        "id": "8f22cce8-c281-456e-a2ef-54a863a0ccdd"
    }
}

{
    "testStarted": {
        "name": "My test",
        "parentId": "8f22cce8-c281-456e-a2ef-54a863a0ccdd",
        "id": "d0a2ea3d-a877-40ba-bc60-eff79545d77e"
    }
}

{
    "testResult": {
        "id": "d0a2ea3d-a877-40ba-bc60-eff79545d77e",
        "status": "PASSED",
        "duration": 120
    }
}

{
    "testResult": {
        "id": "8f22cce8-c281-456e-a2ef-54a863a0ccdd",
        "duration": 123,
        "status": "PASSED"
    }
}
```

Again, this is not a valid GTOP stream due to the inclusion of line breaks within the GTOP messages. Just formatting for human readability.

So, using just 2 message types -- `testStarted` and `testResult` -- we've created a tiny little tree with one branch and one leaf node. We've indicated relations between the branch and its leaves, and we've indicated their status after running, together with how long they've run for. That's a "yay!" as far as I'm concerned.

For more information about the GTOP protocol, I'll refer you to its official minisite -- which is literally empty at the time of writing: [gtop.ilias.xyz](https://gtop.ilias.xyz). Keep an eye on it for future updates.

### Creating adoption

One additional problem with TAP that we haven't really talked about, is its reluctancy to gain much traction. Obviously it's popular with the Perl guys, as its the default format for many of its tooling (Test.pm, Test::Simple, Test::More, etc, etc). Although there's *quite a few* modules for different languages for both producing and consuming TAP, still many developers have never heard of TAP, nor do many (consciously) use it.

Obviously, introducing a new output format for tests is not an easy task.

Furthermore, documentation and specification. In order to properly collect documentation and maintain a properly (versioned) specification, I've created a [github organization](https://github.com/generic-test-output-protocol/). The documentation and specification will, for now, appear on [this little website](https://gtop.ilias.xyz/) Feel free to request access if you feel like you have anything to contribute, and we'll talk it over.

I also plan on putting together a couple of supporting projects:

- a PHPUnit TestListener
- a custom karma reporter
- a custom ESLint formatter
- a JetBrains plugin

Perhaps I'll also make some conversion filters -- TAP to GTOP and GTOP to TAP. Finally, a commandline GTOP reporter could be cool.
