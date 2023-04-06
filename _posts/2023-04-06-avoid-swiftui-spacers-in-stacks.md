---
layout: post
title: "Avoid Spacers in SwiftUI Stacks"
date: 2023-04-06
author: Pavel Zak
categories: development
tags:	SwiftUI Spacer Stack Frame
---


As I teach SwiftUI here and there I have noticed a particular pattern that is being used and I would like to comment on a possible issue it can lead to. Let's explore it!

![image1]

This is a widespread structure in various apps and a very common way how to code it in SwiftUI:

{% highlight swift %}
HStack(spacing: 12) {
	Text(self.text)
	Spacer()
	Image(systemName: "tortoise.fill")
}
{% endhighlight %}

At first glance, everything looks fine. But what if the text starts to overflow? Suddenly it feels, that the wrapping leaves too much space between the text and the trailing icon even though we have set it to some fixed value... 

Hmm, let's examine by replacing the views with just colors: 

{% highlight swift %}
HStack(spacing: 12) {
	Color.blue
	Spacer()
	Color.red
}
{% endhighlight %}


Aha, we are right! The space between is twice as wide as it should be. So the Stack here, even though the spacer does not add anything, puts declared spaces around it.

![image2]

So how to fix it? You can either remove the spacing parameter from the stack and use explicit padding to its child views, or IMO more convenient way is to make one of the views stretchable using a *.frame* modifier and setting its *maxWidth* attribute to *.infinity* like so: 

{% highlight swift %}
HStack(spacing: 12) {
	Text(self.text)
		.frame(maxWidth: .infinity, alignment: .leading)
	Image(systemName: "tortoise.fill")
}
{% endhighlight %}


Let's compare the behavior of before and after approaches:

![image3]

I personally like the latter solution as it
- brings simpler code structure
- has the flexibility to set the *alignment* of the stretched view (ideal for centering of the view content when needed)
- prevents padding/spacing issues when having optional views in the Stacks


[image1]: /assets/posts/14_cell.png "Typical view layout"
[image2]: /assets/posts/14_colors.png "Expected layout and the actual issue"
[image3]: /assets/posts/14_comparison.png "Comparing solutions"