---
layout: post
title: "Expanding Animations in SwiftUI Lists"
date: 2026-03-16
author: Pavel Zak
categories: development
tags:	SwiftUI List Animations Expanding DisclosureGroup
---

# Expanding Animations in SwiftUI Lists

Ever tried to add a simple expand/collapse animation to a SwiftUI `List` and watched in horror as your beautiful animation turned into a janky mess? You're not alone. I faced this issue a couple of months ago and thanks to Donny Wals, I decided to publish my findings.

Let me walk you through the journey of solving this problem, complete with all the failed attempts and eventual success.


## The Goal

We want a simple expandable view: a tappable title that reveals additional content when clicked. For our example, let's use these simple components:

```swift
struct TitleView: View {
    var body: some View {
        Text("Tap to expand/collapse")
            .font(.headline)
            .frame(maxWidth: .infinity, alignment: .leading)
            .padding()
            .contentShape(Rectangle())
    }
}

struct DetailView: View {
    var body: some View {
        Text("Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat.")
            .font(.body)
            .fixedSize(horizontal: false, vertical: true)
            .padding()
    }
}
```

Now let's build a typical expandable view:

```swift
struct TypicalExpandableView: View {
    @State private var isExpanded = false

    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            TitleView()
                .onTapGesture {
                    isExpanded.toggle()
                }

            if isExpanded {
                Divider()
                DetailView()
                    .transition(.opacity)
            }
        }
        .background(AppColor.backgroundSecondary)
        .cornerRadius(8)
        .animation(.default, value: isExpanded)
    }
}
```

Simple, right? And it works *beautifully* in a regular `VStack` or `LazyVStack`. The content smoothly animates in and out, everything looks great.

Let's put our `TypicalExpandableView` into a List and see what happens.

<center>
<video autoplay="" muted="" loop="" controls="controls" width="400">
	<source src="/assets/posts/17-expandable-broken.mov">
	<source src="/assets/posts/17-expandable-broken.webm" type="video/webm">
</video>
</center>

Oof. That's... not great. The content just *jumps* into existence. The List cell snaps to its new height without any smooth transition. It works perfectly in a `VStack` or `LazyVStack`, but in a `List`? Not so much.


## Second Attempt: Tweaking the Animation

Maybe we just need to be more explicit with our animations? Let's try wrapping the toggle in `withAnimation` and adding an `.id()` to help SwiftUI track the view:

```swift
struct TweakedExpandableView: View {
    @State private var isExpanded = false

    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            TitleView()
                .onTapGesture {
                    withAnimation {
                        isExpanded.toggle()
                    }
                }

            if isExpanded {
                Divider()
                DetailView()
                    .transition(.opacity)
            }
        }
        .background(AppColor.backgroundSecondary)
        .cornerRadius(8)
        .id("static")
    }
}
```

<center>
<video autoplay="" muted="" loop="" controls="controls" width="400">
	<source src="/assets/posts/17-expandable-tweaked.mov">
	<source src="/assets/posts/17-expandable-tweaked.webm" type="video/webm">
</video>
</center>

Better! But still not smooth. The animation is less jarring, but we're still getting that awkward jump. List cells just don't like conditional content appearing and disappearing.

## Third Attempt: The Built-In Solution

"Wait," you might think, "doesn't SwiftUI have a built-in component for this?"

Yes! Meet `DisclosureGroup`:

```swift
struct DisclosureExpandableView: View {
    @State private var isExpanded = false

    var body: some View {
        DisclosureGroup(
            isExpanded: $isExpanded,
            content: {
                DetailView()
            },
            label: {
                TitleView()
            }
        )
        .background(AppColor.backgroundSecondary)
        .cornerRadius(8)
    }
}
```

<center>
<video autoplay="" muted="" loop="" controls="controls" width="400">
	<source src="/assets/posts/17-expandable-default.mov">
	<source src="/assets/posts/17-expandable-default.webm" type="video/webm">
</video>
</center>

Finally, a smooth animation! `DisclosureGroup` works perfectly in Lists. So... problem solved?

**Not quite.** `DisclosureGroup` comes with some limitations, mainly the lack of customization options. If you're OK with the default disclosure indicator animation and animation timing, like on your rarely visited settings screen, you're fine. But for a custom-designed UI, we need more control.

## Fourth Attempt: Animating cell height

After some experimentation, I found that using an Animatable view that can animate its height works nicely even in a List. Here is the proof-of-concept implementation:

```swift
struct AnimatableRedBox: View, Animatable {
    var animatableData: CGFloat

    var body: some View {
        Color.red
            .frame(height: animatableData)
    }
}

struct AnimatableExpandableView: View {
    @State private var isExpanded = false

    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            TitleView()
                .onTapGesture {
                    withAnimation {
                        isExpanded.toggle()
                    }
                }

            AnimatableRedBox(animatableData: isExpanded ? 100 : 0)
        }
        .background(AppColor.backgroundSecondary)
        .cornerRadius(8)
        .id("static")
    }
}
```

