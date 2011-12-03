---
layout: post
title: "Choose your weapons"
date: 2011-12-03 13:33
comments: true
categories:
- cli
- zsh
- tmux
- vim
- solarized
- iterm2
- homebrew
- dotfiles
---

After mere days of living life the CLI way, my favorite set of tools has changed
considerably. A week ago, my tools of choice were:

- Macports for package management
- Bash as a shell, running in Terminal.app
- Simple tabs in the terminal for handling multiple sessions, or GNU Screen if I
  was feeling fancy
- nano for editing files

Additionally, I never bothered with making any of these tools work or look the
way I wanted them to work and look. If I ever even bothered configuring anything
(you know, making some bash aliases or whatever), all of that configuration was
just dumped into the appropriate config file, without any order to it, without
any comments, without any structure.

This has changed.

### Homebrew ###

Macports works fine, but homebrew works great. It tries to use as much of the
apple-supplied libs as possible, while offering the option to install your own
python/perl/whatever. What really sets it apart is that it's built on Git.
Change any of the so-called Formulae (using `brew edit`) and upstream changes
are just merged in. It is fairly awesome. Creating new Formulae? `brew create`.
Did I mention how these Formulae are exceedingly simple (but flexible,
extensible, and capable of complex things) ruby classes?

``` ruby Example Formula http://mxcl.github.com/homebrew/ The Homebrew homepage
require 'formula'

class Wget < Formula
  homepage 'http://www.gnu.org/wget/'
  url 'http://ftp.gnu.org/wget-1.12.tar.gz'
  md5 '308a5476fc096a8a525d07279a6f6aa3'

  def install
    system "./configure --prefix=#{prefix}"
    system 'make install'
  end
end
```

Seriously, how simple could it be? Definitely worth checking out.

### Zsh ###

Zsh is alternative shell, in many ways very similar to bash, yet decidedly
different. For one, it's goal seems to be something along the lines of
"implement everything every other shell can do", meaning it's extremely
full-featured, yet, very, *very* lean. There's another reason I switched to zsh,
though: [holman's dotfiles](https://github.com/holman/dotfiles). More on that
later.

### iTerm2 ###

iTerm2 is a *free* replacement for Terminal.app, and the successor to iTerm. It
lets you get work done, and it lets you choose how. It's extremely configurable
and in my experience, does all this with less memory than Terminal.app could
manage. For me, though, its killer feature is a simple one: fullscreen mode,
circumventing Lion's fullscreen thing. Lion's fullscreen apps blank out all
monitors except for your primary monitor. I understand the reasoning (fullscreen
= distraction-free = remove all possible distractions), but I don't want it. Say
I'm doing web-development. I'd have a text-editor (vim, nowadays) open in my
terminal, in fullscreen mode, and a browser on my second monitor. A simple
use-case that justifies having the option to allow me to use both my monitors.
Even in fullscreen, "distraction free" mode.

### tmux ###

Tmux is a modern, BSD replacement for screen. GNU Screen, that is. There are
many problems with GNU Screen. First and foremost, it is pretty much dead. Even
if it weren't, though, it is buggy, the code is unstructured, unmaintainable and
really just discourages improvements. It's configuration syntax is notorously
complicated and obtuse, too. And perhaps worst of all, it doesn't even support
vertical splitting by default. There is a patch, but why would a v4.something
piece of software need an external patch to provide such basic functionality? I
mean, it's a terminal multiplexer, that's really the least it should be able to
do. Tmux is, in all of the above regards, better. It provides a clean,
well-documented client-server model, a simple configuration syntax, it's
relatively bugfree (for as far as I've noticed) and best of all, super-low
memory usage. Screen goes up to about 50 MB with just a few windows open
*easily*, while tmux hasn't even hit 10 MB here. Wonderful!

### vim ###

I'll leave the explaining of why vim rocks to Ryan Smith and Allen Riddle who
have created a [wonderful](http://vimeo.com/6246476 "Why Vim Rocks (One editor to rule them all) - Part 1") 
[three-part](http://vimeo.com/6246492 "Why Vim Rocks (One editor to rule them all) - Part 2") 
[video series](http://vimeo.com/6255868 "Why Vim Rocks (One editor to rule them all) - Part 3")
on Why Vim Rocks.

### Solarized ###

Imagine a color palette, designed for use in both terminal and gui applications,
that's easy on the eye (sufficiently low contrast), yet perfect for syntax
highlighting readability (due to contrasting hues based on exact colorwheel
relations), that comes in both a light abd a dark variety, can be mixed down to
a variety of 5 color palettes that aren't overwhelming yet have a strong
personality, and is available for a shitload of different tools. Excited? Well,
rejoice, it exists. Introducing, Ethan Schoonover's
[solarized](http://ethanschoonover.com/solarized). I use it as the theme for
iTerm2, my tmux config uses the solarized colors, I've the vim theme installed,
and even mutt, my mailclient, is solarized.

### dotfiles ###

I'm just going to link to holman's blogpost about why 
[his dotfiles project is awesome](http://zachholman.com/2010/08/dotfiles-are-meant-to-be-forked/ "Zach Holman says: Dotfiles are meant to be forked")
. And then point you to 
[my fork of his dotfiles](https://github.com/zwilias/dotfiles), including my tmux 
configuration, as well as solarized palettes for both vim and tmux.

... Ok, enough.
