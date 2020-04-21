---
layout: post
title: "How to define custom LLDB command in Xcode"
date: 2020-04-20
author: Pavel Zak
categories: development
tags:	Xcode LLDB command Swift
cover:  "/assets/posts/08_cover.jpg"
---

It has been a couple of months since my last post. I have been quite occupied with a big project where I am helping to modernize its archaic code-base.

On a daily basis, I spend a lot of time debugging and I am usually using several LLDB commands over and over again. For this purpose, I have defined a couple of custom LLDB commands like printing out the contents of `Data` as `String` - so instead of `po String(data: jsonData, encoding: .utf8)` I simply write `printData jsonData`

Since not many devs know about the opportunity to define own custom LLDB aliases or commands, I have put together a short video:

<center>
<iframe width="560" height="315" src="https://www.youtube.com/embed/h9ggWxh8Evs" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</center>

(Fun fact: The video was made with Keynote which I found quite useful for such task)

The example of `.lldbinit` file can be seen on my [gist].

Stay safe!


[gist]: https://gist.github.com/izakpavel/ceefe1f18bff4e69bb7432af7b65960a "gist with a command example"

