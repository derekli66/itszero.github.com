---
layout: post
title: "MetaWatch meets Google Maps"
date: 2011-11-18 08:42
comments: true
categories: [MetaWatch, Projects, Google Maps]
---

{% img right /images/posts/MetaPlay.jpg 200 MetaPlay %}

MetaWatch just shipped out few days ago. Once I got the watch, I starts to play with it, and to see what it is capable to do. I tried to use the intent-based API and put together this little experiment. I use the phone to acquire current location then retrieve Google Maps image for 96x96 display. Before you could deliver images to the watch, you'll need to convert it to Black & White image. I used a threshold filter on the luminance channel for this, and the photo above is the result. Kinda fun, but not really useful. Let's see what can be done next..

Update: Source code are now available [here](https://github.com/itszero/MetaLoc)!