---
layout: post
title: "Create transitions with distortion Effect and Metal Shaders"
date: 2023-06-16
author: Pavel Zak
categories: development
tags:	SwiftUI distortionEffect Metal Shaders
---
his year DubDub is over and I am very excited about the new developer treats that iOS17 will bring us that expand the animation possibilities of SwiftUI. I am talking mainly about the [PhaseAnimator], [KeyframeAnimator] and the ability to utilize Metal shaders on SwiftUI views through modifiers .distortionEffect, .layerEffect, and .colorEffect ([docs]).

In this post, we will play with the .distortionEffect and learn, how to utilize it for creating custom transitions, like this:

![image1]


Let's start with the simple Hello world view and set a basic .distortion effect for the view:

{% highlight swift %}
// code
{% endhighlight %}

We need to add a new Metal file to our Xcode project and define the shader function there. As the documentation says: For a shader function to act as a distortion effect it must have a function signature matching:

{% highlight swift %}
[[ stitchable ]] float2 name(float2 position, args...)
{% endhighlight %}

which tells us, that the function can "alter" the position of every pixel of our View. So in the minimum variation, the Shader can look like this

{% highlight swift %}
empty shader
{% endhighlight %}

This is a working shader, but you cannot see any effect on the view as it is basically an identity function that just returns the input positions.

But with just a few simple alterations to the return value, you start seeing how the view image changes. For example, you can switch the x and y axis and get transposed view.

{% highlight swift %}
transponed shader
{% endhighlight %}

![image2]

Notice, that the shader cannot extend our view dimensions, so if we are transposing the rectangular view, the result will be cropped. But we learned an important lesson here: if we are returning an invalid position from the shader, it just leaves the pixel transparent. And this fact we are gonna utilize for the transitions later.

But before we move to that, let's see, how to extend the shader with more parameters. From the initial function signature we see, that the shader is not aware of the view frame, so naturally, we want to provide at least that parameter. Adding a new param is simple as that, just extend the function with more parameters and provide those parameters in the SwiftUI part like so.

Very easily, we are now able to create waved distortion effect like this:

{% highlight swift %}
wave example
{% endhighlight %}

![image2]

Try to experiment with the shader parameters and messing the output with various functions. How fun, right?

Now, let's utilize what we have learned to build a custom view transition. I want to create an effect, that shifts away the content of the view like so:

![image2]

for that, I extend the shader with one more parameter named effectValue, which controls the offset of the view content. My shader looks like this:

{% highlight swift %}
shift shader
{% endhighlight %}

and for the testing, I am using a simple slider just to be sure that it behaves correctly for all effectValues between 0 and 1.

Once we are happy with the basic effect, we just wrap it into a view modifier:

{% highlight swift %}
view modifier
{% endhighlight %}

And the last step is to use the modifier for a custom Transition definition:

{% highlight swift %}
view modifier
{% endhighlight %}

And we are done. Now it is your time to get creative! Let me know, if you found this article helpful, and send me your animations at [twitter].


[PhaseAnimator]: https://developer.apple.com/documentation/swiftui/phaseanimator/
[KeyframeAnimator]: https://developer.apple.com/documentation/swiftui/keyframeanimator
[docs]: https://developer.apple.com/documentation/swiftui/view-graphics-and-rendering#shaders
[docs]: https://developer.apple.com/documentation/swiftui/view-graphics-and-rendering#shaders

[image1]: /assets/posts/14_cell.png "Typical view layout"
[image2]: /assets/posts/14_colors.png "Expected layout and the actual issue"
[image3]: /assets/posts/14_comparison.png "Comparing solutions"