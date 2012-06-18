---
layout: post
title: "Git Dashboard"
date: 2011-11-18 08:57
comments: true
categories: Git
---

{% img /images/posts/git-dashboard.png %}

I actually made this dashboard few weeks ago, I'm just too lazy to write this
down. /me *flee*

This dashboard is made for collaboration with teams. I was a hackthon-ing with
[bizkit](http://o0o.cse.tw) and [ftt](http://www.facebook.com/itsftt). We set
up the dashboard to pulls new data from the repository automatically and
display in this *pretty format*. It was only intended for fun but it turns out
very useful when merging works from everyone. The tool we used to make it
display on the desktop is called
[NerdTool](http://mutablecode.com/apps/nerdtool), and it was very easy to
setup. You just need to add a new entry with the following command. We have
another repository put under `~/.gitwatch` to prevent the work being interfered
when pulling new changes and to eliminate all local branches.

``` bash Command for git dashboard
date; cd /Users/Zero/.gitwatch/fun-taipei-fork-of-csim-saycheese; git pull >/dev/null 2>&1; git lp2 --all | head -n 34
```

Oh, and for the lp2, it's actually abbrev. of log command with customized
format string. You could add the following entries to your `~/.gitconfig` file.

``` text Add these to ~/.gitconfig
[alias]
  lp2 = log --abbrev-commit --decorate --graph --color --pretty="format:%C(yellow)%h%Cred%d\\ %C(cyan)%an%Creset:\\ %Cgreen%s%Creset\\ %ar" --all
```

Now, have fun! :)