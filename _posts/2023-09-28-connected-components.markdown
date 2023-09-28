---
layout: post
title:  "connected components"
date:   2023-09-28 15:06:34 -0700
# categories:
# permalink:
---

NOT FINISHED

[wikipedia](https://en.wikipedia.org/wiki/Component_(graph_theory))
---

[related problem 1](https://leetcode.com/problems/number-of-connected-components-in-an-undirected-graph/description/), 
[related problem 2](https://cses.fi/problemset/task/1192)


related problem 1 solution:
{% highlight c++ %}
class Solution {
public:
    int countComponents(int N, vector<vector<int>>& edges) {
        vector< vector<int> > AL(N);

        vector<int> seen(N);

        for (const vector<int>& e : edges) {
            AL[e[0]].push_back(e[1]);
            AL[e[1]].push_back(e[0]);
        }

        int cnt = 0;

        for (int i = 0;i < N;++i) {
            if (!seen[i]) {
                ++cnt;
                stack<int> s;
                s.push(i);
                seen[i] = 1;

                while (!s.empty()) {
                    int v = s.top();
                    s.pop();

                    for (int o : AL[v]) {
                        if (!seen[o]) s.push(o);
                        seen[o] = 1;
                    }
                }

            }
        }

        return cnt;
    }
};
{% endhighlight %}

related problem 2 solution:
{% highlight c++ %}
#include<iostream>
#include<vector>
#include<stack>
#include<string>

using namespace std;

#define REP(i,N) for (int i = 0;i < N;++i)

#define ll long long

bool inBounds(const pair<int, int>& p, int N, int M) {
	return p.first >= 0 && p.first < N && p.second >= 0 && p.second < M;
}

int main() {
	int N, M;
	cin >> N >> M;

	vector<string> A(N);

	REP(i, N) {
		cin >> A[i];
	}

	int dx[] = {-1, 1, 0, 0};
	int dy[] = {0, 0, -1, 1};

	stack<pair<int, int> > st;

	vector< vector<int> > seen(N, vector<int>(M));

	int out = 0;

	REP(i, N) {
		REP(j, M) {
			if (A[i][j] == '.') {
				st.push({i, j});
			}
		}
	}

	while (!st.empty()) {
		pair<int, int> p = st.top();
		st.pop();

		if (!seen[p.first][p.second]) {
			++out;
			seen[p.first][p.second] = 1;
		}

		REP(i, 4) {
			pair<int, int> next(p.first + dx[i], p.second + dy[i]);

			if (inBounds(next, N, M) && A[next.first][next.second] == '.' && !seen[next.first][next.second]) {
				seen[next.first][next.second] = 1;
				st.push(next);
			}
		}
	}

	cout << out << endl;
}
{% endhighlight %}

