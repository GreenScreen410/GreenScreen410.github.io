---
layout: post
title: "가장 쉽게 설명하는: 볼록 껍질 (Convex Hull)"
date: 2025-02-02 17:09 +0900
categories: [Algorithm, Geometry]
math: true
---

## 개요

![](https://blog.kakaocdn.net/dn/bsaK9V/btsHLux6zwj/Ns0MHqCYwcKAGctuPKoxEK/img.png){: .w-50 .center}
_(사진 출처: <https://david0506.tistory.com/62>)_

볼록 껍질(Convex Hull)은 점들이 주어졌을 때 **모든 점을 포함하는 볼록 다각형**을 구성하는 알고리즘으로, 주로 특정 상황에서의 최소 다각형을 구하는데 사용되곤 합니다.  
여기서 볼록 다각형이란, **모든 내각이 180도 이하로만 이루어진 다각형**을 말합니다.

볼록 껍질을 구하는데에는 다양한 알고리즘이 사용되지만, 대표적으로 **그레이엄 스캔**과 **앤드류 모노톤 체인**이 사용됩니다.

## 그레이엄 스캔 (Graham Scan)

```cpp
vector<Point> convex_hull(vector<Point> points) {
  sort(points.begin(), points.end(), [](Point a, Point b) {
    return make_pair(a.y, a.x) < make_pair(b.y, b.x);
  });
  
  Point pivot = points.front();
  sort(next(points.begin()), points.end(), [pivot](Point a, Point b) {
    if (ccw(pivot, a, b) > 0) return true;
    else if (ccw(pivot, a, b) < 0) return false;
    else return make_pair(a.y, a.x) < make_pair(b.y, b.x);
  });

  vector<Point> hull;
  for (auto p : points) {
    while (hull.size() >= 2 && ccw(hull[hull.size() - 2], hull.back(), p) <= 0) hull.pop_back();
    hull.push_back(p);
  }

  return hull;
}
```
그레이엄 스캔은 $O(N \log N)$의 시간 복잡도를 가지는 알고리즘으로, 스택을 사용하여 구현합니다.
> C++의 `vector`와 같이 **스택과 동일한 역할을 수행할 수 있는** 자료 구조를 사용해도 됩니다.
{: .prompt-tip }

### 반시계 방향으로 정렬하기

```cpp
sort(points.begin(), points.end(), [](Point a, Point b) {
  return make_pair(a.y, a.x) < make_pair(b.y, b.x);
});
```
먼저, y좌표가 작은 순서대로, y좌표가 같으면 x좌표가 작은 순서대로 점들을 정렬합니다.  
람다 함수와 `make_pair`를 통해 별도의 `compare` 함수를 생성하지 않고, **매우 직관적이게 한 줄로 정렬을 끝낼 수 있습니다.**  

```cpp
Point pivot = points.front();
sort(points.begin() + 1, points.end(), [pivot](Point a, Point b) {
  if (ccw(pivot, a, b) > 0) return true;
  else if (ccw(pivot, a, b) < 0) return false;
  else return make_pair(a.y, a.x) < make_pair(b.y, b.x);
});
```
좌표를 정렬하면, 자연스럽게 `points`의 첫 번째 원소(`points.front()`)인 `pivot`은 y좌표가 가장 낮은 점이 되는데, 이 `pivot`을 기준으로 CCW 값이 양수인 방향으로 점을 정렬합니다.  
만약 양수도, 음수도 아니라면(일직선상에 있다면) 좌표 순서대로 정렬합니다.

### 볼록 껍질 완성하기

```cpp
vector<Point> hull;
for (auto p : points) {
  while (hull.size() >= 2 && ccw(hull[hull.size() - 2], hull.back(), p) <= 0) hull.pop_back();
  hull.push_back(p);
}
```
볼록 껍질을 이루는 점들의 좌표를 반환할 `hull` 벡터를 생성한 뒤, 이어서 아래 과정을 수행합니다.

1. 처음에는 `hull` 벡터가 비어있기 때문에 while문이 실행되지 않고, `hull.push_back(p)`가 그대로 실행됩니다.
2. 점이 두 개 모인 후부터 while문이 실행됩니다. `hull`의 마지막 두 점(`hull[hull.size() - 2]`, `hull.back()`)과 현재 점 `p`를 확인합니다.
3. 점 `p`를 `hull`에 추가하기 전, 만약 이 세 점이 시계 방향, 또는 일직선상에 배치되어 있으면(즉, CCW 값이 0 이하일 경우), 마지막 점은 볼록 껍질에 포함될 수 없으므로 `pop_back()`으로 제거합니다. 이 과정을 통해 적합하지 않은 점 `p`를 모두 제거합니다.
    > [일부 문제](https://www.acmicpc.net/problem/3679)에서는 **일직선상에 있는 모든 점을** 볼록 껍질에 포함시켜야 할 수도 있습니다.  
    > ~~해당 문제는 엄밀히 말하자면 볼록 껍질 문제는 아닙니다.~~
    {: .prompt-danger }
4. 이 과정을 전체 점들에 대해 반복하면, 최종적으로 볼록 껍질을 이루는 점들만 순서대로 남게 됩니다.

```cpp
sort(points.begin(), points.end(), [](Point a, Point b) {
  return tie(a.y, a.x) < tie(b.y, b.x);
});
sort(points.begin() + 1, points.end(), [p = points.front()](Point a, Point b) {
  return ccw(p, a, b) > 0 || (ccw(p, a, b) == 0 && tie(a.y, a.x) < tie(b.y, b.x));
});
```
정렬하는 과정을 한 줄로 정리하면 이렇게 사용할 수도 있습니다. 코드가 상당히 짧아집니다.
