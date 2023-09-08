---
layout: post
title:  "how to check if a point is in a triangle"
date:   2023-09-08 15:06:34 -0700
categories: jekyll update
---

Problem: given a point r and a triangle defined by points ABC, tell if the point is inside or outside the triangle.

My C++ solution:

{% highlight c++ %}
bool inside_triangle(int rx, int ry, int ax, int ay, int bx, int by, int cx, int cy) {
	int determinant = (by - cy) * (ax - cx) + (cx - bx) * (ay - cy);

	int alpha = ((by - cy) * (rx - cx) + (cx - bx) * (ry - cy)) / determinant;

	int beta = ((cy - ay) * (rx - cx) + (ax - cx) * (ry - cy)) / determinant;

	int gamma = 1 - alpha - beta;

	return alpha >= 0 && alpha <= 1 && beta >= 0 && beta <= 1 && gamma >= 0 && gamma <= 1;
}
{% endhighlight %}

Why does this work?

I'm going to explain how I made this later because I am hungry at the moment. If you
cannot wait, my solution has to do with this: [barycentric_points][https://en.wikipedia.org/wiki/Barycentric_coordinate_system#Determining_location_with_respect_to_a_triangle]


Email me at nawatheyvarun (at) gmail (dot) com if you have questions.
