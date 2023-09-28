---
layout: post
title:  "problem - september 24, 2023"
date:   2023-09-24 15:06:34 -0700
---

problem:

you are given an array of n integers. In one operation, you can choose two indices i and j, and delete arr[i] if 2* arr[i] ≤ arr[j]. You cannot use a particular element more than once.

Return the minimum size of the array after performing the operation any number of times.


examples:

N = 4
arr = [1, 2, 3, 4]

operation 1: choose 1 and 3. The array becomes [2, 3, 4]
operation 2: choose 2 and 4. The array becomes [3, 4]

the answer is 2.

---

Let me know if you think I am wrong by email. I appreciate any possible learning.

Solution:

Notice that the minimum size you can ever reach after any number of operations is the half the initial size N.

look at all elements as candidates for removal for a moment. 1 can be removed by any number in [2, inf). 2 can be removed by any number in [4, inf). for 3 the range is [6, inf) and for 4 the range is [8, inf). Note that smaller elements have greater ranges than bigger ones.


This should hopefully lead you to the fact that if x < y, then x can always match with elements that y can match with, but the same cannot necessarily be said for y.

Take this example:

1 2 3 4

2 can only match with 4, but 1 can match with 2, 3, and 4

So, larger elements should be considered before smaller elements. Since only N/2 elements can be removed in the best case, the smallest 50% of numbers should be considered in descending order.

{% highlight c++ %}
#include<iostream>
#include<vector>
#include<algorithm>


using namespace std;

#define REP(i,N) for (int i = 0;i < N;++i)

#define ll long long

int main() {
	int N;
	cin >> N;

	vector<int> A(N);

	REP(i, N) cin >> A[i];

	sort(A.begin(), A.end());

	int j = N - 1;
	int cnt = 0;

	for (int i = N / 2 - 1;i >= 0;--i) {
		if (2 * A[i] <= A[j]) {
			++cnt;
			--j;
		}
	}

	cout << N - cnt << endl;
}
{% endhighlight %}
