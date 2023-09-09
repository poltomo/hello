---
layout: post
title:  "how to check if a point is in a triangle"
date:   2023-09-08 15:06:34 -0700
# categories:
# permalink:
---

Problem: [cses point location test][https://cses.fi/problemset/task/2189/]

C++ solution:

{% highlight c++ %}
#include<iostream>


using namespace std;

#define REP(i,N) for (int i = 0;i < N;++i)

#define ll long long

#define x first
#define y second

int main() {
	int T;
	cin >> T;

	pair<int, int> a;
	pair<ll, ll> b;
	pair<ll, ll> r;

	REP(i, T) {
		cin >> a.x >> a.y >> b.x >> b.y >> r.x >> r.y;

		b.x = b.x - a.x;
		b.y = b.y - a.y;

		r.x = r.x - a.x;
		r.y = r.y - a.y;

		a.x = 0;
		a.y = 0;

		ll cross = (b.x * r.y) - (r.x * b.y);

		if (cross == 0)
			cout << "TOUCH" << endl;
		else
			cout << ((cross < 0) ? "RIGHT" : "LEFT") << endl;


	}
}
}
{% endhighlight %}

The cross product of vector a and vector b in R^2 is (a.x * b.y) - (b.x * a.y) or the determinant of matrix A = [ a b ].

The cross product is not commutative. If the cross product is 0, then the vectors are colinear. If the cross product is less than 0, then the first vector (in the previous case, a) is to the left of vector b.

The solution is just a matter of using this fact on the vectors: (b - a) and (r - a).
