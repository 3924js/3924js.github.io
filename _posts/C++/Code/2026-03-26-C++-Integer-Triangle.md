---
title: "C++: 정수 삼각형"
author: Jaeseong Kim
date: 2026-03-27 12:00:00 +0800
categories: [C++, Code]
tags: [C++, Code, Dynamic Programming, Memoization, Tabulation]
---
https://school.programmers.co.kr/learn/courses/30/lessons/43105

학교에서 프로그래밍을 배울 때는 한참 효율성에 관심이 많았던 것 같다. 때문에 병렬 프로그래밍, 멀티 코어화, 최적화같은 기법에 관심이 많았는데 이후 배우는 많은 개념중 가장 먼저 배웠던 최적화 기법이였다. 오늘 추천된 문제는 우연히도 그 당시 배웠던 문제와 상당히 유사하다.
## 조건
* 삼각형의 형상으로 정수가 배열되어있다. 삼각형은 벡터의 형태로 주어진다.
	* 삼각형의 각 행은 이전 행보다 1이 더 긴 백터로 이루어져있다. 
* 삼각형 위쪽 끝에서 아래쪽으로 한칸씩 내려올때, 지나온 정수들의 합이 가장 클때의 값을 찾아야한다.
## 동적 계획법(Dynamic Programming)
동적 계획법은 계산중에 생기는 값들이 버려지지 않고, **기억**되어 다음 계산에 쓰이는 방식이다. 이 방식은 일일히 계산하면 매우 비효율적인 브루털 포스가 안좋은 접근인 문제나 수열의 계산같이 계산과정의 값이 결과 값을 얻는데 필수적인 문제에 적용될 수 있다. 구현방식은 크게 방식이 크게 메모이제이션과 타뷸레이션으로 나뉜다.
 **메모이제이션(Memoization)**은 함수의 재귀호출을 이용해 구현하는 방식으로, **함수를 재귀 호출**하면서  한번 계산한 값을 기억하도록 하는 기법이다. 재귀호출을 이용하기 때문에 스택 오버플로우의 위험은 있으나, 대게는 문제 구조 자체가 코드로 들어나니 더 직관적인 편이고, 필요한 만큼만 메모리가 할당되기에 반복이 많지 않은 작업에선 더 효율적일 수 있다. 함수를 결과값에 가까운 위에서 호출하는 것으로 시작해 작은 부분에 대한 탐색으로 이어지기에 **탑다운(Top-Down)** 방식으로도 불린다. 모든 경우의 수를 체크하지 않아도 되는 상황에선 메모이제이션이 아래 타뷸레이션보다 더 빠른 작동시간을 보장해준다.
**타뷸레이션(Tabulation)**은 그에 비해 재귀호출을 하지 않고, 미리 만들어놓은 저장공간에 계산과정을 기록하며 진행된다. 재귀함수 대신 **반복문**을 사용하고, 저장공간을 따로 할당하기에 힙 메모리에 필요한만큼 할당이 미리 이루어진다. 스택 오버플로우의 위험은 없지만 기본적으로 사용하는 메모리의 양은 더 많을 수 있다. 코드는 비직관적으로 보일 수 있지만, 동작의 속도는 타뷸레이션이 많은 계산량을 필요로 할땐 더 빠른 편이다. 맨 밑의 기저 조건부터 하나씩 계산해 올라가기에 **바텀업(Bottom-Up)**방식으로도 불린다. 모든 경우의 수를 확인해야 하는경우, 메모이제이션보다 타뷸레이션이 작동속도가 더 빠르다.
공통점은 **이전 계산의 결과를 그대로 이어받아 이후에 있을 계산에도 반영**한다는 것이다. 만약 그러지 않는다면, 그 해법은 브루트 포스(Brute Force)과 다를바가 없어 모든 경우를 계산하며 아닌 경우의수는 그냥 버리는 비효율적인 방식일 것이다.

## 풀이
이 경우에는 이미 삼각형이 수정가능하게 값복사된 매개변수로 주어졌기에 타뷸레이션처럼 접근하면 쉽게 풀린다.
* 삼각형 위쪽부터 아래쪽까지 아래 과정을 반복한다.
	* 바로 위에 있는 두 값중 큰값을 자신에게 더한다.
	* 왼쪽 혹은 오른쪽 모서리는 바로 위의 값을 자신에게 더한다.
* 끝나면 밑변에 있는 값들중 가장 큰 값이 나올 수 있는 가장 큰 정수의 합이다.

```c++
#include <string>
#include <vector>

using namespace std;

int solution(vector<vector<int>> triangle) {
    int answer = 0;
    for(auto i = triangle.begin() + 1; i != triangle.end(); i++){
        for(int j = 0; j < (*i).size(); j++){
            if(j == 0){  //첫번째는 이전층 첫번째 원소만큼 증가
                (*i)[0] += (*(i-1)).front();
            }
            else if (j == (*i).size() - 1){   //마지막은 이전층의 마지막 원소만큼 증가
                (*i)[j] += (*(i-1)).back();
            }
            else{   //가운데는 이전층 바로 위 2개 원소중 큰쪽만큼 증가
                (*i)[j] += max((*(i-1))[j - 1], (*(i-1))[j]);
            }
        }
    }
    //밑변에서 가장 큰 수 찾기
    const vector<int>& Bottom = triangle.back();
    for(auto i = Bottom.begin(); i != Bottom.end(); i++){
        if(answer < *i){
            answer = *i;
        }
    }
    return answer;
}
```

메모이제이션을 이용하면 아래처럼 캐와 재귀함수를 활용하는 방향으로 코드를 만들 수 있다. 다만 정답은 맞게 구하더라도 지금 문제에선 모든 경우를 확인해야하는 상황이라, 이 해법의 경우에도 효율성 테스트에서 몇몇 시간초과가 뜨는 예시들이 존재한다. 그래도 위에서 언급했듯 메모이제이션 쪽이 코드가 간결한건 사실이다.
지금 구조에서 캐시 없이 모든 경우의 수를 검사하는 구조가 되면 재귀함수를 이용한 브루트 포스 해법이되며 삼각형이 커질때 매우 많은 시간을 잡아먹게 된다.

```c++
#include <string>
#include <algorithm>
#include <vector>

using namespace std;
int RecursiveCall(const vector<vector<int>>& triangle, int depth, int index, vector<vector<int>>& memory){
    //밑변이면 자기 자신 반환(기저 조건)
    if(depth == triangle.size() - 1){
        return triangle[depth][index];
    }
    
    //계산했던 값이면 메모리에서 반환
    if(memory[depth][index] != -1) return memory[depth][index];
    
    //아니면 자기 자신과 아래 2개중 큰 값과의 합을 반환
    else{
        int value = triangle[depth][index] 
            + max(RecursiveCall(triangle, depth+1,index, memory),
                  RecursiveCall(triangle, depth+1,index+1, memory));
        memory[depth][index] = value;
        return value;
    }
}

int solution(vector<vector<int>> triangle) {
    vector<vector<int>> memory(triangle.size()-1, vector<int>(triangle.size()-1,-1));
    return RecursiveCall(triangle, 0,0, memory);
}
```