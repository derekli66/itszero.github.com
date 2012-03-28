---
layout: post
title: "Trying out the impress.js"
date: 2012-01-06 02:25
comments: true
categories: [new technology, impress.js, presentation]
---

There is a interesting javacsript library being released on the github these days, the [impress.js](http://bartaz.github.com/impress.js/). Impress.js is a framework for making presentations on a open canvas, or so called the [Prezi-style](http://prezi.com). I tried out the Prezi few years ago. It's cool, but as the README from the impress.js said: ``impress.js may not help you if you have nothing interesting to say ;)''. I don't really have a chance to make a will-use-on-the-stage presentation on a Prezi.

Fortunately, there is a regular meeting of my lab, and it's just my turn to present a read paper this week. I spent a whole night and messed around with the impress.js. You can find my result here: [A dynamic user authentication scheme for wireless sensor network](/attic/dynamic-ua-for-wsn/). I'm not really get used to arrange contents flat, so the result is not perfect yet.

![A presentation made using impress.js](/images/impressjs_dynamic_ua.png)

It's really fun to make slides using HTML5/CSS3, and make them *alive* through impress.js. For example, I made that Smartcard entirely in the CSS3, which used border-radius and linear-gradient properties. I also used some CSS3 animation property in the Security Analysis section. Once you enter the slide, *Login Replay* becomes larger and the other two fade out. Next slide zooms in on the detail of login replay, the text will rotate for a little bit to give you a visual clue. You can use whatever CSS3 trick you know to make a presentation. Best of it? You can host it on whatever provider you want since it's just some plain HTML/CSS/JS files. Dropbox? Sure. Github? Done.

However, it does have some strange quirk. First, the relative location and zoom-on-active of slides seems not necessarily remain constant across different computer / resolution / browser configuration set. My slides did need some tweak to make sure no text being blocked out on both my iMac 27" and MacBook Air 13". You'll need to manually align your slides to the center, so you can expect some negative values on X, Y values. It would be awesome if the library can take care of this of me. :P

In any case, this is still a fascinating library worth you to play with. Get it now on the github and perhaps you can make your own slides, or better, contribute some patches back to the project.
