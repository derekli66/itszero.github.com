---
layout: post
title: "the Solarized Dark Vim"
date: 2012-04-19 22:38
comments: true
categories: vim
---

Today, I spent a whole day on reconfigure my Vim. Vim is a powerful monster
that needs care. Once you feed it with (your precious) time, Vim will pay you
back with a nice and pretty editing experience.

{% img /images/posts/solarized-dark-vim.png 600 "Solarized Dark vim" %}

Two things I'll talk about today: Solarized Dark and Powerline.

## Solarized Dark

[Solarized Dark] is a color palette designed by [Ethan Schoonover]. From my
experiences, this color scheme proves to be usable across many kinds of
computers and monitors. The colors are smooth and clear. Color settings for
most of daily use apps are provided on the official website. Even if it doesn't
provided officially, you usually still can find it somewhere on the Internet.
:-p In short, pretty color for your eyes and large userbase. Go get it now!

To use this color scheme (properly) in Vim, some prerequisites must be met.

  * A decent terminal emulator with properly configured 256-colors support
  * Install vim-colors-solarized plugin

Since there are way too many terminal apps to be introduced one-by-one, I'll
save your time from that. I can tell you that I'm using this color scheme on
[iTerm2], [mintty], [gnome-terminal]. Oh, Did I mention that I just gave your
links to those color scheme files? :-D

P.S. OSX's Terminal is no good, try iTerm2 if you're using OSX.

Once you prepared your terminal environment for colorful life, we can get to
the business. The vim-colors-solarized plugin can be installed by adding this
line to your .vimrc, given that you're using [Vundle], and please do use it.

``` vim
Bundle 'altercation/vim-colors-solarized'

colorscheme solarized
```

Save, reopen your vim, and run `:BundleInstall`. Do reopen your vim again, then
Voilà!

## Powerline

[Powerline] is a plugin designed to power your Vim's **laststatus**. It gives
you a super pretty and useful statusbar right after installation. If you want
to install this plugin, add those to your .vimrc.

``` vim
Bundle 'Lokaltog/vim-powerline'

let g:Powerline_symbols = 'fancy'
```

Set g:Powerline_symbols to **fancy** gives you best look, but it requires you
to use a patched font. You can find those fonts
[here](https://github.com/Lokaltog/vim-powerline/wiki/Patched-fonts).
To use those fonts, simple download it and change your favorite terminal app's
fonts to ``[Your favorite font name here] for Powerline''.

## Conclusions

Your Vim should now be look liked mine in the screenshot. However, this is just
a start. The real power of vim lies in the plugins. Maybe I'll write another
post to talk about my favorite vim plugins sometimes later. Bon voyage, on the
journey of becoming a power Vim user. :-)

[Solarized Dark]: http://ethanschoonover.com/solarized
[Ethan Schoonover]: http://ethanschoonover.com/
[iTerm2]: https://github.com/brantb/solarized/tree/master/iterm2-colors-solarized
[mintty]: https://github.com/mavnn/mintty-colors-solarized
[gnome-terminal]: https://github.com/sigurdga/gnome-terminal-colors-solarized
[Vundle]: https://github.com/gmarik/vundle
[Powerline]: https://github.com/Lokaltog/vim-powerline
