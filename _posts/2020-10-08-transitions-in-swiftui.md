---
layout: post
title: "Mastering transitions in SwiftUI"
date: 2020-10-08
author: Pavel Zak
categories: development
tags:	SwiftUI Transition ViewModifier ConditionalView
cover:  "/assets/posts/11_cover.jpg"
---

Transitions play a vital role in the user experience of our apps. They are visual keys signalizing that the app or screen context is changing.

In this article, we will go through all important parts related to implementation of transitions in SwiftUI - from the very basics to more advanced techniques.

![vid1]


## Triggering the transition

Let us remind, what is the transition:

**Transition is an animation that might be triggered when some View is being added or removed to the View hierarchy**

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

There is a Text that is displayed only in the situation when 'showText' is true. It is important to mention:

* The state change needs to be done withing 'withAnimation()' block. If you explicitely state the animation here, it will affect the transition timing curve and duration.
* By default, the transition is fade in /fade out. We will see below, how to change it
* Note how layout changes and adapts to additional view causing the button iself to jump a bit lower. This behaviour is completely valid for our situation with VStack, just keep in mind that inserted/removed view may affect surrounding views.

Transition may be triggered also when changing '.id()' of a view. In such situation SwiftUI, removes the old View and creates a new one which may trigger transition on both old and new views.

## Basic transitions

To change default transition we can set up a new one using '.transition' view modifier. There are several basic transition already available for basic view transformations

* .scale
* .move
* .offset
* .slide
* .opacity

In our example, let me demonstrate usage of 'move' transition that shifts view being added/removed from/towards the leading edge of **it's** frame. Feel free to experiment with the other

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

## Combining transitions

The list of basic transitions is pretty short and most probably would not be sufficient, but luckily we can create more complex transitions with two powerful mechanisms

* transition combination
* transition created from any view modifier

You can combine two transitions using '.combine(with:)' method that returns a new transition that is the result of **both transitions being applied**.

The combination of transitions is not limited, so for example this combination of opacity, scale and move transition is perfectly fine

{% highlight swift %}
.transition( AnyTransition.move(edge: .leading).combined(with: AnyTransition.opacity).combined(with: .scale) )
{% endhighlight %}


## Using TabView

If you are familiar with UIKit, you would most probably look for a Scrollview with set property `isPagingEnabled = true` to start with. Unfortunately, SwiftUI does not have the exact counterpart, but that does not mean, there are no solutions for our case. Of course, one can use `UIScrollView` based component and wrap it using `UIViewRepresentable` to SwiftUI, but let me focus on pure SwiftUI solutions only.

Since the recent SwiftUI update, often denoted as SwiftUI2, we can use `TabView` for our purpose. Even though TabView is mainly purposed for Views organized in Tabbar, applying `.tabViewStyle(PageTabViewStyle())` creates exactly what we would expect from a paginated scroll view, it even creates a paging indicator at the bottom of the view! 

*(To remove or customize paging indicator appearance, set `.indexViewStyle(PageIndexViewStyle())` appropriately ;)*

{% highlight swift %}
struct ContentView: View {
    let pages: [IntroPage]
    
    @State private var currentPage = 0
    
    var body: some View {
        VStack {
        
            TabView(selection: $currentPage) {
                ForEach (0 ..< self.pages.count) { index in
                    IntroPageView(page: self.pages[index])
                        .tag(index)
                        .padding()
                }
            }
            .tabViewStyle(PageTabViewStyle()) // the important part
        
			// NEXT button
            HStack {
                Spacer()
                Button(action: {
                    withAnimation (.easeInOut(duration: 1.0)) {
                        self.currentPage = (self.currentPage + 1)%self.pages.count
                    }
                }) {
                    Image(systemName: "arrow.right")
                        .font(.largeTitle)
                        .foregroundColor(Color.white)
                        .padding()
                        .background(Circle().fill(Color.purple))
                }
            }
            .padding()
        }
    }
}
{% endhighlight %}

Nice, clean, and easy. There is only one drawback - this works on iOS14 only... So what if you want/need to support iOS13 as well?

## PagingScrollView based on HStack

