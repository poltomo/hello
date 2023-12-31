---
layout: post
title:  "leetcode 1444. number of ways of cutting a pizza"
date:   2023-09-13 15:06:34 -0700
# categories:
# permalink:
---

Problem: [number of ways to cut pizza](https://leetcode.com/problems/number-of-ways-of-cutting-a-pizza/description/)

Leetcode Solution page: [my solution](https://leetcode.com/problems/number-of-ways-of-cutting-a-pizza/solutions/4045890/thoroughly-explained-solution/)

# Problem
The problem asks to find the number of ways to cut the pizza k times such that each piece has at least one apple.

# Note
This problem cannot be solved with combinatorics.
The top half of horizontal cuts is given to a person.
The left half of vertical cuts is given to a person.

# Intution
The problem is fit for dynammic programming.

Say you make a vertical cut; the number of ways to cut the pizza k times given you made a vertical cut, is simply the the number of ways to cut the remaining pizza k-1 times.

Also, notice that you can represent the remaining pizza by the top left position.

This leads me to the dp states: **dp[c][row][col]**, where c is the remaining cuts to make, row is the topmost row, and col is the leftmost column.

dp[c][row][col] = **number of ways to cut the pizza(row, col) k times such that each piece has at least one apple.**


# So what are the transitions from state to state?

Say you make the leftmost vertical cut on pizza(row, col). The pizza remaining will be pizza(row, col + 1).

Similarly, the topmost horizontal cut yields pizza(row + 1, col). And, we already made clear that the cuts remaining will be c - 1.

**Remember, pizza(0, 0) is the starting pizza.**

So is this the recurrence?

dp[c][row][col] = (dp[c - 1][row + 1][col] + dp[c - 1][row + 2][col] + ... + dp[c - 1][N - 1][col]) + (dp[c - 1][row][col + 1] + dp[c - 1][row][col + 2] + ... + dp[c - 1][row][M - 1])

- N = total number of rows
- M = total number of columns

No! You can't just make these cuts all willy-nilly.

Look at the first term in the first summation: dp[c - 1][row + 1][col]. The only way you can factor that into the result is if the left half has at least one apple in it. How do you confirm that?

Basically, you can use a 2D prefix sum.

pfx[row][col] = number of apples in pizza(row, col) = pfx[row + 1][col] + pfx[row][col + 1] - pfx[row + 1][col + 1]


That means that dp[c - 1][row + 1][col] is included only if (pfx[row][col] - pfx[row + 1][col]) > 0, i.e. the left half of the vertical cut has at least 1 apple


The base case (c = 0 cuts to make) is quite simple:
if pfx[row][col] > 0, then dp[0][row][col] = 1. Otherwise, dp[0][row][col] = 0.


And, the recurrence is:

dp[c][row][col] = ((pfx[row][col] - pfx[row + 1][col] > 0) ? dp[c - 1][row + 1][col] : 0) + ... + ((pfx[N - 1][col] - pfx[N - 1][col] > 0) ? dp[c - 1][N - 1][col] : 0) +

((pfx[row][col] - pfx[row][col + 1] > 0) ? dp[c - 1][row][col + 1] : 0) + ... + ((pfx[row][M - 1] - pfx[N - 1][M - 1] > 0) ? dp[c - 1][row][M - 1] : 0)


Sure, I could write this way clearer in summation notation.


bottom up dynammic programming
{% highlight c++ %}
#define REP(i, n) for(int i = 0;i < n;++i)
class Solution {
public:
    int ways(vector<string>& P, int k) {
        int N = P.size();
        int M = P[0].size();

        const int MOD = 1e9 + 7;

        vector< vector<int> > A(N + 1, vector<int>(M + 1));

        vector< vector< vector<int> > > dp(k, vector< vector<int> >(N, vector<int>(M)));

        for (int row = N - 1;row >= 0;--row) {
            for (int col = M - 1;col >= 0;--col) {
                A[row][col] = (P[row][col] == 'A') + A[row + 1][col] + A[row][col + 1] - A[row + 1][col + 1];

                dp[0][row][col] = A[row][col] > 0;
            }
        }

        for (int i = 1;i < k;++i) {
            REP(row, N) {
                REP(col, M) {

                    for (int next_row = row + 1;next_row < N;++next_row) {
                        if (A[row][col] - A[next_row][col])
                            dp[i][row][col] = (dp[i][row][col] + dp[i - 1][next_row][col]) % MOD;
                    }

                    for (int next_col = col + 1;next_col < M;++next_col) {
                        if (A[row][col] - A[row][next_col])
                            dp[i][row][col] = (dp[i][row][col] + dp[i - 1][row][next_col]) % MOD;
                    }
                }
            }
        }

        return dp[k - 1][0][0];
    }
};
{% endhighlight %}

bottom up dynammic programming (optimized memory)
{% highlight c++ %}
#define REP(i, n) for(int i = 0;i < n;++i)
class Solution {
public:
    int ways(vector<string>& P, int k) {
        int N = P.size();
        int M = P[0].size();

        const int MOD = 1e9 + 7;

        vector< vector<int> > A(N + 1, vector<int>(M + 1));

        vector< vector<int> > dp(N, vector<int>(M));

        for (int row = N - 1;row >= 0;--row) {
            for (int col = M - 1;col >= 0;--col) {
                A[row][col] = (P[row][col] == 'A') + A[row + 1][col] + A[row][col + 1] - A[row + 1][col + 1];

                dp[row][col] = A[row][col] > 0;
            }
        }

        for (int i = 1;i < k;++i) {
            REP(row, N) {
                REP(col, M) {
                    dp[row][col] = 0;

                    for (int next_row = row + 1;next_row < N;++next_row) {
                        if (A[row][col] - A[next_row][col])
                            dp[row][col] = (dp[row][col] + dp[next_row][col]) % MOD;
                    }

                    for (int next_col = col + 1;next_col < M;++next_col) {
                        if (A[row][col] - A[row][next_col])
                            dp[row][col] = (dp[row][col] + dp[row][next_col]) % MOD;
                    }
                }
            }
        }

        return dp[0][0];
    }
};
{% endhighlight %}

top down dynammic programming
{% highlight c++ %}
const int MOD = 1e9 + 7;

class Solution {
    int dp(int c, int row, int col, int N, int M, vector< vector< vector<int> > >& vvvi, const vector< vector<int> >& A) const {
        if (c == 0) return (A[row][col] > 0);

        if (vvvi[c][row][col] != -1) return vvvi[c][row][col];

        int res = 0;
        for (int next_row = row + 1;next_row < N;++next_row) {
            if (A[row][col] - A[next_row][col])
                res = (res + dp(c - 1, next_row, col, N, M, vvvi, A)) % MOD;
            }

        for (int next_col = col + 1;next_col < M;++next_col) {
            if (A[row][col] - A[row][next_col])
                res = (res + dp(c - 1, row, next_col, N, M, vvvi, A)) % MOD;
        }

        vvvi[c][row][col] = res;
        return res;
    }
public:
    int ways(vector<string>& P, int k) {
        int N = P.size();
        int M = P[0].size();

        vector< vector<int> > A(N + 1, vector<int>(M + 1));

        vector< vector< vector<int> > > T(k, vector< vector<int> >(N, vector<int>(M, -1)));

        for (int row = N - 1;row >= 0;--row) {
            for (int col = M - 1;col >= 0;--col) {
                A[row][col] = (P[row][col] == 'A') + A[row + 1][col] + A[row][col + 1] - A[row + 1][col + 1];
            }
        }

        return dp(k - 1, 0, 0, N, M, T, A);
    }
};
{% endhighlight %}
