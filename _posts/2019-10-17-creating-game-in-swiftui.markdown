---
layout: post
title: "Creating a game in SwiftUI - a lookback"
date: 2019-10-17
author: Pavel Zak
categories: development
tags:	swiftUI game experience
cover:  "/assets/posts/animatingGradientCover.jpg"
---

I created a game. In SwiftUI. A madness, you say? Let us have a look at it...


I fully dived in [SwiftUI] two+ months ago and since the best thing to learn something is actually to ship a product I decided to create a game. Why? I was amazed by SwiftUI animation capabilities and it seemed to suit my idea for the game mechanics pretty well.

In this post I would like to look back on those two months and put together little summary of my experience with this new tech and recommendation for those who are thinking about start using SwiftUI as well.

## Flipinity

This post is not intended to promote the game itself, but for the context of what was actually created, check out short trailer. In case you are interested in more details about the game story, I have created a separate blog post to that,

![Flipinity trailer](http://img.youtube.com/vi/glsHiZwmTS0/0.jpg)](https://www.youtube.com/watch?v=glsHiZwmTS0)

## Living in Beta

One of the main factors that affected my experience with SwiftUI was the Beta environment. To be able to develop in SwiftUI I had to adopt beta versions of iOS, MacOS Catalina and XCode. One had to be really careful which versions are installed where since it was quite common that there are incompatibillities. I have never read change-logs so carefully before. 

Also SwiftUI was evolving during whole summer. Things that were working before or was introduced in Apple tutorials often stopped working or became obsolete. One example - beta 3 brought a bug when settig up a fill in shape Path caused app to crash. The only thing that I could do was to discard custom path shapes and wait till Beta 5 that fixed that.

## Good times

Learning SwiftUI was quite time consuming since there was no proper documentation and I had learn by trial and error approach. Even though, I had so much fun. Declarative thinking quickly got under my skin and I was able to produce UI pretty quickly and focus on game mechanics and app business logic. 
Main advantages of using SwiftUI (in my oppinion) are:

** Speed of development - despite having much more experience with UIKit just after two months I am able to prototype and fine tune UI in SwiftUI much faster. Building layout and iterating over it is piece of cake when compared to the Autolayout hell I was familiar with.

** App architecture - SwiftUI quietly forces you to write better app architecture. No more massive view controllers or 3rd party MVVM libs. Thanks to [Combine] it is intuitive to keep business and presentation logic apart. To be honest, this is my first *personal* project that is covered with tests - thanks to SwiftUI.

** Reusable styling - Thanks to ViewModifiers and Style structs it is easy to reuse styling across multiple views. It is also beneficial to define colors in Asset catalog to support Dark mode without any hussle (but that is not limited to SwiftUI only)

** Debugging - SwiftUI enables to run or preview any view from the project almost instantly. In my case, live previews eased the learning phase, but I hardly use them nowadays. Why? See next bullet-point

** Readability - Once familiar with it, you can *see* what view will be produced just by reading its code. This is something I have never experienced before.

** Cross-device support - You can have single codebase spanning across iOS, MAcOS, tvOS and even WatchOS, which is quite impressive.

** Animation system - This was the selling point in my case. Producing eye candies is not just fun but extremely easy. On top of the fact that you can create amazing effects just by change of some view property, there are AnimatableModifier and GeometryEffect at your hand - two almost hidden gems that can unleash your creativity

## Challenges

Most challenges I have faced resulted from the unknown environment and the Beta factors I have already mentioned. But it is fair to mention several challenges or drawbacks that I still percieve:

** Mixing gestures - mixing of gesture recognizers is quite tricky, especially in the combination with ScrollView or Slider. Even though it is possible to set gesture recognizer as `.simultaneousGesture` I had troubles to mix for instance slider within view with set drag gesture.

** Paging Scrollview - There is only basic Scrollview implementation that does not support paging (in the manner we are used to from UIKit). I ended up with custom impementation of PagingScrollview. 

** You cannot throw UIKit away - The set of *standard* views available in SwiftUI is limited when compared to what we used to have in UIKit. Views like UIPageControl, MKMapView have no counter part in SwiftUI yet and you need to wrap them as [UIViewRepresentable].

** Uncertain future  - One thing that I am curious about is how SwiftUI will evolve. I suspect that we can witness future changes that may not be fully backwards compatible - like we saw during various Swift versions.

** Peformance - I need to mention this in the context of my game. In optimizing phase of development, I noticed how several harmless looking animations did have very high energy impact and could turn the device red hot. I did try to experiment with setting the right drawingGroup to pass the rendering to Metal, but did not succeed and ended up with removal of those effects.

## Verdict

So what is my conclusion? Is SwiftUI good tool to write a game with? I would say it suits only limited set of logic and board games. For the rest it would be beneficial to look into one of available game engines.

BUT this whole game experiment brought myself to another closure.

I kinda fell in love with SwiftUI and all my upcoming apps will be certainly written in it!

## Message to potential beginners

In Twitter and recently visited conferences, I have came across several discussions if it is the right time to start with SwiftUI. As you can see from this post I say - YES! Do not be affraid of it. The community is growing fats, several books are were published recently by renowed authors so the chance you will be feeling lost is now almost zero.

If I can advice you some of the resources:

** start with [Apple tutorials] and [WWDC19] videos
** there is a packed visual reference created by Mark Moyens
** if you wish to understand context and background more, I recommend [SwiftUI Kickstart] by Daniel Stahlberg
** awesome articles covering advanced topics can be found at [SwiftUILab], [Majid's blog] and [Azam's blog]



[SwiftUI]: https://developer.apple.com/documentation/swiftui
[AnimatableModifier]: https://swiftui-lab.com/swiftui-animations-part3/
[GeometryEffect]: https://izakpavel.github.io/development/2019/08/29/tweaking-animations-with-GeometryEffect.html
[Apple tutorials]: https://developer.apple.com/tutorials/swiftui/tutorials
[WWDC19]: https://developer.apple.com/videos/all-videos/?q=SwiftUI
[Combine]: https://developer.apple.com/documentation/combine
[PagingScrollview]: https://github.com/izakpavel/SwiftUIPagingScrollView
[Azam's blog]: https://medium.com/@azamsharp
[Majid's blog]: https://mecid.github.io
[SwiftUILab]: https://swiftui-lab.com
[SwiftUI kickstart]: https://gumroad.com/l/swiftuikickstart
[UIViewRepresentable]:https://developer.apple.com/documentation/swiftui/uiviewrepresentable
[blog post]: "TODO"

[plot]: /assets/posts/plot.png "Plotted interpolation curve"