<center>
<video autoplay="" muted="" loop="" controls="controls" width="400">
	<source src="/assets/posts/17-expandable-red.mov">
	<source src="/assets/posts/17-expandable-red.webm" type="video/webm">
</video>
</center>

Aha! The red box animates smoothly in the List. By implementing `Animatable` and animating a single `CGFloat` value, SwiftUI smoothly interpolates between states. The List knows exactly what height the cell should be at every frame of the animation.


## The Final Solution

Now we just need to generalize this approach to work with any content - not just a fixed-height red box, but dynamic text and self-sizing views. Here is my implementation of a simple expandable component I'm using in my codebase. Having the detail view in an overlay ensures that the detail text doesn't change during expansion/collapsing. It allows you to tweak the detail animation even more with offsets/scales/distortions or more ;)

```swift
struct ExpandableBaseView<Header: View, Content: View>: View, Animatable {
    var animatableData: CGFloat

    let header: Header
    let content: Content

    @State var baseFrame: CGRect = .zero
    @State var expansionFrame: CGRect = .zero

    init(animatableData: CGFloat,
         @ViewBuilder header: () -> Header,
         @ViewBuilder content: () -> Content) {
        self.animatableData = animatableData
        self.header = header()
        self.content = content()
    }

    var expandedContent: some View {
        content
            .opacity(animatableData)
            .getFrame($expansionFrame, space: .local)
            .offset(y: baseFrame.height)
    }

    var body: some View {
        ZStack(alignment: .bottom) {
            header
                .frame(maxWidth: .infinity)
                .getFrame($baseFrame, space: .local)
                .overlay(expandedContent, alignment: .top)
        }
        .frame(height: baseFrame.height + expansionFrame.height * animatableData,
               alignment: .top)
    }
}

struct WorkingExpandableView: View {
    @State private var isExpanded = false

    var body: some View {
        ExpandableBaseView(
            animatableData: isExpanded ? 1 : 0,
            header: {
                TitleView()
                    .onTapGesture {
                        withAnimation() {
                            isExpanded.toggle()
                        }
                    }
            },
            content: {
                DetailView()
            }
        )
        .background(AppColor.backgroundSecondary)
        .cornerRadius(8)
    }
}
```

<center>
<video autoplay="" muted="" loop="" controls="controls" width="400">
	<source src="/assets/posts/17-expandable-final.mov">
	<source src="/assets/posts/17-expandable-final.webm" type="video/webm">
</video>
</center>

Perfect! Smooth animation, works in Lists, and supports any content you throw at it.

## How It Works

The magic happens by:

1. **Implementing `Animatable`** - Animates a single value from 0 to 1 when toggling isExpanded flag
2. **Measuring content sizes** - Uses SwiftUI's `GeometryReader` to measure both header and expanded content
3. **Calculating exact heights** - Sets frame height to `headerHeight + contentHeight * progress`
4. **Positioning content** - Uses overlays and offsets to position expanded content below the header

The content exists during the entire animation - it's just scaled from 0 to full height smoothly.

### The Frame Measurement Helper

The `getFrame()` helper is a custom view modifier that uses `GeometryReader` and preference keys to measure view dimensions without affecting layout. Here's how it works:

```swift
struct FrameMeasurePreferenceKey: PreferenceKey {
    typealias Value = [String: CGRect]

    static var defaultValue: Value = [:]

    static func reduce(value: inout Value, nextValue: () -> Value) {
        value.merge(nextValue()) { $1 }
    }
}

struct MeasureFrameViewModifier: ViewModifier {
    @Binding var frame: CGRect
    let space: CoordinateSpace
    @State private var frameId = UUID()

    func body(content: Content) -> some View {
        content
            .background(
                GeometryReader { geometry in
                    Color.clear
                        .preference(
                            key: FrameMeasurePreferenceKey.self,
                            value: ["frame_\(frameId)": geometry.frame(in: space)]
                        )
                }
            )
            .onPreferenceChange(FrameMeasurePreferenceKey.self) { preferences in
                if let frame = preferences["frame_\(frameId)"] {
                    self.frame = frame
                }
            }
    }
}

extension View {
    func getFrame(_ frame: Binding<CGRect>, space: CoordinateSpace) -> some View {
        modifier(MeasureFrameViewModifier(frame: frame, space: space))
    }
}
```

This utility:
- Uses a hidden `GeometryReader` in the background to measure view size
- Reports the measurement through SwiftUI's preference system
- Doesn't interfere with the view's layout or appearance
- Updates automatically when the view's size changes

With this helper, `ExpandableBaseView` can dynamically measure both the header and content sizes, then calculate the exact height needed at each point in the animation.

## Wrapping Up

I hope you find this useful! Let me know if you run into any troubles, or even a happy smiley face in DMs is rewarding. Get in touch on [Twitter]!

Aaand Happy animating! 🎨

[Twitter]: https://twitter.com/myridiphis