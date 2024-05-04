---
title: Exporting colorized content from the terminal
slug: exporting-colorized-content-from-the-terminal
tags: terminal,shell,html,color
domain: til.jacklinke.com
---

## Background

I have recently been working on significanty enhancing the test suite for [my primary django application](https://www.watervize.com) with some of [Adam Johnson's](https://adamj.eu/) recommendations in his excellent book [Speed Up Your Django Tests](https://adamchainz.gumroad.com/l/suydt), which I had skimmed before, but am finally digging into.

I currently have my tests outputting at full verbosity so I can see detailed output, and I wanted to maintain a simple log of this output. The colorized output of pytest (and many other tools) is very helpful for visually keeping track of what you're reading, and I would lose that benefit if outputing plain text. So I started digging around online for recommendations.

I found lots of suggestions for rather complicated custom scripts, weird terminal workarounds, and hacks, but when I came across a recommendation for ansi2html it seemed like just the right tool for the job - straightforward and effective.

## The Tools

There is a python version of [ansi2html](https://pypi.org/project/ansi2html/) that can be installed via pip and used in Python to convert text with ansi color codes to html files or in the shell to capture output and convert the associated ansi color codes to html.

This functionality is also available as a tool (combined with a couple other complimentary tools) via your system's package management system as `colorized-logs` *([Debian](https://packages.debian.org/source/sid/colorized-logs),[Ubuntu](https://packages.ubuntu.com/search?keywords=colorized-logs),[Gentoo](https://packages.gentoo.org/packages/sys-apps/colorized-logs), etc)*. This version, written in C, can be found at Adam Borowski's [colorized-logs](https://github.com/kilobyte/colorized-logs) GitHub Repo, which also contains the additional tools.

I went with the latter, though if I ever find myself working with ansi color in Python, I'd definitely try out the python package.

Note: I also found [this shell script](https://raw.githubusercontent.com/pixelb/scripts/master/scripts/ansi2html.sh) which has similar aims, and might work better for some folks.

## Use

For use in the shell, all three variants use the same basic syntax:

```shell
ls -l --color=always | ansi2html > ls.html  # for the python and C-based versions
ls -l --color=always | ansi2html.sh > ls.html  # for the shell-script version
```

Or in my use case:

```shell
docker compose -f docker-compose.local.yml exec django pytest --cov=apps --color=yes | ansi2html > pyest_run_20240504_1450.html
```
