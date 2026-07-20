---
title: "C++: 깊이 우선 탐색(Depth-First Search, DFS), 너비 우선 탐색(Breadth First Search, BFS)"
author: Jaeseong Kim
date: 2026-07-15 8:00:00 +0800
categories: [C++, Algorithm]
tags: [C++, Algorithm, DFS, BFS]
---
## 그래프 탐색
* 한 정점이 여러 간선을 가질 수 있는 구조 탓에 원하는 경로를 깔끔하게 찾아내는 방법은 제한적이고, 대신 완전탐색 형태의 탐색법이 주를 이룬다. 그중 가장 기초로 기준 삼은 지점을 기준으로 멀리 떨어진 정도인 깊이(Depth)에 따라 멀리있는/깊이 있는 다른 정점을 향해 먼저 다가갈건지, 아니면 주변의 가까운/얕은 정점들을 먼저 탐색할건지에 따라 깊이 우선 탐색(Depth-First Search, DFS)과 너비 우선 탐색(Breadth First Search, BFS)가 있다. 
* 시작할 정점에서부터 확인하고 미방문한 이웃 정점을 자료구조에 넣는 방식이라는 기본은 같으며, 그자리에 LIFO/FIFO 구조중 무엇이 오는지에 따라 DFS/BFS가 갈린다.
    * 단, 무한루프에 빠지지 않기 위해 방문한 정점을 기억하고 재방문하지 않도록 해야한다.

## 깊이 우선 탐색(Depth-First Search, DFS)
![image](assets/img/260715-dfs.png)
* 한 길을 깊게 우선 확인한다. 막다른 정점에 도달하면 되돌아온다. (손짚고 미로 탐색과 같은 방식)
* 재귀함수를 이용해 손쉽게 구현할 수 있다.

```c++
#include <iostream>
#include <vector>
using namespace std;

//예시 그래프
vector<vector<int>> graph = {
    {1, 2},    // 0번 정점의 이웃
    {0, 3, 4},
    {0, 4},
    {1},
    {1, 2}
};
//방문 정점 체크용
vector<bool> visited(graph.size(), false);

//DFS 재귀함수
void DfsRecursive(int current)
{
    visited[current] = true;
    cout << current << ' ';

    for (int next : graph[current])
    {
        if (!visited[next])
            DfsRecursive(next);
    }
}

int main()
{
    DfsRecursive(0); // 0 1 3 4 2
}
```

* LIFO 방식이기에 명시적 스택을 이용한 형태도 가능하다. 스택 오버플로우가 날 정도로 큰 그래프를 탐색해야 할 때 대안으로 쓸 수 있다.

```c++
#include <iostream>
#include <stack>
#include <vector>
using namespace std;

//예시 그래프
vector<vector<int>> graph = {
    {1, 2}, {0, 3, 4}, {0, 4}, {1}, {1, 2}
};

//스택 기반 DFS
void DfsStack(int start)
{
    vector<bool> visited(graph.size(), false);
    stack<int> st;
    st.push(start);
    visited[start] = true;

    //스택 비어있을 때까지 반복, 확인(팝)하고 이웃 정점 푸쉬
    while (!st.empty())
    {
        int current = st.top();
        st.pop();
        cout << current << ' ';

        for (int next : graph[current])
        {
            if (!visited[next])
            {
                visited[next] = true;
                st.push(next);
            }
        }
    }
}

int main()
{
    DfsStack(0); // 이웃 정점 삽입 순서에 따라 방문 순서 다를 수 있음.
}
```

## 너비 우선 탐색(Breadth First Search, BFS)

![image](assets/img/260715-bfs.png)

* 가까운 정점을 우선적으로 탐색한다. (정점을 기준으로 파동처럼 퍼지는 구조)
* FIFO 방식이어서 큐를 이용하면 손쉽게 처리할 수 있다.

```c++
#include <iostream>
#include <queue>
#include <vector>
using namespace std;

//예시 그래프
vector<vector<int>> graph = {
    {1, 2}, {0, 3, 4}, {0, 4}, {1}, {1, 2}
};

//큐 기반 BFS
void Bfs(int start)
{
    vector<bool> visited(graph.size(), false);
    queue<int> q;
    q.push(start);
    visited[start] = true;

    //큐 빌때까지 반복, 확인(팝)하고 이웃 정점 푸쉬
    while (!q.empty())
    {
        int current = q.front();
        q.pop();
        cout << current << ' ';

        for (int next : graph[current])
        {
            if (!visited[next])
            {
                visited[next] = true;
                q.push(next);
            }
        }
    }
}

int main()
{
    Bfs(0); // 0 1 2 3 4
}

```
* 간선의 가중치가 모두 같거나 없을 때, 특정한 정점을 처음 만났을 때의 깊이가 곧 시작 정점으로부터 그 정점까지의 최단거리다.
## 활용
* 게임
    * 같은색 칠하기 문제(Floodfill)
    * 갈 수 있는 영역 찾기
    * 폭발 범위, 보스 추격
    * 미로 - DFS로 도달여부 확인, BFS로 최단거리 확인
* 웹서버
    * 네트워크 거리 측정: 홉(Hop) - 몇 번 라우터를 건너뛰면(Hop) 도달할 수 있는지 BFS로 거리 계산
    * SNS 친구 추천 - BFS로 친구의 친구 확인
    * 웹 크롤링 - 가까운 페이지부터 BFS
    * JSON/XML 트리 파싱 - DFS, 브래켓 깊은 속으로 들어가며 처리
* CS
    * 파일 탐색기 - 폴더 계층 구조
        * 용량 계산, 검색, 빌드 도구에서 import 순환 검출등 구조에 대한 탐색 -> DFS
    * 컴파일러 구문(Syntax) 트리 순회 - DFS, 브라켓을 통한 스코프 확인
    * 분산 시스템(Distributed System) 메시지 전파 - P2P 네트워크/ 병렬컴퓨팅
    * GNN(Graph Neural Network)의 이웃 수집
* 중요하고 무궁무진한 활용처, 그에 따른 기술면접 최빈출 유형 문제
