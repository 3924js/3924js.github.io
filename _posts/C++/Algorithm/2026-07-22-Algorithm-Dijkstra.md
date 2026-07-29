---
title: "C++: 다익스트라 알고리즘(Dijkstra algorithm)"
author: Jaeseong Kim
date: 2026-07-22 8:00:00 +0800
categories: [C++, Algorithm]
tags: [C++, Algorithm, Dijkstra]
---
## 다익스트라 알고리즘(Dijkstra Algorithm)
![Graph](/assets/img/260722-graph.png)
* 가중치가 있는 그래프에서 사용 가능한 최소 거리 찾기 알고리즘으로, 주어진 시작 정점에서 다른 정점들로 가중치 합이 가장 적은 경로를 탐색한다.
* BFS에 큐 대신 우선순위 큐(Priority Queue)를 사용한 형태이다. 즉, 다익스트라에서 주어진 그래프의 모든 가중치가 동일하다면 BFS와 개념적으론 동일하게 작동한다.
* V개의 정점과 E개의 간선을 가지는 그래프에 대해 $O((V+E)logV)$의 시간복잡도를 가진다.
	* (V개의 정점 확인 + E개의 간선 확인) x 우선순위 큐 사용에 따른 V개 삽입/삭제 연산
* 사용하기 위해선 모든 가중치는 0 이상으로 음수 가중치가 없어야한다.

## 과정
처음에는 초기화해주는 과정이 필요하다.
* 시작하는 정점은 거리를 0으로, 다른 거리는 무한대(혹은 아주 큰 값)/미확정 으로 설정해놓는다.
* 정점까지의 거리를 저장할 다른 자료구조를 하나 선언한다.
* 시작할 정점과 누적거리 0을 우선순위 큐에 삽입한다. 이 때 해당 정점까지의 누적거리는 자료구조에도 기록해놓는다.
이후론 크게 2가지 단계를 반복하는 구조로 이루어져 있다. 보면 그리디 알고리즘의 형태이다.
* 확정(Finalization)
	* 누적 거리가 가장 짧은 정점의 거리가 맞는지 확인한다.
		* 우선순위 큐에서 pop하면 자료구조에 의해 누적거리가 최소인 정점이 튀어나온다.
		* 이때 튀어나온 노드의 누적 거리가 별도로 저장된 누적 거리와 다르면 무시한다. 오래된, 더 먼 거리이니 의미가 없다.
		* 아니라면 확정으로 판단하고 갱신으로 진행한다.
	* 다익스트라 알고리즘의 사용조건인 0 이상의 가중치에 의해 이 때 확정된 거리값은 더 짧아질 수 없고, 최소값이 보장된다.
* 갱신(Relaxation)
	* 방금 확정된 정점과 인접한 이웃 정점에서 더 짧은 거리가 발견되면 최신화한다.
		* 최신화된 이웃 정점들은 새로운 누적 거리와 함께 우선순위 큐에 삽입한다.
		* 이때 더 오래된, 더 긴 누적 거리를 가진 정점은 확정 과정에서 더 먼 거리 조건에 의해 무시되니 괜찮다.
* 위 두 단계를 우선순위 큐가 빌때까지 반복한다.
* 구현 방식에 따라 누적거리의 기록을 큐에서 꺼내는 시점에 기록하는 방법, 우선순위 큐 없이 탐색을 이용하는 방식등 구조는 다양하게 구현 가능하다.

