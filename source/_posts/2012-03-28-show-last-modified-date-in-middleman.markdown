---
layout: post
title: "Show last modified date in Middleman"
date: 2012-03-28 13:39
comments: true
categories: middleman, tricks
---

One thing that you might want to put in the footer is the "last modified date". The first way come in to my mind to do this is very simple. Just put a line to output current date in ruby code.

``` ruby Wrong way to do last modified date
  Some website™
  #{"Last Modified: %s  " % DateTime.now.strftime("%Y/%m/%d")}
```

            ISLPED'12 Conference  
            Contact [webmaster](phchou@uci.edu)  
            #{"Last Modified: %s  " % File.mtime(Dir.glob("source/#{request.path}*")[0]).strftime("%m/%d/%Y")}
            Banner photo credit goes to [gromgull](http://www.flickr.com/photos/gromgull/2476659336/)

