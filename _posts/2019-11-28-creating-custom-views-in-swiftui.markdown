---
layout: post
title: "Custom controls in SwiftUI"
date: 2019-11-28
author: Pavel Zak
categories: development
tags:	swiftUI shape animation toggle path
cover:  "/assets/posts/gameCover.jpg"
---

In my recent project, I have created custom toggle for baby gender selection. In this post, I would like to demonstrate, how to create such a custom view in [SwiftUI] and comment it in the order how I usually approach these things.
But first, let's have a look at what I am aiming for.
TODO gif/video

## Start with Shape
I usually start with the preparation of custom shapes and paths that will be later used in view composition. Drawing shapes using lines and arcs is pretty straightforward, you only need to be cautious about the point coordinates. I always prepare myself small shape annotation on the paper before I start coding.
TODO shape annotation
TODO code
## Shape transformation

Having the basic shape ready, it is time to allow it to transform from the starting Venus symbol to the Mars symbol. Here I will make things easier for me and I will accomplish the transformation from cross to arrow just by moving 4 points like this.
TODO drawing
Let's add a new property called arroweness, that will control the position of transformed points and set it via animatableData property. This will allow SwiftUI to animate the shape just by the change of our arroweness property.
TODO CODE

## View Composition
The final Toggle View is Composed from two main views stacked in ZStack: - First, there is a rounded rectangle with gradiend overlay - and on the top there is the gender shape overlayed with white circle, which will make the illusion of actual Toggle.
TODO CODE

## State switching
By default both the views are centered along the y-axis. To implement the state changing animation we need to do the following - add main state property that can be linked to our model via @Binding - change y coordinate of foreground arrow based on the state - change rotation of gender symbol



[SwiftUI]: https://developer.apple.com/documentation/swiftui


[finished]: /assets/posts/tweet.png "Tweet about our bet"
[gameConcept]: /assets/posts/gameConcept.jpg "Game mechanics"
[oldPrototype]: /assets/posts/oldPrototype.jpg "Old prototype"
[baby]: /assets/posts/baby.jpg "Baby-sitting at work"