```c++
#include <functional>
#include <iostream>
#include <limits>
#include <queue>
#include <utility>
#include <vector>

//간선 표현용, (가중치, 도착 정점)
using Edge = std::pair<int, int>;

//다익스트라 실행
std::vector<int> Dijkstra(const std::vector<std::vector<Edge>>& graph,int start)
{
		//임의의 큰수 설
    const int INF = std::numeric_limits<int>::max();

    std::vector<int> distance(graph.size(), INF);

    // priority_queue는 기본적으로 큰 값을 먼저 꺼내므로 greater를 사용한다.
    // {누적 거리, 정점 번호}
    using Node = std::pair<int, int>;
    std::priority_queue<Node, std::vector<Node>, std::greater<Node>> pq;

    distance[start] = 0;
    pq.push({0, start});
		
		//우선순위 큐가 빌때까지 반복한다.
    while (!pq.empty())
    {
		    //1.확정
        const auto [currentDistance, current] = pq.top();
        pq.pop();

        // 이미 더 짧은 경로가 발견된 오래된 큐 항목은 무시한다.
        if (currentDistance != distance[current])
        {
            continue;
        }
				
				//2.갱신
        for (const auto& [weight, next] : graph[current])
        {
            const int newDistance = distance[current] + weight;
						
            if (newDistance < distance[next])
            {
                distance[next] = newDistance;
                pq.push({newDistance, next});
            }
        }
    }

    return distance;
}

int main()
{
     // 정점 번호
    // 0: A, 1: B, 2: C, 3: D, 4: E, 5: F, 6: G
		// 무방향 간선이므로 양쪽 정점에 모두 추가한다.
		
		//인접 리스트 그래프
    std::vector<std::vector<Edge>> graph(7);

    // A - B : 2
    graph[0].push_back({2, 1});
    graph[1].push_back({2, 0});

    // A - C : 5
    graph[0].push_back({5, 2});
    graph[2].push_back({5, 0});

    // B - C : 1
    graph[1].push_back({1, 2});
    graph[2].push_back({1, 1});

    // B - D : 3
    graph[1].push_back({3, 3});
    graph[3].push_back({3, 1});

    // C - E : 2
    graph[2].push_back({2, 4});
    graph[4].push_back({2, 2});

    // D - E : 2
    graph[3].push_back({2, 4});
    graph[4].push_back({2, 3});

    // D - F : 2
    graph[3].push_back({2, 5});
    graph[5].push_back({2, 3});

    // E - G : 4
    graph[4].push_back({4, 6});
    graph[6].push_back({4, 4});

    // F - G : 1
    graph[5].push_back({1, 6});
    graph[6].push_back({1, 5});

    const std::vector<int> distance = Dijkstra(graph, 0);

    const std::vector<char> vertexNames = {
        'A', 'B', 'C', 'D', 'E', 'F', 'G'
    };

    for (int vertex = 0;
         vertex < static_cast<int>(distance.size());
         ++vertex)
    {
        if (distance[vertex] == std::numeric_limits<int>::max())
        {
            std::cout << vertexNames[vertex] << ": 도달 불가\n";
        }
        else
        {
            std::cout << vertexNames[vertex]
                      << ": "
                      << distance[vertex]
                      << '\n';
        }
    }
    /*
    A: 0
    B: 2
    C: 3
    D: 5
    E: 5
    F: 7
    G: 8
    */
}
```

## 왜 음수 가중치는 안되나?

![negative](/assets/img/260722-negative.png)
* 갱신된 정점에서 연결된 다른 이웃 정점들의 실제 답인 경로와는 다를 수 있다. 즉 그리디 판단이 성립하지 않는다. (예시 기준 Z 결과값: 11, 실제 되야 하는 값: -86)
* 누적되는 거리가 감소할일이 없다면, 우선순위 큐에 의해서 뽑히는 가장 짧은 거리에서 접근 가능한 정점이 최소거리임을 보장할 수 있다.
* 이를 해결하고 싶으면 기존 다익스트라 알고리즘으론 안되고, 한번에 모든 간선을 매번 구하는 변형 알고리즘인 벨만-포드 알고리즘을 활용하자.

## 활용
* 게임
	* 이동 - RPG 자동이동, RTS 그룹 이동,
		* 격자형 맵/2차원 배열에서의 이동/탐색: 4개 간선 그래프에 알고리즘 적용으로 해결 가능.
		* 지형간 이동 - 가중치를 통한 이동비용 표현
		* NavMesh 내부 길찾기
		* 보스 추격
	* 게임 내 경로와 관련된 상황들에 적용 가능, 다만 많은 경우 A*로 성능 향상 가능.
* 웹서버
	* 인터넷 라우팅
		* Open Shortest Path First(OSPF), IS-IS(Intermediate System-to-Intermediate System)
		* 각 라우터가 정보를 모아 라우팅 테이블 구성 -> 홉 외로도 속도/지연까지 고려한 실제 최단 경로 탐색
	* Content Delivery Network/Content Distribution Network(CDN) 엣지 라우팅
	* 네비게이션
		* 지름길을 미리 압축하는 전처리로 탐색시간 감소
		* 양쪽에서 동시에 탐색하여 빠른 최단경로 탐색
		* 차량 호출 매칭, 경로 안내
	* 물류 배송 경로 최적화  
* CS
	* 회로 설계 신호 경로
	* 유전자/단백질 네트워크
	* 네트워크 플로우
		* 최소 비용 최대 유량(Minimum Cost Maximum Flow, MCMF) 알고리즘
	* 파생 길찾기/그래프 탐색 알고리즘
		* A*: 휴리스틱이 추가된 형태, 게임 길찾기 표준.
		* 벨만-포드: 음수 가중치 허용, 
		* Johnson's: 벨만-포드 + 다익스트라, 모든 쌍 최단 경로 파악.
		* CH(Contraction Hierarchies): 대규모 도로망 탐색용 최단거리 전처리.
		* 특수 케이스에 효율적인 다른 탐색 알고리즘들
	* 네트워크 구조의 경로와 관련된 상황들.
* 가중치 그래프에서 경로와 거리를 탐색해야 하는 상황의 시작점.