---
layout: post
title: "Magical Particle Effects with SwiftUI Canvas"
date: 2024-06-21
author: Pavel Zak
categories: development
tags:	SwiftUI Canvas Particles BlendMode TimelineView
---

In one of the previous posts, I shared a simple way of [Creating particle effects in SwiftUI]({% link _posts/2020-12-12-create-particles-in-swiftui.md %}). The approach is super easy and utilizes the power of viewModifiers, but I would not recommend it for production use as it is performance-greedy when having a bigger amount of particles in place (because each particle is a single view)

In this post, I will introduce you to an **alternate and better** approach - rendering the particles with the `Canvas` view. So let's get into it 💪.

## Setup

We will start with the following view outline:

{% highlight swift %}
struct ParticleCanvasView1: View {
    
    var body: some View {
        TimelineView(.animation) { context in
            Canvas { context, size in
	       let particleSymbol = context.resolveSymbol(id: 0)!
                let position = CGPoint(x: size.width/2, y: size.height/2)                    
                context.draw(particleSymbol, at: position, anchor: .center)
            } symbols: {
                SingleParticleView()
                    .tag(0)
            }
        }
    }
}
{% endhighlight %}


This view features an outer [TimelineView] that ensures its content (inner view) is regularly re-drawn. (note the `.animation` parameter allowing the system to decide the optimal refresh rate)
The content here is a [Canvas] view. Those of you who come from good old UIKit days might be already familiar with the concept of the drawing context. In simple words, we get a canvas area with the view dimensions and we can draw/rasterize
various entities on it - like shapes and images.

In our case, we will draw a single particle represented with a `SingleParticleView`. Please note, how the SingleParticleView is being used as a **drawing element**. It is added to a symbols parameter allowing SwiftUI to **pre-render** it and thus be very performant later in the drawing calls - thus an ideal candidate for the many particles in the place ;)


At this moment, let's just set the `SingleParticleView` as an orange dot, but we will tune it soon

{% highlight swift %}
struct SingleParticleView: View {
    var body: some View {
        Circle().fill(Color.orange)
            .frame(width:5, height:5)
    }
}
{% endhighlight %}

## I like to move it

Now, let's move that particle. 

I will build here a fire-ish effect blending multiple upwards moving particles - so as a good start let's periodically move the single particle up from the canvas bottom:

 {% highlight swift %}
struct ParticleCanvasView1: View {
    let movementDuration = 2.0
    
    var body: some View {
        TimelineView(.animation) { context in
            let timeInterval = context.date.timeIntervalSinceReferenceDate;
            
            let time = timeInterval.truncatingRemainder(dividingBy: movementDuration)/movementDuration
            
            Canvas { context, size in
                let particleSymbol = context.resolveSymbol(id: 0)!
                let position = CGPoint(x: size.width/2, y: (1-time)*size.height)
                    
                context.draw(particleSymbol, at: position, anchor: .center)
            } symbols: {
                SingleParticleView()
                    .tag(0)
            }
        }
    }
}
{% endhighlight %}

You can see, that I am controlling the upwards movement with the *time* variable. What exactly is it in this context? Well, the timeline view already gives us access to the time property, but for our use, I want to have something normalized that can be easily bound with the particle movement.
I want the particle movement to take exactly 2 seconds (see movementDuration) so the code computes time as a truncating remainder, making sure it will periodically grow from 0 to 1 forever. As you can see in the following video:

[TODO video]

## Remember goniometry?

As a next step, we will upgrade the movement from the simple linear to something more firey :). My perception of fire movement is that it is waving, so let me change the code to move the particle along a cosinus wave whose amplitude is smaller the higher the particle is:

{% highlight swift %}
struct ParticleCanvasView: View {
    let movementDuration = 2.0
    
    func particlePosition(timeInterval: Double, canvasSize: CGSize) -> CGPoint {
        let time = timeInterval.truncatingRemainder(dividingBy: movementDuration)/movementDuration
        let rotations:CGFloat = 3
        let amplitude: CGFloat = 0.1+0.8*(1-time)
        let x = canvasSize.width/2 + cos(rotations*time*CGFloat.pi*2)*canvasSize.width/2*amplitude
        
        return CGPoint(x: x, y: (1-time)*canvasSize.height)
    }
    
    var body: some View {
        TimelineView(.animation) { context in
            let timeInterval = context.date.timeIntervalSinceReferenceDate;
            
            Canvas { context, size in
                let particleSymbol = context.resolveSymbol(id: 0)!
                let position = particlePosition(timeInterval: timeInterval, canvasSize: size)
                
                context.draw(particleSymbol, at: position, anchor: .center)
            } symbols: {
                SingleParticleView()
                    .tag(0)
            }
        }
    }
}
{% endhighlight %}

[TODO video]

please note the position computation was moved to a separate function so the Canvas content remains clean.

## Make it many

At this point, I am quite happy with the movement. I am sure we will fine-tune the constants here and there, but that can come later. Now, we want to draw more particles so let's wrap the drawing into the for-cycle like this:

{% highlight swift %}
let particleCount = 100
// …
for i in 0..<particleCount {
    let position = particlePosition(timeInterval: timeInterval+(Double(i)/Double(particleCount)), canvasSize: size)
    context.draw(particleSymbol, at: position, anchor: .center)
}
{% endhighlight %}

[TODO video]

## Randomize

We can finally see more particles, but they all share the same path, so let me initiate each particle with random starting wave rotation and starting time offset:

{% highlight swift %}
struct ParticleCanvasView: View {
    let movementDuration: Double
    let particleCount: Int
    let startingParticleOffsets: [CGFloat]
    let startingParticleAlphas: [CGFloat]
    
    init(particleCount: Int = 200, movementDuration: Double = 3.0) {
        self.particleCount = particleCount
        self.movementDuration = movementDuration
        self.startingParticleOffsets = Array(0..<particleCount).map {_ in CGFloat.random(in: 0...1)}
        self.startingParticleAlphas = Array(0..<particleCount).map {_ in CGFloat.random(in: 0...CGFloat.pi*2)}
    }
    
    func particlePosition(index: Int, timeInterval: Double, canvasSize: CGSize) -> CGPoint {
        let startingRotation: CGFloat =  startingParticleAlphas[index]//CGFloat(index)/CGFloat(particleCount)*CGFloat.pi
        let startingTimeOffset = startingParticleOffsets[index]*movementDuration
        
        let time = (timeInterval+startingTimeOffset).truncatingRemainder(dividingBy: movementDuration)/movementDuration
        let rotations:CGFloat = 3
        let amplitude: CGFloat = 0.1+0.8*(1-time)
        
        let x = canvasSize.width/2 + cos(rotations*time*CGFloat.pi*2+startingRotation)*canvasSize.width/2*amplitude
        
        return CGPoint(x: x, y: (1-time)*canvasSize.height)
    }
{% endhighlight %}

[TODO video]

## Improving the effect appearance

In terms of particle motion, I consider this done, but we still need to fine-tune this effect's appearance to get some juiciness. 

The first improvement is changing the particle opacity during the opacity movement - this is quite simple by changing the context opacity before a draw call: 

{% highlight swift %}
context.opacity = positionAndAlpha.1
{% endhighlight %}

Next, let's utilize the blending capabilities of SwiftUI and set the particle appearance like this:
{% highlight swift %}
struct SingleParticleView: View {
    var body: some View {
        Circle().fill(Color.orange.opacity(0.4))
            .frame(width:35, height:35)
            .blendMode(.plusLighter)
            .blur(radius: 10)
    }
}
{% endhighlight %}
We make particles here as a nice big blurry spots, that blends together to form nice fire volume. The blendMode(.plusLighter) combines overlapping orange dots, effectively brightening the result where the patrticles intersect.

[TODO video]

## “Art“ time

From the theory point of view, I have presented here all the typical steps I go through when creating similar effects. The final step is to experiment with the code, fine-tune the variables, and change or even add more parameters to reach the expected liking,

The final effect I was happy with is:

{% highlight swift %}
struct ParticleCanvasView: View {
    let movementDuration: Double
    let particleCount: Int
    let startingParticleOffsets: [CGFloat]
    let startingParticleAlphas: [CGFloat]
    
    init(particleCount: Int = 200, movementDuration: Double = 3.0) {
        self.particleCount = particleCount
        self.movementDuration = movementDuration
        self.startingParticleOffsets = Array(0..<particleCount).map {_ in CGFloat.random(in: 0...1)}
        self.startingParticleAlphas = Array(0..<particleCount).map {_ in CGFloat.random(in: 0...CGFloat.pi*2)}
    }
    
    func particlePositionAndAlpha(index: Int, timeInterval: Double, canvasSize: CGSize) -> (CGPoint, CGFloat) {
        let startingRotation: CGFloat = startingParticleAlphas[index]
        let startingTimeOffset = startingParticleOffsets[index]*movementDuration
        
        let time = (timeInterval+startingTimeOffset).truncatingRemainder(dividingBy: movementDuration)/movementDuration
        let rotations:CGFloat = 1.5
        let amplitude: CGFloat = 0.1+0.8*(1-time)
        
        let x = canvasSize.width/2 + cos(rotations*time*CGFloat.pi*2 + startingRotation)*canvasSize.width/2*amplitude*0.8
        let y = (1-time*time)*canvasSize.height
        
        return (CGPoint(x: x, y: y), 1-time)
    }
    
    var body: some View {
        TimelineView(.animation) { context in
            let timeInterval = context.date.timeIntervalSinceReferenceDate;
            
            Canvas { context, size in
                let particleSymbol = context.resolveSymbol(id: 0)!
                
                for i in 0..<particleCount {
                    let positionAndAlpha = particlePositionAndAlpha(index: i, timeInterval: timeInterval, canvasSize: size)
                    context.opacity = positionAndAlpha.1
                    context.draw(particleSymbol, at: positionAndAlpha.0, anchor: .center)
                }
            } symbols: {
                SingleParticleView()
                    .tag(0)
            }
        }
    }
}
{% endhighlight %}

[TODO video]

And one more, even more polished example to motivate YOU to come up with something nice.

[TODO video]
## Your turn!

Now it is your time to get creative! 

Let me know, if you find this article helpful, and send me your animations on [Twitter].


[TimelineView]: https://developer.apple.com/documentation/swiftui/timelineview
[Canvas]: https://developer.apple.com/documentation/swiftui/canvas/
[Twitter]: https://twitter.com/myridiphis

[image1]: /assets/posts/15_01.png "Main view"
[image2]: /assets/posts/15_02.png "Transposed view"
[image3]: /assets/posts/15_03.png "Flag effect"
[image4]: /assets/posts/15_04.png "Flag effect"

[video1]: /assets/posts/15_01.png "Transition effect"