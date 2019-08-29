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
                .rotationEffect(Angle(radians: Double.pi*sliderValue))
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
.rotationEffect(Angle(radians: Double.pi*(abs(offsetValue-0.5)-0.5)))
{% endhighlight %}

![sliderImage]

Still looking fine, right? Lets add there two buttons that will set the offset to minimum and maximum values and wrap it in `withAnimation` block.

{% highlight swift %}
/// ...
			HStack {
                Button(action: {
                    withAnimation(.easeInOut) {
                        self.sliderValue = 0
                    }
                }) {
                    Text("Set to 0").padding()
                }
                Button(action: {
                    withAnimation(.easeInOut) {
                        self.sliderValue = 1
                    }
                }) {
                    Text("Set to 1").padding()
                }
            }
/// ...
{% endhighlight %}

Suddenly we see that the rotation is animated between various offsets but not in the manner we have *designed* and definitively not between edge positions.

![sliderButtons]

This is simply because even though the `sliderOffset` changes from 0 to 1, the rotation does not change - it is 0 on both edges and **animation system seems pointless to interpolate between same values**. 

So what if we really want it to animate just like when the slider is being dragged?

As a potential solution seems to be creation of custom modifier like this and setting the offsetValue as animatable data

{% highlight swift %}
struct CustomRotationModifier: ViewModifier {
    
    var offsetValue: Double // 0...1
    
    var animatableData: Double {
        get { offsetValue }
        set { offsetValue = newValue }
    }
    
    func body(content: Content) -> some View {
        content
            .rotationEffect(Angle(radians: Double.pi*(abs(offsetValue-0.5)-0.5)))
    }
}
{% endhighlight %}

Sadly, soon we realize that this is dead-end

We need somethin else...

## GeometryEffect comes to the scene##

As you definitively guessed, the situation will be different using GeometryEffect. GeometryEffect is another modifier, but it behaves differently while animation interpolates. Its main method `func effectValue(size: CGSize) -> ProjectionTransform` is called **continuously during interpolation** of its `offsetValue` property. 

Lets replicate our functionality with GeometryEffect implementation:

{% highlight swift %}
struct CustomRotationEffect: GeometryEffect {

    var offsetValue: Double // 0...1
    
    var animatableData: Double {
        get { offsetValue }
        set { offsetValue = newValue }
    }
    
    func effectValue(size: CGSize) -> ProjectionTransform {
        let angle = Double.pi*(abs(offsetValue-0.5)-0.5)
        
        let affineTransform = CGAffineTransform(translationX: size.width*0.5, y: size.height*0.5)
        .rotated(by: CGFloat(angle))
        .translatedBy(x: -size.width*0.5, y: -size.height*0.5)
        
        return ProjectionTransform(affineTransform)
    }
}
{% endhighlight %}

The return value of `effectValue` method is a [ProjectionTransform] struct which is basically 3x3 transformation matrix supporting affine transformations (like scale, translation, rotation) that is applied on the view in visual space.

![rotationEffect]

You can see in the example that even though our effect aims on the rotation, we had to introduce 2 translations. That is because the rotation is being done around point *(0,0)* (top left corner) so first we shift center of the view to that point, rotate it and move back to previous position. 

## Better use-case ##

## Real life examples ##


Did you like this article? 

Feel free to comment or criticise so the next one is even better. And share it with other SwiftUI adopters ;)


[GeometryEffectt]: https://developer.apple.com/documentation/swiftui/geometryeffect
[SwiftUI]: https://developer.apple.com/documentation/swiftui
[tutorials]: https://developer.apple.com/tutorials/swiftui/creating-and-combining-views
[rotationEffect]:https://developer.apple.com/documentation/swiftui/view/3278649-rotationeffect
[ProjectionTransform]: https://developer.apple.com/documentation/swiftui/projectiontransform

[sliderImage]: /assets/posts/sliderImage.gif "Moving slider"
[sliderButtons]: /assets/posts/sliderButtons.gif "Controlling position with buttons"
[rotationEffect]: /assets/posts/rotationEffect.gif "GeometryEffect animates view properly"