---
title: "CondParse, a condition string parser in PHP."
layout: post
date: 2016-03-23 22:10
tag:
- php
- parser
- lexer
- bre
headerImage: false
projects: true
hidden: true # don't count this post in blog pagination
description: "CondParse, a condition string parser in PHP."
jemoji: '<img class="emoji" title=":interrobang:" alt=":interrobang:" src="https://assets-cdn.github.com/images/icons/emoji/unicode/2049.png" height="20" width="20" align="absmiddle">'
author: ilias
externalLink: false
---

[CondParse](https://github.com/zwilias/condparse) is a "condition string" parser, written in PHP.

[![Build Status](https://travis-ci.org/zwilias/condparse.svg?branch=master)](https://travis-ci.org/zwilias/condparse)
[![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/zwilias/condparse/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/zwilias/condparse/?branch=master)
[![Code Coverage](https://scrutinizer-ci.com/g/zwilias/condparse/badges/coverage.png?b=master)](https://scrutinizer-ci.com/g/zwilias/condparse/?branch=master)
[![Latest Stable Version](https://poser.pugx.org/zwilias/cond-parse/v/stable)](https://packagist.org/packages/zwilias/cond-parse)
[![Total Downloads](https://poser.pugx.org/zwilias/cond-parse/downloads)](https://packagist.org/packages/zwilias/cond-parse)

---

A piece of legacy software our team was using, contained a rather botched attempt at a business rules engine. This engine has been permeated so deeply into the whole system, it's become barely possible to refactor it, let alone replace it entirely. So, accepting that technical debt, the least we could do was attempt to clean it up.

A "condition string", as the engine refers to its business rule descriptors, would look something like this: `#1# && #2#`. This condition would evaluate to `true` if and only if both the object referred to by `id:1` and `id:2` would also evaluate to `true` when called. This makes a fair amount of sense.

However, the approach it took to evaluating a condition string was to quite simply transform the string into something akin to `return Condition::evaluate("1") && Condition::evaluate("2");` and throwing that string into `eval`. While that did work fine, it made debugging, optimization, opcode-caching and so on a bit of a nightmare. In other words, the usual problems that tend to arise when using `eval` in production code.

So, I attempted to create a replacement system. The result of which has become a rather performant, rather extensible, and fairly comprehensible condition-string parser.

---

First, it takes your input string - for example `#1# && #2#` - and runs it through the lexer which just turns it into a stream of tokens. This tokenstream is provided to the parser, together with a "TokenMap" (which basically tells the parser which token should be turned into which Operand), who ends up returning a single, executable Operand.

This operand is, in so many words, just an infix representation of the input. In fact, the builtin operands have a toString method which would just return an infix representation. The example input would, hence, be rendered into something representing `AND(#1#, #2#)`.

---

Find the code [on GitHub](https://github.com/zwilias/condparse). Not actively maintained, but a fun experiment nevertheless.
