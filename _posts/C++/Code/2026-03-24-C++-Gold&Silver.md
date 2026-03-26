---
title: "C++: 금과은 옮기기"
author: Jaeseong Kim
date: 2026-03-20 12:00:00 +0800
categories: [C++, Code]
tags: [C++, Code, Binary Search]
---
https://school.programmers.co.kr/learn/courses/30/lessons/86053
추천 문제로 떠서 시도해봤는데 나에게는 꽤나 어려운 문제였다. 탐색 문제로 접근해야되는데, 브루털 포스 기법밖에 생각이 안났기 때문이였다. 조건은 다음과 같다.
## 조건
* 새로운 도시 건설에 필요한 금과 은의 양이 a, b로 주어진다.
* 각 도시가 보유한 금과 은의 양은 g[], s[]로 주어진다.
* 각 도시에서 새로운 도시 건설현장까지 편도로 t[]만큼 걸리며, 한번에 w[]만큼만 트럭으로 운반할 수 있다.
* 새 도시 건설에 필요한 금과 은을 옮기는데 필요한 최소 시간을 구해야한다.
문제만 읽어봤을 때는 수식의 형태로 변환이 가능하지 않을까 싶었는데 경우가 너무 복잡해보였다. 자원이 2개로 나뉘었고, 도시도 얼마나 많고 또 각 도시가 얼마나 가지는지를 가지고 깔끔하게 수식 형태로 정리될 그림은 아니였다. 그래서 다음 방법에 대해 생각해보았다.
## 브루털 포스(Brutal Force)
**모든 경우의 수를 전부 계산하여 결과값을 찾아내는 알고리즘**의 분류이다. 경우의 수가 크지 않거나, 사용 가능한 자원과 시간이 여유가 있을 때 충분히 좋은 기법인데, 플로우를 짜는게 다른 알고리즘을 생각할 때보다 간단한 편이기에 간단화된 몇가지 테스트 케이스에만 가능성을 확인하는 정도로는 충분히 적용될 수 있다고 할 수 있다.
처음에 브루털 포스쪽 기법으로 들어가려고 했으나 머지않아 소용이 없음을 알게됬다. 도시의 수가 무한정 많아질 수 있고, 그 도시마다 가지는 변수값도 다 제각기 다르며, 어떻게 운반량을 분배할지 계산하는 행위가 너무 시간이 오래 걸릴 것이 훤히 보이는 문제이기 때문이다. 도시의 갯수가 10개를 넘기지 않는다던가, 모든 도시의 이동시간이나 운반량이 동일하다던가 하면 보다 고려할 법한 상황이겠지만, 일단은 현재 문제를 푸는데는 부적합한 접근이라 할 수 있다.
## 이진 탐색(Binary Search)
브루털 포스가 모든 경우의 수를 계산하는 것과는 다르게, 탐색 가능한 경우의 수를 좁히며 **모든 경우를 다 확인하지 않고 답을 찾아내는 방법**을 **탐색(Search)**라고 한다. 그중에서도 **이진 탐색(Binary Search)**은 한번쯤은 해보았던 업다운 게임과 유사하다. 찾고자 하는 값이 현재 확인하고 있는 값보다 작냐 크냐를 기준으로 확인할 범위를 빠르게 줄여나가는 것이다. 물론 줄여나가는 속도나 방법은 상황과 문제에 따라 다르겠지만, 기본적인 개념 자체인 "**범위를 줄인다**"는 것은 동일하다.
## 범위가 없어도 이진탐색 가능?
그렇다면 지금같은 문제의 범위가 없는 문제에도 과연 이진탐색이 적용 가능할까? 범위가 없으면 만들면 된다. 옮길 수 있는 최소한의 시간을 찾는 문제인 것을 비틀어서, 특정한 시간이 주어졌을 때 필요한 만큼 옮길 수 있는지에 대한 질문으로 바꾸면 범위를 만들 수 있다. 하한선은 0부터 시작하면 될 것이고(더 높은 수에서 시작해도 좋겠지만 에러에 취약할 수도 있고 구조가 상대적으로 불안하다.) 상한선은 넓게 몇배씩 값을 높여가며 가능함이 확인된 값으로 정하면 충분하다. 지금처럼 시간 하나만이 찾는 기준이 아닌 경우에는 이진탐색이 적용 불가능하지만, 지금은 이정도로 충분할것이다.
## 처음엔 잘못된 시도
처음 시도했던 구조는 다음과 같다. 주어진 시간을 상정하고, 해당 시간 안에 금과 은을 옮길 수 있는지 확인한다. 일단 주어진 시간안에 옮길 수 있는 총 용량을 계산해 a+b보다 많은지 확인한다. 그 후, 각 도시에서 보낼 수 있는 금과 은을 확인하는데, 한쪽 자원만 있는 도시는 해당 자원만 금 혹은 은 총량 합산에 넣고, 둘 다 가지고 있는 경우엔 각각을 별도의 금/은 합산에 넣는다. 모든 도시에서 확인이 끝나면 
```c++
#include <string>
#include <vector>

using namespace std;
bool CanMoveEnough(int RequiredTime, int a, int b, vector<int> g, vector<int> s, vector<int> w, vector<int> t){
    //시간안에 움직이는 전체용량이 필요량을 넘기는지 확인
    long long totalWeight = 0;
    for(int i = 0; i < w.size() ; i++){
        totalWeight += (RequiredTime / (2 * t[i]) + (RequiredTime % (2 * t[i])) / t[i]) * w[i]; 
    }  
    //넘기면 금 은 분배
    if(totalWeight >= a + b){
        long long movedGold = 0;
        long long movedSilver = 0;
        long long RemainGold = 0;
        long long RemainSilver = 0;
        for(int i = 0; i < g.size(); i++){
            if(g[i] != 0 && s[i] == 0){ //금만 있을때
                movedGold += min(g[i], (RequiredTime / (2 * t[i]) + (RequiredTime % (2 * t[i])) / t[i]) * w[i]);
            }
            else if(g[i] == 0 && s[i] != 0){ //은만 있을 때
                movedSilver += min(s[i], (RequiredTime / (2 * t[i]) + (RequiredTime % (2 * t[i])) / t[i]) * w[i]);
            }
            else{//둘다 있을 때
                RemainGold += min(g[i], (RequiredTime / (2 * t[i]) + (RequiredTime % (2 * t[i])) / t[i]) * w[i]);
                RemainSilver += min(s[i], (RequiredTime / (2 * t[i]) + (RequiredTime % (2 * t[i])) / t[i]) * w[i]);
            }
        }
        //필요량 충족 가능한지 계산
        if(a <= movedGold + RemainGold && b <= movedSilver + RemainSilver){
            return true;
        }
        else{
            return false;
        }
    }
    else{
        return false;
    }
}

long long solution(int a, int b, vector<int> g, vector<int> s, vector<int> w, vector<int> t) {
    int RequiredTime = 1;
    //옮길 수 있는 시간 값 찾기
    while(true){
        RequiredTime *= 2;
        if(CanMoveEnough(RequiredTime,a,b,g,s,w,t)){
            break;
        } 
    }
    //찾은값과 이전 값 사이에서 이진탐색
    int Dist = RequiredTime / 4;    //타겟 값의 거리
    int Change = Dist / 2;
    while(Change != 1){
        if(CanMoveEnough(RequiredTime - Dist,a,b,g,s,w,t)){
            Dist += Change;
        } 
        else{
            Dist -= Change;
        }
        Change /= 2;
    }
    if(CanMoveEnough(RequiredTime - Dist,a,b,g,s,w,t)){
        return RequiredTime - Dist;
    } 
    else{
        return RequiredTime - Dist + 1;
    }

}
```
기본 제공되는 테스트 케이스는 통과 했지만, 제출 후에는 거의 대부분을 틀리거나 시간 초과해버렸다. 생각을 해봤을때 몇가지 문제가 있는 것 같다.
1. 불필요하게 반복문이 많다.  금과 은 총량을 먼저 확인한 후에 각 도시의 금과 은을 각각 확인하기 위해 다시 순회하는데, 한번의 순회에서 전부 확인하면 순회 시간을 더 줄일 수 있을 것이다.
2. 금과 은을 다 가지고 있는 도시에 대해서 RemainGold 와 RemainSilver라는 개별의 변수들에 값을 저장해 확인하고있다. 금과 은은 보유량에 비해 운송 가능량, 혹은 금과 은의 보유 비율에 따라 둘 다 가진 도시에서 운송하는 것에 영향을 받지 않을까 해서 나눴던 것인데, 틀린 후에 다시 확인해보니 그럴 필요는 없는 것 같다.
3. 이진탐색의 방식이 생각보다 비효율적이다. 걸리는 시간이 생각보다 많이 걸리는데, 상한값을 찾기위한 과정과 다시 하한값을 찾기위한 과정을 2번으로 나누는 것보단 위아래로 가능한 최대범위를 설정하고 절반씩 가르는게 맞는 것 같다.
## 다시 이해하고 재시도
새로이 구성한 코드는 훨씬 간결해지고 쉬워졌다.
1. 일단 운반 가능량을 계산하는 반복문을 다시 짰다. 기존에는 반복문으로 확인 후 다른 반복문을 확인했다면, 지금은 어차피 한조건만 틀어져도 안된다는 것에 근거해 한 반복문 안에서 모든 도시를 확인해 운반 가능량을 계산한다. 금과 은, 그리고 운반 가능한 총량이 모두 필요한 양보다 많다면 그 시간은 가능한 시간값이다.
2. 상한, 하한을 찾기위해 처음에 계산해나가는 과정을 아예 생략하고, 작은 값인 0, 어느정도 큰 임의의 값인 $2^15$에서 값이 통과 가능한지 확인하고 범위를 좁혀나간다. 상한과 하한이 역전되는 그 지점이 바로 정답값이다.
```c++
#include <string>
#include <vector>
#include <climits>

using namespace std;
bool CanMoveEnough(long long RequiredTime, int a, int b, vector<int> g, vector<int> s, vector<int> w, vector<int> t){
    //시간안에 움직이는 양 계산
    long long TotalGold = 0;
    long long TotalSilver = 0;
    long long TotalWeight = 0;
    for(int i = 0; i < w.size() ; i++){
        long long MovableWeight = (RequiredTime / (2 * t[i]) + (RequiredTime % (2 * t[i])) / t[i]) * w[i];
        long long MovableGold = min(MovableWeight, static_cast<long long>(g[i]));
        long long MovableSilver = min(MovableWeight, static_cast<long long>(s[i]));
        TotalGold += MovableGold;
        TotalSilver += MovableSilver;
        TotalWeight += min(MovableGold + MovableSilver , MovableWeight);
    }
    //조건 만족하는지 확인
    if(TotalGold >= a && TotalSilver >= b && TotalWeight >= static_cast<long long>(a)+b){
        return true;
    }
    else{
        return false;
    }
}
long long solution(int a, int b, vector<int> g, vector<int> s, vector<int> w, vector<int> t) {
    long long low = 0;
    long long high = 2e15;
    while(low < high){
        long long middle = (low + high) / 2;
        if(CanMoveEnough(middle,a,b,g,s,w,t))high = middle;
        else low = middle + 1;
    }
    return low;
}
```
문제에 대한 이해도에서 결국 차이났던게 아닌가 싶다. 처음에 오히려 어렵게 접근해서 많이 돌아갔던 문제인 것 같다. 