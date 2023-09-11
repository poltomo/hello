---
layout: post
title:  "leetcode 815. bus routes"
date:   2023-09-10 15:06:34 -0700
___

problem: [bus routes](https://leetcode.com/problems/bus-routes/description/)

I thought this was a pretty cool problem.


{% highlight c++ %}
#include<vector>
#include<algorithm>

using namespace std;

#define REP(i, n) for (int i = 0;i < n;++i)

class Solution {
public:
    int numBusesToDestination(vector< vector<int> >& routes, int source, int target) {
        if (source == target) return 0;

        int N = routes.size();

        REP(i, N) {
            sort(routes[i].begin(), routes[i].end());
        }

        queue< pair<int, int> > q;
        vector<int> seen(N);

	// add all bus routes that contain source
        REP(i, N) {
            auto it1 = lower_bound(routes[i].begin(), routes[i].end(), source);

            if (it1 != routes[i].end() && *it1 == source) {                
                q.push({i, 0});
                seen[i] = 1;
            }
        }
        
        while (!q.empty()) {
            pair<int, int> bus = q.front();
            q.pop();

            auto it = lower_bound(routes[bus.first].begin(), routes[bus.first].end(), target);

            if (it != routes[bus.first].end() && *it == target) {
                return bus.second + 1;
            }

            REP(i, N) {
                if (seen[i]) continue;
                vector<int> I;
                set_intersection(routes[bus.first].begin(), routes[bus.first].end(), routes[i].begin(), routes[i].end(), back_inserter(I));
                if (!I.empty()) {
                    q.push({i, bus.second + 1});
                    seen[i] = 1;
                }
            }
        }

        return -1;
    }
};
{% endhighlight %}

It seems intuitive to think of the bus routes themselves as edges because they connect stops to other stops. But in this case, the question focuses on the number of busses, not the number of stops between source and target.

When you realize that you can only go from using one route to another if they share stops, then its more intuitive to think of the routes as vertices, and the stops as edges. This way you can use breadth first search to find the shortest path.