As already mentioned, the standard Scrollview in SwiftUI has very limited capabilities compared to UIScrollView. It is not possible to enable pagination and prior to SwiftUI2 it was also impossible to scroll to a specific subview. (Now you can use `ScrollViewReader` for that purpose, but again - supported on iOS14 only)

A bit harder way but with plenty of freedom is to take `HStack` and implement the scrolling using gesture recognizers. With this approach, you had to work with stack offset and change it according to user actions. You can inspire and/or directly use the component [SwiftUIPagingScrollview] that I have prepared and is available on Github. The interface is far from ideal as it requires messing with `GeometryReader`, but for illustration:

{% highlight swift %}
GeometryReader { geometry in
	PagingScrollView(activePageIndex: self.$currentPage, 
					 itemCount: self.pages.count, 
					 pageWidth: geometry.size.width, 
					 tileWidth: geometry.size.width, 
					 tilePadding: 0) {
    	ForEach (0 ..< self.pages.count) { index in
        	IntroPageView(page: self.pages[index])
        }
   	}
}
{% endhighlight %}

This solution gives us great scrolling experience:

![vid2]

## Transitions and `id` modifier

If you are not very strict about direct scrolling with your finger, an interesting approach is to use custom transitions that simulate the scrolling animation. The transitions are being triggered when views are being added or removed from the view hierarchy. You can create any fancy and asymmetric transition using composition of viewModifiers, but for now we will be OK just by combination of standard moving transition, like this:

{% highlight swift %}
extension AnyTransition {
    static var pageTransition: AnyTransition {
        let insertion = AnyTransition.move(edge: .trailing)
            .combined(with: .opacity)
        let removal = AnyTransition.move(edge: .leading)
            .combined(with: .opacity)
        return .asymmetric(insertion: insertion, removal: removal)
    }
}
{% endhighlight %}

Please note that I have added there also opacity change just for the fun and demonstration of transition combinations.

Now, to setup several pages with transitions one could write something like this:

{% highlight swift %}
Group {
	if 0 == self.currentPage {
    	IntroPageView(page: self.pages[0])
    }
    if 1 == self.currentPage {
    	IntroPageView(page: self.pages[1])
    }
    if 2 == self.currentPage {
    	IntroPageView(page: self.pages[2])
    }
    if 3 == self.currentPage {
    	IntroPageView(page: self.pages[3])
	}
}.transition(AnyTransition.pageTransition)

{% endhighlight %}

As you see, that is not very nice and scaleable. (But note the usage of `Group` view that sets the transition to each of its subviews)


Much nicer and more elegant solutuon is to use identity modifier `id` like so:
{% highlight swift %}
IntroPageView(page: pages[self.currentPage])
	.transition(AnyTransition.pageTransition)
	.id(self.currentPage)
{% endhighlight %}

![vid3]

Nice, right? Whenever assigned identifier changes, the view is being replaced with the new one and thus transitions are triggered both for the old view (removal) and new view (insertion).

## Summary

Today I have tried to present several ways of building up onboarding screens in SwiftUI. I have examined three approaches that can satisfy most of the use cases - at least I believe so. Let me review them once again:


### Use TabView

➖ iOS14 only; low *coolness* factor (can be tweaked with parallax effects though); cannot set animation style to tab change

➕ quick and easy

### Implement custom Scrollview based on HStack

➖ non-trivial implementation; mixing with other scrollable components might lead to issues

➕ ability to fine-tune everything; great scrolling feeling

### Use just transitions

➖ no way to mimic true scrollview behavior (if you need it)

➕ clean and easy; you can go crazy with custom transitions, like the one below. But that is for another story and my future blog post 😉.

![vid4]


[swiftui.training session]: https://swiftui.training
[undraw.co]: https://undraw.co/
[SwiftUIPagingScrollview]: https://github.com/izakpavel/SwiftUIPagingScrollView


[vid1]: /assets/posts/11_vid1.gif "Demonstration of onboarding screen"
[vid2]: /assets/posts/11_vid2.gif "Implementation using custom SwiftUIPagingScrollView"
[vid3]: /assets/posts/11_vid3.gif "Implementation with custom transitions"
[vid4]: /assets/posts/11_vid4.gif "You can be very creative with transitions"