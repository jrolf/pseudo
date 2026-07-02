# Example 7: Dijkstra's Algorithm in C++

Graph algorithms in C++ bury their ideas under types. This is a standard
Dijkstra implementation with a binary-heap frontier, including the one weird
line every reader trips over.

## The original

```cpp
#include <queue>
#include <vector>
#include <limits>

using Edge = std::pair<int, long long>;  // (neighbor, weight)

std::vector<long long> dijkstra(
    const std::vector<std::vector<Edge>>& graph, int source) {
  const long long INF = std::numeric_limits<long long>::max();
  std::vector<long long> dist(graph.size(), INF);
  std::priority_queue<std::pair<long long, int>,
                      std::vector<std::pair<long long, int>>,
                      std::greater<>> frontier;

  dist[source] = 0;
  frontier.push({0, source});

  while (!frontier.empty()) {
    auto [cost, node] = frontier.top();
    frontier.pop();
    if (cost > dist[node]) continue;

    for (auto [next, weight] : graph[node]) {
      long long candidate = cost + weight;
      if (candidate < dist[next]) {
        dist[next] = candidate;
        frontier.push({candidate, next});
      }
    }
  }
  return dist;
}
```

## The Pseudo translation

```python
# DIJKSTRA: CHEAPEST COST FROM ONE PLACE TO EVERY OTHER PLACE

Define "dijkstra", given [graph, source]:

    Record the best known cost to reach every place:
        Start every place at "unreachable", except the starting place
        at zero.

    Keep a frontier of places waiting to be explored,
    always handing back the cheapest one first.
    Put the starting place on the frontier at cost zero.

    While the frontier is not empty:
        Take the cheapest waiting entry off the frontier.

        # The frontier cannot update an entry already inside it, so when
        # a place's cost improves, a duplicate cheaper entry is added
        # instead. The old, worse entry still surfaces eventually - and
        # this check is where it gets recognized and thrown away.
        If this entry is worse than the best known cost for its place:
            Skip it; it is stale.

        For each road leaving this place:
            Work out the cost of reaching the neighbor through this place.
            If that beats the neighbor's best known cost:
                Record the improvement.
                Add the neighbor to the frontier at the improved cost.

    Return the table of best known costs.
```

## What the translation reveals

The line `if (cost > dist[node]) continue;` looks like defensive noise, and
deleting it still passes most casual tests. It is actually a deliberate
design decision called lazy deletion - the priority queue cannot decrease a
key, so the algorithm tolerates duplicates and discards stale ones on
arrival. The Pseudo dedicates its longest voiceover to exactly that line,
which is precisely proportional to how often it confuses readers of the C++.
