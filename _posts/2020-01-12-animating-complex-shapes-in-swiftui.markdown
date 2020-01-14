---
layout: post
title: "Animating complex shapes in SwiftUI"
date: 2020-01-12
author: Pavel Zak
categories: development
tags:	swiftUI shape animation animatableData path charts
cover:  "/assets/posts/06_blueprint.jpg"
---

Hello and welcome to another blog post about SwiftUI. This time, we will talk about the animation of complex shapes in SwiftUI. You will understand animatableData and will be able to implement custom Shape class that depends on multiple parameters. 


## AnimatableData

Animating shapes can be easy thanks to `animatableData` property. We have seen example of such animation in my previous post about [custom controls]. The magic behind animatableData is actually quite simple math. During the animation, the property value is being interpolated from starting value to the ending according to the animation timing curve.

To demonstrate it once again, lets start with a simple rectangle with cout-out rounded corners. 

{% highlight swift %}
struct CoutOutRectangle: Shape {
    var cornerRadius: CGFloat
    
    var animatableData: CGFloat {
        get { cornerRadius }
        set { cornerRadius = newValue }
    }
    
    func path(in rect: CGRect) -> Path {
        return Path { path in
            let width = rect.width
            let height = rect.height
            
            path.move(to: CGPoint(x: cornerRadius, y: 0))
            path.addLine(to: CGPoint(x: width-cornerRadius, y: 0))
            
            path.addQuadCurve(to: CGPoint(x: width, y: cornerRadius), control: CGPoint(x: width-cornerRadius, y: cornerRadius))
            //
            path.addLine(to: CGPoint(x: width, y: height-cornerRadius))
            path.addQuadCurve(to: CGPoint(x: width-cornerRadius, y: height), control: CGPoint(x: width-cornerRadius, y: height-cornerRadius))
            //
            path.addLine(to: CGPoint(x: cornerRadius, y: height))
            path.addQuadCurve(to: CGPoint(x: 0, y: height-cornerRadius), control: CGPoint(x: cornerRadius, y: height-cornerRadius))
            //
            path.addLine(to: CGPoint(x: 0, y: cornerRadius))
            path.addQuadCurve(to: CGPoint(x: cornerRadius, y: 0), control: CGPoint(x: cornerRadius, y: cornerRadius))
        }
    }
}
{% endhighlight %}


Notice that I have created a single property called `cornerRadius` that is being set or read from `animatableData` setter and getter. That is enough for SwiftUI to perform the animation whenever we change the corner radius. 

Lets test it within dumb demo view:

{% highlight swift %}
struct AnimationDemoView: View {
    @State var cornerRadius: CGFloat = 0.0

    var body: some View {
        CoutOutRectangle(cornerRadius: self.cornerRadius)
            .fill(Color.green)
            .onAppear{
                withAnimation (Animation.easeOut(duration: 0.4).repeatForever(autoreverses: true)){
                    self.cornerRadius = 20.0
                }
            }
    }
}

{% endhighlight %}

It works!

![rectangle]

## AnimatablePair

Now, in the case things are more complicated and we need to animate the shape according to two values, we can utilize `AnimatablePair<T>` type. Here the only obstacle is to define the right getter and setter to pass data between our control properties and values stored in AnimatablePair.

As a example, we will build a wedge shape that can be used to compose pie charts. The wedge has two main properties - `angleOffset` and `wedgeWidth` that we both want to be animatable. The best explanation of both properties gives following illustration:

And the resulting code is here. Note how AnimatablePair`s first and second values are being mapped to our properties.

{% highlight swift %}
struct WedgeShape: Shape {
    var angleOffset: Double // in radians
    var wedgeWidth: Double // in radians
    
    public var animatableData: AnimatablePair<Double, Double> {
        get {
           AnimatablePair(Double(angleOffset), Double(wedgeWidth))
        }

        set {
            self.angleOffset = newValue.first
            self.wedgeWidth = newValue.second
        }
    }
    
    func path(in rect: CGRect) -> Path {
        return Path { path in
            let width = Double(rect.width)
            let height = Double(rect.height)
            
            let middlePoint = CGPoint(x: width/2, y: height/2)
            let startingPoint = CGPoint(x: width/2 + cos(self.angleOffset)*width/2, y: height/2 + sin(self.angleOffset)*height/2)
            path.move(to: middlePoint)
            
            path.addLine(to: startingPoint)
            path.addArc(center: middlePoint, radius: rect.width/2, startAngle: Angle(radians: self.angleOffset), endAngle: Angle(radians: self.angleOffset+self.wedgeWidth), clockwise: false, transform: CGAffineTransform.init(scaleX: 1, y: rect.height/rect.width).translatedBy(x: 0, y: (rect.width-rect.height)/2))
            
            path.addLine(to: middlePoint)
        }
    }
}
{% highlight swift %}

Having wedge shape ready, it is now possible to build a ZStack of these views to create an animatable piechart, like this:

![pieChart]

## AnimatableVector

In the case things are more complicated and we need to animate the shape according to two values, we can utilize AnimatablePair<T>

TODO implementation of  nimatable vector
// example of chart shape
// vid

example of shape morphing
// vid

Mention transitions

## The challenge

Now try to play with the AnimationVector by yourself and as a challenge implement morphable shapes like this one below.

Do not hesitate to share your solution or ask for help, I will gladly assist you.


*Did you enjoy this article? Do you have anything to add?*

*Feel free to comment or criticise so the next one is even better. Or share it with other SwiftUI adopters ;)*


[SwiftUI]: https://developer.apple.com/documentation/swiftui
[custom controls]: TODO


[toggle]: /assets/posts/06_toggle.gif "Toggle in action"
[pieChart]: /assets/posts/07_piechart.gif "Animated pie chart composed from several wedges"

[rectangle]: /assets/posts/07_cutoutrectangle.gif "Animated rectangle with cut out corners"


