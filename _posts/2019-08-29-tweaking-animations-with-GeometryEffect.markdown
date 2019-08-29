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

You can see in the example that even though our effect aims on the rotation, we had to introduce 2 translations. That is because the rotation is being done around point *(0,0)* (top-left corner) so first we shift center of the view to that point, rotate it and move back to previous position. 

## Better use-cases ##

OK, but what is all of this good for?

I am using geometry effect as a addition to standard modifiers mainly in these cases:
* to introduce event-based forth and back animation 
* there is need of another transformation, but only during animation

I will share examples:

### Button triggered animation###

This is actually the case how I got on the track of GeometryEffect. I wanted the button to perform scale operation when tapped. As you probably know, there is [ButtonStyle] protocol to alter appearance of the button that is usable in such cases (and it animates) but I wanted the animation to happen **after** the tap not depending on the tap duration.

And if you want to create eye candy animation that is applicable not just on buttons, this is the easy way.

So what about a like button?

![likeButton]

{% highlight swift %}
struct LikeEffect: GeometryEffect {

    var offsetValue: Double // 0...1
    
    var animatableData: Double {
        get { offsetValue }
        set { offsetValue = newValue }
    }
    
    func effectValue(size: CGSize) -> ProjectionTransform {
        let reducedValue = offsetValue - floor(offsetValue)
        let value = 1.0-(cos(2*reducedValue*Double.pi)+1)/2

        let angle  = CGFloat(Double.pi*value*0.3)
        let translation   = CGFloat(20*value)
        let scaleFactor  = CGFloat(1+1*value)
        
        
        let affineTransform = CGAffineTransform(translationX: size.width*0.5, y: size.height*0.5)
        .rotated(by: CGFloat(angle))
        .translatedBy(x: -size.width*0.5+translation, y: -size.height*0.5-translation)
        .scaledBy(x: scaleFactor, y: scaleFactor)
        
        return ProjectionTransform(affineTransform)
    }
}

struct LikeButtonView: View {
    @State var likes : Double = 0
    
    var body: some View {
        HStack {
            Text("likes: \(Int(likes))")
                .frame(width: 150, alignment: .leading)
                .padding()
            Button(action:{
                withAnimation(.spring()) {
                    self.likes += 1
                }
            }) {
                HStack {
                    Text("like more")
                    Image(systemName: "hand.thumbsup")
                        .modifier(LikeEffect(offsetValue: likes))
                }
                .padding()
            }
        }
    }
}
{% endhighlight %}


As you can see it is just reusing of the priciples from our demo. It demonstrates the distinct effect of ButtonStyle that is changing button appearance on tap and animation being done after the tap is triggered. Notice that no animation is performed when the tap is cancelled which is yet another benefit,

### Additional transformation during animation###

Into this category our first example fits, but lets have a look on something different. 

What about some fancy menu? Here we will demonstrate how well you can combine effect of standard modifiers with GeometryEffect. Lets have a look at the result and the code below.

![jumpyMenu]

{% highlight swift %}
struct JumpyEffect: GeometryEffect {

    var offsetValue: Double // 0...1
    
    var animatableData: Double {
        get { offsetValue }
        set { offsetValue = newValue }
    }
    
    func effectValue(size: CGSize) -> ProjectionTransform {
        let reducedValue = offsetValue - floor(offsetValue)
        let value = 1.0-(cos(2*reducedValue*Double.pi)+1)/2

        let translation   = CGFloat(-50*value)
        
        let affineTransform = CGAffineTransform(translationX: translation, y: 0)
        
        return ProjectionTransform(affineTransform)
    }
}

struct JumpyMenuView: View {
    @State var selectedOption : Int = 0
    @State var menuOffset : Double = 0
    
    let itemHeight: CGFloat = 40
    
    var body: some View {
        HStack (alignment: .top){
            Circle()
                .fill(Color.orange)
                .frame(width:10, height:10)
                .offset(x: 20, y: (CGFloat(self.selectedOption) * self.itemHeight + 15.0) )
                .modifier(JumpyEffect(offsetValue: self.menuOffset))
            VStack (alignment: .leading, spacing: 0){
                ForEach(0..<6) { index in
                    Button(action:{
                        withAnimation(.spring()) {
                            self.selectedOption = index
                            self.menuOffset += 1
                        }
                    }) {
                        HStack {
                            Image(systemName: "\(index).circle")
                            Text("Jumpy Menu Item")
                        }
                    }
                    .frame(height: self.itemHeight)
                    .rotation3DEffect(Angle(degrees: self.selectedOption == index ? -30 : 5), axis: (x: 0, y: 1, z: 0))
                }
            }
        }
    }
}
{% endhighlight %}

Here the `JumpyEffect` implementation is even simpler than before - we are just altering `x` coordinate of the orange circle. Notice that the `y` coordinate is controlled by standard `.offset` modifier and only the jumpy part is handled by the GeometryEffect. 
Such combinations in multiple axis allows us to **break animation into smaller pieces** than writing complex paths.

That is all for today, you can find all codes at this [github repo]


*Did you like this article?*

*Feel free to comment or criticise so the next one is even better. Or share it with other SwiftUI adopters ;)*



[GeometryEffect]: https://developer.apple.com/documentation/swiftui/geometryeffect
[SwiftUI]: https://developer.apple.com/documentation/swiftui
[tutorials]: https://developer.apple.com/tutorials/swiftui/creating-and-combining-views
[rotationEffect]:https://developer.apple.com/documentation/swiftui/view/3278649-rotationeffect
[ProjectionTransform]: https://developer.apple.com/documentation/swiftui/projectiontransform
[ButtonStyle]: https://developer.apple.com/documentation/swiftui/buttonstyle
[Swift Talk #166]: https://talk.objc.io/episodes/S01E166-geometry-effects
[github repo]: https://github.com/izakpavel/GeometryEffectAnimations

[sliderImage]: /assets/posts/sliderImage.gif "Moving slider"
[sliderButtons]: /assets/posts/sliderButtons.gif "Controlling position with buttons"
[rotationEffect]: /assets/posts/rotationEffect.gif "GeometryEffect animates view properly"
[likeButton]: /assets/posts/likeButton.gif "LikeButton with onTap animation"
[jumpyMenu]: /assets/posts/jumpyMenu.gif "Special menu that combines various modifiers and GeometryEffect"