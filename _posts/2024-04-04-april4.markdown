---
layout: post
title:  "cses Range Updates and Sums"
date:   2024-04-04 03:13:00 -0700
---
[problem](https://cses.fi/problemset/task/1735/)  
[my submission](https://cses.fi/paste/4f4b13d8100becce8830e1/)  

The idea is to use a lazy segment tree to get the sum of ranges in log N time complexity.

```c++
int A[N];
long long t[N << 2];
```
The standard segment tree idea is that the sum of a vertex's segment is equal to the sum of vertex's children's segments.
This is made clear in a segment tree's construction.
```c++
void build(int i, int l, int r) {
	if (l == r) {
		t[i] = A[l];
	}
	else {
		int m = l + (r - l) / 2;
		
		build(i<<1, l, m);
		build(i<<1|1, m + 1, r);
		t[i] = t[i<<1] + t[i<<1|1];
	}
}
```

However, an eager as opposed to lazy range update on a segment tree might affect many ranges, degrading the log N time complexity of operations.
For example, adding 1 to every element in the array would require the entire tree to be calculated again.  

You can think of a lazy update to a range as completed for that range, but pending for its sub-range. Prior to any descent into those subranges,
you should propagate the changes.  

For example, say I add 1 to all values in the array: I can increase the segment tree's root sum and remember that 1 was added to all values with a lazy tag.
Any reads to that exact range will just return the range's sum. But now what if I want the sum of a range within the one I just lazily updated? Look below.

Ignore the set operation for now. Notive the new lazy add member added to each vertex.
```c++
struct node {
	ll sum;
	ll lz_add;
	node() {}
} t[N << 2];
```

```c++
void add(int i, int l, int r, int a, int b, ll x) {
	if (a > b) {
		return;
	}
	else if (l == a && r == b) {
		t[i].lz_add += x;
		t[i].sum += x * (r - l + 1);
	}
	else {
		// 1. what should be done right here?
		
		int m = l + (r - l) / 2;
		add(i<<1, l, m, a, min(b, m), x);
		add(i<<1|1, m + 1, r, max(a, m + 1), b, x);
		
		// 2. and what should be done here?
	}
}
```

1. Prior to any descent into those subranges, you have to propagate the changes you made and clear the lazy tag.  

2. Since you are going to update a lower range lazily as well, the segment tree invariant needs to be upholded.  

item 2 is quite simple:
```c++
t[i] = t[i<<1] + t[i<<1|1];
```

I will leave item 1 to you. Look at my submission or the usaco guide if you are confused.
[USACO guide](https://usaco.guide/plat/RURQ)
