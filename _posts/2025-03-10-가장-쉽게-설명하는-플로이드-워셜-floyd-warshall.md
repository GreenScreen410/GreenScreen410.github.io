---
layout: post
title: 가장 쉽게 설명하는 - 플로이드-워셜 (Floyd-Warshall)
date: 2025-03-10 12:51 +0900
categories: [Algorithm, Graph]
math: true
---

## 개요

플로이드-워셜(Floyd-Warshall)은 **모든 정점에서** 모든 정점으로의 최단 거리를 구하는 알고리즘으로, $O(N^3)$의 시간 복잡도를 가지고 있습니다.
다익스트라 알고리즘은 가중치가 음수인 경우에는 사용할 수 없다는 단점이 있지만, 플로이드-워셜은 사용할 수 있습니다.

> 음수 사이클이 있는 그래프의 경우 사용할 수 없습니다.
{: .prompt-danger }

## 과정

```cpp
for (int k = 0; k < N; k++) {
  for (int i = 0; i < N; i++) {
    for (int j = 0; j < N; j++) {
      graph[i][j] = min(graph[i][j], graph[i][k] + graph[k][j]);
    }
  }
}
```
`k`는 경유지, `i`는 출발 정점, `j`는 도착 정점입니다.  
**그냥 `i` 정점에서 `j` 정점으로 가는 것보다, `k` 정점을 경유해서 가는 비용이 더 적다면 해당 값으로 연결 정보를 업데이트하여 줍니다.**  
위 과정이 플로이드-워셜의 핵심 과정이며, 전부입니다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbsEp9X%2FbtsJMiO5KHt%2FKA8nqaK3meCKRdJnDM0MfK%2Fimg.png)
_사진 출처: <https://sam0308.tistory.com/130>_
자기 자신과의 거리는 0으로 초기화하고, 연결 정보를 기록합니다. 나머지 칸은 아직 거리를 알 수 없으므로 ∞이라는 아주 큰 값으로 초기화 해줍니다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FnuRnK%2FbtsJMrrJl1K%2F8NhuX4XMKIvms103tiLDpk%2Fimg.png)
_사진 출처: <https://sam0308.tistory.com/130>_

플로이드-워셜의 핵심은 **정점 경유**입니다.

**경유하는 정점을 기준으로** 탐색을 시작해 줍니다. 0번 정점을 경유해서 만들 수 있는 경로는 1번 정점 → 2번 정점, 2번 정점 → 1번 정점입니다.  
위 인접 행렬에서는 1번 정점에서 2번 정점으로, 2번 정점에서 1번 정점으로 갈 수 있는 경로가 ∞이라고 되어 있었는데, 0번 정점을 경유하여 가면 8이 되므로 업데이트 하여 줍니다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbJ9vuF%2FbtsJLy6tVtC%2Fp8hXQI9IcU7ThNK102angK%2Fimg.png)
_사진 출처: <https://sam0308.tistory.com/130>_

위 과정을 모든 정점에 대해 진행하면 이러한 인접 행렬을 얻을 수 있습니다.  
여기서 눈여겨 볼만한 점은 0번 정점에서 4번 정점으로 가는 경로인데, 1번 정점을 경유하였을 때에는 17이였지만, 2번 정점을 경유하였을 때는 7로 다시 업데이트 되었습니다.  
이렇게 모든 정점을 탐색하기 때문에, 항상 최소 비용을 갖는 경로를 찾아낼 수 있습니다.

> ∞을 초기화 할 때 단순히 `numeric_limits`와 같은 것을 사용한다면 경유하는 비용을 계산할 때 **오버플로우가 발생할 수 있습니다.**  
> 또 너무 작은 값을 주어서도 안됩니다.
{: .prompt-danger }

## 해결할 수 있는 문제

### [BOJ 11403 - 경로 찾기](<https://www.acmicpc.net/problem/11403>)

별다른 가중치 없이 정점의 연결 정보만 입력됩니다. 즉, 그저 두 정점이 서로 연결될 수 있는지만 확인하면 됩니다.  
연결될 수 있는지 판단한다는 것은, 즉 **경유할 수 있는 정점이 있는지 판단하는 것과 동일하기 때문에** 단순히 `G[i][k]`, `G[k][j]`가 서로 1인지만 확인하면 됩니다.

```cpp
#include <iostream>
#include <vector>
using namespace std;

int main() {
  int N;
  cin >> N;
  
  vector<vector<int>> G(N, vector<int>(N));
  for (int i = 0; i < N; i++) {
    for (int j = 0; j < N; j++) {
      cin >> G[i][j];
    }
  }
  
  for (int k = 0; k < N; k++) {
    for (int i = 0; i < N; i++) {
      for (int j = 0; j < N; j++) {
        if (G[i][k] && G[k][j]) G[i][j] = 1;
      }
    }
  }
  
  for (int i = 0; i < N; i++) {
    for (int j = 0; j < N; j++) {
      cout << G[i][j] << " ";
    }
    cout << "\n";
  }
  
  return 0;
}
```

### [BOJ 11404 - 플로이드](<https://www.acmicpc.net/problem/11404>)

가장 기본적인 플로이드-워셜 문제입니다.

```cpp
#include <climits>
#include <iostream>
#include <vector>
using namespace std;

auto floyd_warshall(auto& graph) {
  for (int k = 1; k < graph.size(); k++) {
    for (int i = 1; i < graph.size(); i++) {
      for (int j = 1; j < graph.size(); j++) {
        graph[i][j] = min(graph[i][j], graph[i][k] + graph[k][j]);
      }
    }
  }
  return graph;
}

int main() {
  int n;
  cin >> n;
  
  int m;
  cin >> m;
  
  vector<vector<long long>> graph(n + 1, vector<long long>(n + 1, INT_MAX));
  for (int i = 1; i <= n; i++) {
    graph[i][i] = 0;
  }
  
  while (m--) {
    long long a, b, c;
    cin >> a >> b >> c;
    graph[a][b] = min(graph[a][b], c);
  }
  
  auto result = floyd_warshall(graph);
  for (int i = 1; i <= n; i++) {
    for (int j = 1; j <= n; j++) {
      cout << (result[i][j] == INT_MAX ? 0 : result[i][j]) << " ";
    }
    cout << "\n";
  }
  
  return 0;
}
```
