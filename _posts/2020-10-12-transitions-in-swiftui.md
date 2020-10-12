---
layout: post
title: "Mastering transitions in SwiftUI"
date: 2020-10-12
author: Pavel Zak
categories: development
tags:	SwiftUI Transition ViewModifier ConditionalView
cover:  "/assets/posts/11_cover.jpg"
---

Transitions play a vital role in the user experience of our apps. They are visual keys signalizing that the app or screen context is changing.

In this article, we will go through all important parts related to implementation of [transitions] in SwiftUI - from the very basics to more advanced techniques. At the end of this article, you will be able to implement transitions like this:

![vid1]


## Triggering the transition

Let us remind, what is the transition:

**Transition is an animation that might be triggered when some View is being added to or removed from the View hierarchy**

In practice, the change in the view hierarchy can usually come from the conditional statements, so let us have a look at the following example. 

{% highlight swift %}
struct BasicTransitionView: View {
    @State var showText = false
    
    var body: some View {
        VStack {
            if (self.showText) {
                Text("HELLO WORLD")
            }
            Button(action: {
                withAnimation() {
                    self.showText.toggle()
                }
            }) {
                Text("Change me")
            }
        }
    }
}
{% endhighlight %}

// TODO video

There is a Text that is displayed only in the situation when 'showText' is true. It is important to mention:

* The state change needs to be done withing 'withAnimation()' block. If you explicitely state the animation here, it will affect the transition timing curve and duration.
* By default, the transition is fade in /fade out. We will see below, how to change it
* Note how layout changes and adapts to additional view causing the button iself to jump a bit lower. This behaviour is completely valid for our situation with VStack, just keep in mind that inserted/removed view may affect surrounding views.

Transition may be triggered also when changing '.id()' of a view. In such situation SwiftUI, removes the old View and creates a new one which may trigger transition on both old and new views.

## Basic transitions

To change default transition we can set up a new one using '.transition' view modifier. There are several types already available for basic view transformations

* .scale
* .move
* .offset
* .slide
* .opacity

In our example, let me demonstrate usage of 'move' transition that shifts view being added/removed from/towards the leading edge of **it's** frame. Feel free to experiment with the other types.

{% highlight swift %}
struct BasicTransitionView: View {
    @State var showText = false
    
    var body: some View {
        VStack {
            if (self.showText) {
                Text("HELLO WORLD")
                    .transition(.move(edge: .leading))
            }
            Button(action: {
                withAnimation() {
                    self.showText.toggle()
                }
            }) {
                Text("Change me")
            }
        }
    }
}
{% endhighlight %}

// TODO video

## Combining transitions

The list of basic transitions is pretty short and most probably would not be sufficient, but luckily we can create more complex transitions with two powerful mechanisms

* transition combination
* transition created from any view modifier

You can combine two transitions using '.combine(with:)' method that returns a new transition that is the result of **both transitions being applied**.

The amount of transition combinations is not limited, so for example this combination of opacity, scale and move transition is perfectly fine:

{% highlight swift %}
.transition( AnyTransition.move(edge: .leading).combined(with: AnyTransition.opacity).combined(with: .scale) )
{% endhighlight %}

![vid4]

## Assymetric transitions

In case you require to perform different transition on removal than on insertion, it is possible to create assymetric transition using static function 'asymmetric(...)', for example

{% highlight swift %}
.transition( AnyTransition.asymmetric(insertion: .scale, removal: .opacity))
{% endhighlight %}

![vid4]

Assymetric transitions are especially handy in situations when the inserted view could overlap removed view and thus produce unaesthetic animation.

## Custom transitions

The creative part begins with the possibility to define your own transition based on view modifiers. Let me start with a very simple example to demonstrate the basics. 

Here is a custom view modifier that masks the view with Rounded rectangle shape. The value parameter is expected to have values between 0 and 1 when 0 clips the view entirely and 1 reveals the full view.

{% highlight swift %}
struct ClipEffect: ViewModifier {
    var value: CGFloat
    
    func body(content: Content) -> some View {
        content
            .clipShape(RoundedRectangle(cornerRadius: 100*(1-value)).scale(value))
    }
}
{% endhighlight %}

Now, we can define custom transition that applies this effect as follows:

{% highlight swift %}
.transition(AnyTransition.modifier(active: ClipEffect(value: 0), identity: ClipEffect(value: 1)))
{% endhighlight %}

Note: To ease future re-usability, it may be beneficial to define your transitions as static members of AnyTransition:

{% highlight swift %}
extension AnyTransition {
    static var clipTransition: AnyTransition {
        .modifier(
            active: ClipEffect(value: 0),
            identity: ClipEffect(value: 1)
        )
    }
}
{% endhighlight %}

