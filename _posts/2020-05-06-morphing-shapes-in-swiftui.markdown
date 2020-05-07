---
layout: post
title: "Morphing shapes in SwiftUI"
date: 2020-05-06
author: Pavel Zak
categories: development
tags:	swiftUI shape animation animatableData animatablePair animatableVector path morphing transition
cover:  "/assets/posts/09_cover.jpg"
---

Hello and welcome to another blog post about [SwiftUI] animations. In the [previous post] dedicated mainly to `AnimatableData`, we have constructed `AnimatableVector` that allowed us to create animatable charts. Today we will utilize the same class for morphing shapes.  


## Shape representation

Shapes in SwiftUI can be constructed as a composition of vector paths and/or shape primitives. If we want to create morphing animation between two shapes, we need to find universal representation first. This can be done either ad-hoc for specific shapes - as we saw in the previous post about [custom controls] - but it is better to find a more generic one that will allow us to morph any shape to any shape without limitations.

The most common approach is to describe the shape as an approximation of its outline using a finite set of line segments. This means, that we will distribute `N` points around the shape outline and when doing the morphing animation, we can easily move these outline points from the position given by shape A to the new position in the shape B. For better understanding, check the following illustration:

![morphExplanation]

## Implementation of morphable shape in SwiftUI

So how to implement all of this in SwiftUI? Let me start with a new shape representation based on multiple control points. This is quite easy, the only crucial part is that these control points are being represented using our `AnimatableVector` so it is capable to morph using animation. 

But hey - the shape points are two-dimensional (having x and y coordinate), while our `AnimatableVector` holds only an array of Doubles. What can be done about that? Well, there are two solutions:
* you can implement another custom vector holding array of `CGPoints` and let it conform to the `AnimatableData` protocol
* or use the same `AnimatableVector` and expect even indexed component to be x coordinate and odd-indexed components to by y coordinate. I choose this solution for the following `MorphableShape` implementation:

{% highlight swift %}
struct MorphableShape: Shape {
    var controlPoints: AnimatableVector
    
    var animatableData: AnimatableVector {
        set { self.controlPoints = newValue }
        get { return self.controlPoints }
    }
    
    func point(x: Double, y: Double, rect: CGRect) -> CGPoint {
        // vector values are expected to by in the range of 0...1
        return CGPoint(x: Double(rect.width)*x, y: Double(rect.height)*y)
    }
    
    func path(in rect: CGRect) -> Path {
        return Path { path in
            
            path.move(to: self.point(x: self.controlPoints.values[0], 
				                     y: self.controlPoints.values[1], rect: rect))
            
            var i = 2;
            while i < self.controlPoints.values.count-1 {
                path.addLine(to:  self.point(x: self.controlPoints.values[i], 
					                         y: self.controlPoints.values[i+1], rect: rect))
                i += 2;
            }
            
            path.addLine(to:  self.point(x: self.controlPoints.values[0], 
				                         y: self.controlPoints.values[1], rect: rect))
        }
    }
}
{% endhighlight %}

Let's check the implementation using several random points:

![morphExample]

{% highlight swift %}
// hepler func creating vector with random values
func randomVector(count: Int)->AnimatableVector {
    let randomValues = Array(1...count).map{_ in Double.random(in: 0...1.0)}
    return AnimatableVector(with: randomValues)
}

// demo view for our proof of concept of morphable shapes
struct DemoView: View {
    static let pointCount = 8 // number of control points
	// vector holds twice as many elements
    @State var controlPoints: AnimatableVector = randomVector(count: pointCount*2) 
    
    var body: some View {
        VStack {
            MorphableShape(controlPoints: self.controlPoints)
                .fill(Color.orange)
                .frame(width: 256, height: 256)
                .overlay( // overlay the shape with the same shape to create outline
                    MorphableShape(controlPoints: self.controlPoints)
                        .stroke(Color.white, lineWidth: 3)
                )
            
            Button (action: {
                withAnimation { // feel free to play with animation curves here
                    // randomize points
                    self.controlPoints = randomVector(count: DemoView.pointCount*2)
                }
            }){
                Text("Randomize")
            }
        }
    }
}
{% endhighlight %}

