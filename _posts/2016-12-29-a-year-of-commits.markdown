---
title: "A year of commits - a visual rundown"
layout: post
date: 2016-12-29 18:34
tag:
- git
- os x
- brew
- hook
headerImage: false
blog: true
description: "After automatically taking snapshots on every commit for close to a year, here's the result"
author: ilias
externalLink: false
---

<iframe src="https://player.vimeo.com/video/197416047?color=fff" width="810" height="456" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>

## Wait what?

Somewhere around april, I had a couple of minutes and decided that it would be cool to take a snapshot of myself, every time I `git commit` anything. So, using a couple of lines of ruby I found [somewhere in a PR](https://github.com/rharder/imagesnap/pull/10/commits/23f6feaf2ec2b17ae0ac96d7493dbc96113e9123?short_path=04c6e90#diff-04c6e90faac2675aa89e2176d2eec7d8) I created a git hook to do exactly that.

In order to make sure I'd be able to use this on all my repositories without having to add it manually each and every time (which I'd surely forget), I setup a [template directory](https://git-scm.com/docs/git-init#_template_directory) and ensured the hook was in there and marked as a post-commit hook.

## Show me the goods

First, ensure you've installed `imagesnap`. Assuming you're using homebrew like the rest of us:

```bash
$ brew install imagesnap
```

Now, let's get you set up with a git template directory:

```bash
# Create the directory structure
mkdir -p ~/.git-template/hooks

# Create a post-commit file in that directory and feed it the commit-hook:
cat > ~/.git-template/hooks/post-commit << EOF
#!/usr/bin/env ruby
file="~/.gitshots/#{Time.now.to_i}.jpg"
unless File.directory?(File.expand_path("../../rebase-merge", __FILE__))
  puts "Taking capture into #{file}!"
  system "imagesnap -q -w 3 #{file} &"
end
exit 0
EOF

# Make it executable
chmod o+x ~/.git-template/hooks/post-commit

# Finally, "activate" your init dir
git config --global init.templatedir '~/.git_template'
```

Want to just... Execute all of the above, all lazy-like?

```bash
$ curl https://git.io/vMIGe | bash
```

Yeah, I know. Piping curl through bash is generally a very bad idea. [More here](https://www.seancassidy.me/dont-pipe-to-your-shell.html)
{:.note}

*Et voilà*, we have a working git post-commit hook. Now, let's see all your faces in just about a year.
