---
layout: post
title: "Show last modified date in Middleman"
date: 2012-03-28 13:39
comments: true
categories: middleman tricks
---

It is very convenient to build a static website using middleman along with some tools like compass, twitter's bootstrap framework. You can easily build a professional looking website while maintaining high code readability and maintainability.

When you're designing a website for a conference or event, one thing that you might want to put in the footer is the "last modified date". The first way come in to my mind to do this is very simple. Just put a line to output current date in ruby code.

``` haml Wrong way to do last modified date
  Some website™
  #{"Last Modified: %s  " % DateTime.now.strftime("%Y/%m/%d")}
```

I was thinking that the middleman will only build changed file. Though this code will show the last `build time', but it'll act effectively as last modified time for file. I was wrong. :( If you build the website in different date, you'll find out hat middleman is smart enough to know that the evaluated result of ruby code in the layout has changed. It will then rebuild every file that includes this layout. However, it is very hard to know which file is being generated in middleman. I tried lots of way and finally find this. When middleman building a website, it simulates a Sinatra request in the following format:

```
http://example.org/your_page.html
```

In sinatra, you can get the request via `request' object in page. This works in middleman too. However, it gives you no clue about the real filename since the engine extension has been stripped in output. Fortunatly, in most condition, one file is going to use only one engine to build, so just append a * operator after the path should be just fine. The result code would look like this:

``` haml Right way to do last modified date
  Some website™
  #{"Last Modified: %s  " % File.mtime(Dir.glob("source/#{request.path}*")[0]).strftime("%m/%d/%Y")}
```

Voila, now you have it. :) If you have a better way to do this, please share with me!