## Converting standard shape to MorphableShape

All the above code is enough to morph any shape to any shape, but getting the shape representation as a set of control points may be tricky. Well, it is definitively something you cannot put together "by hand".

Luckily, there is a way how to generate them using SwiftUI [Path] API. We can sample the shape outline using the `trimmedPath()` method, so let me create a simple Path extension, that provides a vector of N control points along any Path:

{% highlight swift %}

extension Path {
    // return point at the curve
    func point(at offset: CGFloat) -> CGPoint {
        let limitedOffset = min(max(offset, 0), 1)
        guard limitedOffset > 0 else { return cgPath.currentPoint }
        return trimmedPath(from: 0, to: limitedOffset).cgPath.currentPoint
    }
    
    // return control points along the path
    func controlPoints(count: Int) -> AnimatableVector {
        var retPoints = [Double]()
        for index in 0..<count {
            let pathOffset = Double(index)/Double(count)
            let pathPoint = self.point(at: CGFloat(pathOffset))
            retPoints.append(Double(pathPoint.x))
            retPoints.append(Double(pathPoint.y))
        }
        return AnimatableVector(with: retPoints)
    }
}
{% endhighlight %}

Now, it can be utilized like this: 

{% highlight swift %}
let N = 100
let shape = Circle() // or any other shape
let shapeControlPoints: AnimatableVector = shape.path(in: CGRect(x: 0, y: 0, width: 1, height: 1))
                                                .controlPoints(count: N)
{% endhighlight %}

Please, note, that with setting the `CGRect` size to 1 we are assuring to have the control point coordinates in the range `0...1` which is exactly what our `MorphableShape` expects to get.

And that's it! 

![animatedShapes]

## Use cases

If you are wondering, what is this all good for, I present here several ideas:

* morphing of icons (for multi-state buttons)
![recording]

* custom transitions and morphing of clip masks
![heart]

* creating a pointless music video in SwiftUI 🤪

<center>
<iframe width="560" height="315" src="https://www.youtube.com/embed/r_XorK0cjv8" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</center>

## Final notes

* Described algorithm works best for one-component/compact shapes. It works also for multi-component shapes, but the resulting animation may not be eye-pleasant. 
* For large curved or complex shapes, the number of control points needs to be quite high (hundreds at least) to have a smooth result.
* The only tricky thing with this approach is that the shapes may differ in the origin and direction of how they are being constructed. If these parameters are different, the morphing animation would not work as expected and shape may "flip" during interpolation. When creating my custom shapes, I usually start the path at the top-left corner and construct the shape in the clockwise direction to avoid these issues. Nevertheles, the control points could be programatically rearranged so the vector always start around the same orientation - but so far I have not implemented this part:)
* It would be awesome, if we could get Path from SFSymbols....🤞


*Did you like this article? What do you want me to focus on next?*

*Feel free to comment or criticize so the next one is even better. Or share it with other SwiftUI adopters ;)*



[SwiftUI]: https://developer.apple.com/documentation/swiftui
[Path]: https://developer.apple.com/documentation/swiftui/path
[custom controls]: https://nerdyak.tech/development/2019/11/28/creating-custom-views-in-swiftui.html
[previous post]: https://nerdyak.tech/development/2020/01/12/animating-complex-shapes-in-swiftui.html


[animatedShapes]: /assets/posts/07_shapes.gif "Demonstration of morphing of various shapes"
[morphExplanation]: /assets/posts/09_explanation.gif "Morphing using interpolation of control points"
[morphExample]: /assets/posts/09_example.gif "Morphable shape ready to be animated"
[recording]: /assets/posts/09_recording.gif "Example of morphing icon on payer view"
[heart]: /assets/posts/09_heart.gif "Example of morphing transition"