when used at our example '.transition(.clipTransition)', the result looks like this:

TODO video

Please note that:

* modifier transition depends on two states: **active and identity**. Identity is applied when the view is fully inserted in the view hierarchy and active when the view is gone. 
* During the transition, SwiftUI interpolates between these two states and thus the type of active and identity modifier **needs to be the same**! (XCode will complain otherwise)


## Mastering transitions

So far we went through quite basic stuff so maybe, you have not relized how powerful all these things are. Making transition from custom view modifiers allows you to unleash your creativity and with a little help of GeometryEffect or AnimatableModifier you can create transitions that stand out. Let me showcase several exmaples:

### Counting up transition

Handy for score or achievement screens, count up to the value of Text view using this simple AnimatableModifier:

{% highlight swift %}
struct CountUpEffect: AnimatableModifier {
    var value: CGFloat
    
    var animatableData: CGFloat {
        get { return value }
        set { value = newValue }
    }
    
    func body(content: Content) -> some View {
        Text("\(self.value, specifier: "%.1f")")
            .font(.title)
            .foregroundColor(self.value<100 ? .primary : .red)
    }
}
{% endhighlight %}

In this case, we need to provide a parameter to the transition itself, so I have prepared simple extension that will provide us with the right transition:

{% highlight swift %}
extension AnyTransition {
    static func countUpTransition(value: CGFloat)->AnyTransition {
        return AnyTransition.modifier(active: CountUpEffect(value: 0), identity: CountUpEffect(value: value))
    }
}
{% endhighlight %}

And for the sake of excercise, we will combine it with the '.scale' transition when applied to the view:

{% highlight swift %}
.transition(AnyTransition.countUpTransition(value: self.textValue).combined(with: .scale))
{% endhighlight %}

TODO VID

### Taking content apart

Combining several layers of content in the view modifier is my absolute favourite way to create unique transitions. The sliding door transition is a nice way how to demonstrate this approach

Once again, let me start with the effect itself:

{% highlight swift %}
struct SlidingDoorEffect: ViewModifier {
    let shift: CGFloat
    
    func body(content: Content) -> some View {
        let c = content
        return ZStack {
            c.clipShape(HalfClipShape(left: false)).offset(x: -shift, y: 0)
            c.clipShape(HalfClipShape(left: true)).offset(x: shift, y: 0)
        }
    }
}

struct HalfClipShape: Shape {
    var left: Bool
        
    func path(in rect: CGRect) -> Path {
        // shape covers lef or right part of rect
        return Path { path in
            let width = rect.width
            let height = rect.height
            
            let startx:CGFloat = left ? 0 : width/2
            let shapeWidth:CGFloat = width/2
            
            path.move(to: CGPoint(x: startx, y: 0))
            
            path.addLines([
                CGPoint(x: startx+shapeWidth, y: 0),
                CGPoint(x: startx+shapeWidth, y: height),
                CGPoint(x: startx, y: height),
                CGPoint(x: startx, y: 0)
            ])
        }
    }
}
{% endhighlight %}

This effect duplicates the view content into ZStack and masks/clips only the left part of the bottom layer and right part of the top layer. Both *copies* of the content are later moved apart depending on the shift parameter.

{% highlight swift %}
Text("HELLO WORLD")
	.padding()
    .background(Color.orange)
    .transition(AnyTransition.modifier(active: SlidingDoorEffect(shift: 170), identity: SlidingDoorEffect(shift: 0)))
{% endhighlight %}

You can go really crazy when mixing multiple occurences of the content so please, bear in mind that

* mixing more content layers may lead to performance issues
* even though I refer to the content layers as *copies* they are **different views of the same type**!
* if the content is animated, it will remain live during the whole transition



## THE Task

The best way to learn something new is to practice it. So for this topic I have a chalenge for you. Implement transition that *explodes* the view on removal. Example:

VID

How to do it?

1) start with a GeometryEffect that animates a view along a curve
2) prepare a masking shape for content sub-regions
3) create explosion effect that multiplies the content into many sub-regions, masking each one of them with relevant clipShape and offseting using the effect from the first step
4) wrap new effect into transition
5) PROFIT


[transitions]: https://developer.apple.com/documentation/swiftui/anytransition


[vid1]: /assets/posts/11_vid1.gif "Demonstration of onboarding screen"
[vid2]: /assets/posts/11_vid2.gif "Implementation using custom SwiftUIPagingScrollView"
[vid3]: /assets/posts/11_vid3.gif "Implementation with custom transitions"
[vid4]: /assets/posts/11_vid4.gif "You can be very creative with transitions"