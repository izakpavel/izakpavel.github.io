---
layout: post
title: "Tweaking animations with GeometryEffect"
date: 2019-08-29
author: Pavel Zak
categories: development
tags:	swiftUI geometryEffect animation
cover:  "/assets/header_image.jpg"
---

In this article I will talk about [GeometryEffect] and some techniques how this modifier can be used to spice up  animations in your [SwiftUI] apps. If you are new to SwiftUI, i recommend you to go through set of [tutorials] provided by Apple that serve as a great kickstart reference. 

At first glance [GeometryEffect] is just another modifier as others, like `.rotationEffect` `.offset` or `.scaleEffect` but even though it can be used in the same manners there is one important difference of how animation system handles them

Lets build a sample project to demonstrate it.

*Since the SwiftUI is still in beta all code provided below was working in XCode 11 beta 6*

## Setup ##

Lets create simple View with a image and slider. And lets assume we want the image to be rotated based on slider position. This is easy job to do and can be done like this using [rotationEffect].


{% highlight swift %}
struct ContentView: View {
    @State var sliderValue : Double = 0
    
    var body: some View {
        VStack {
            Spacer()
            Image(systemName: "smiley")
                .font(.title)
                .padding()
                .rotationEffect(Angle(degrees: 360*sliderValue))
            Slider(value: $sliderValue, in: 0...1, step: 0.01)
            Spacer()
        }
        .padding()
    }
}
{% endhighlight %}

When moving the slider, it works just like we expected, the image is rotated around. 

Now, lets change how the image is rotated and make it rotated only when the slider is in middle of the range.

We will adjust computation of rotation angle like this 
{% highlight swift %}
.rotationEffect(Angle(degrees: 180*(abs(sliderValue-0.5)-0.5)))
{% endhighlight %}

![sliderImage]

Still looking fine, right? Lets add there two buttons that will set the offset to minimum and maximum values and wrap it in `withAnimation` block. Suddenly we see that there the behaviour

{% highlight swift %}

{% endhighlight %}


[GeometryEffectt]: https://developer.apple.com/documentation/swiftui/geometryeffect
[SwiftUI]: https://developer.apple.com/documentation/swiftui
[tutorials]: https://developer.apple.com/tutorials/swiftui/creating-and-combining-views
[rotationEffect]:https://developer.apple.com/documentation/swiftui/view/3278649-rotationeffect

[sliderImage]: /assets/posts/sliderImage.gif "Moving slider"