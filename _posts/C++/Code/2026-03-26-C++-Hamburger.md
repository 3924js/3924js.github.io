---
title: "C++: 햄버거 만들기"
author: Jaeseong Kim
date: 2026-03-26 12:00:00 +0800
categories: [C++, Code]
tags: [C++, Code, Stack]
---
https://school.programmers.co.kr/learn/courses/30/lessons/133502#

내일배움캠프 코딩테스트 연습목록중에 있던 연습문제인데, 딱 보기엔 괄호 짝 맞추기 문제랑 유사해보여서 스택을 이용해 접근해보았다. 괄호 짝맞추기 문제도 나는 스택말고 횟수와 깊이를 확인하는 방식으로 풀었어서, 평소 내가 익숙한 방식과는 다른 접근법이였다. 만약 스택에 대해 익숙하지 않았다면 괄호 문제와 비슷하게 깊이를 추적하는 문제로 갔을 것이다. 근데 푼 다음에 생각해보면 그게 더 어려운 접근법이였을 것으로 보이기도 한다.
## 조건
* 햄버거 재료가 빵, 야채, 고기가 각각 1,2,3이란 번호로 ingredient 벡터 안에 주어진다.
* 빵, 야채, 고기, 빵의 순서로, 그러니까 1,2,3,1순서가 있으면 햄버거로 조합할 수 있다.
* 햄버거로 조합해 빼낸 후 남은 재료가 다시 순서가 맞는 경우, 해당 부분도 다시 햄버거로 조합할 수 있다.
	* 1,2,1,2,3,1 , 3 ,1 -> 1,2,(1,2,3,1),3,1 -> 1, 2, 3, 1 -> 총 2개
* 만들 수 있는 햄버거의 총 갯수를 구해야 한다.
## 풀이
앞부터 재료들을 순회하면서 스택에 재료를 쌓는다.
* 빵(1)이면 쌓거나 조합한다.
    * 스택 맨 위가 고기(3)이면 3번 pop한다. 즉 햄버거를 조합한다. 이에 따라 answer를 1 올린다.
    * 아닐경우 일단 스택에 쌓는다. 언제든 1은 재료가 될 수 있다.
* 다른 재료(2,3)이면 아래를 확인하고 결정한다.
    * 1 차이라면, 즉 순서에 맞는 재료면 스택에 쌓는다.
    * 아니면 스택을 초기화한다. 중간에 잘못된 재료가 끼어있으면, 그 전까지 있던 재료는 스택 윗재료가 빠져 접근이 열리는 상황이 없기에 쓸모가 없다.
* 끝까지 반복한 후 answer를 반환한다.

```c++
#include <string>
#include <vector>
#include <stack>

using namespace std;

int solution(vector<int> ingredient) {
    int answer = 0;
    stack<int> burger;
    for(auto i = ingredient.begin(); i != ingredient.end(); i++){
        // 1
        if(*i == 1){
            //안비어있고 맨위가 3일때
            if(!burger.empty() && burger.top() == 3){
                burger.pop();
                burger.pop();
                burger.pop();
                answer++;
            }
            //아니면 일단 추가
            else{
                burger.push(*i);
            }
        }
        // 2 혹은 3
        else if(!burger.empty() && burger.top() == *i -1){
            burger.push(*i);
        }
        else{
            burger = stack<int>();
        }
    }
    return answer;
}
```