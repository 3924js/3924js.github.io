---
title: "C++: 완전 탐색(Brute Force)"
author: Jaeseong Kim
date: 2026-07-14 8:00:00 +0800
categories: [C++, Algorithm]
tags: [C++, Algorithm, Brute Force]
math: true
---
## 완전 탐색(Brute Force, Exhaustive Search)
* 무차별 대입(Brute Force)라는 암호학 용어에서 유래한 말로, 모든 경우의 수를 확인하는 방법을 뜻한다.
* 완벽한 정확성을 보장할 수 있지만, 시간이 오래걸려 큰 데이터나 자료를 탐색할때는 부적합할 수 있다.
    * 적용이 가능하지만 처리시간 탓에 쓰지 못할경우 백트래킹, 동적 프로그래밍등 다른 기법으로 선회 가능하다. 다른말로는 느리긴 하지만 가장 범용적이고 확실히 적용 가능한 기법이다.
* 정렬된 배열같은 단조성을 필요로 하는 이진 탐색과는 다르게, 구조가 없는 사실상 모든 경우에도 적용이 가능하다는 점에서 성격이 다르다.
* 재귀를 이용해 손쉽게 구현할 수 있다.

## 상황에 따른 적용
* 순열(Permutation)
    * 순서가 중요하다.
        * ABC와 CAB는 같지 않다. 순서가 다르면 다른 경우의 수이다.
    * 경우의 수 탓에 $O(n!)$이고 n = 13만 되어도 4.79억정도로 1억 회를 넘어서며 쓰기 어렵다.
    * 예: 4명 중 3명을 골라 1, 2, 3등 자리에 배치한다. `A, B, C`와 `C, B, A`는 다른 결과다.
    ![image](assets/img/260714-permutation.png)

    ```c++
    #include <iostream>
    #include <vector>
    using namespace std;

    vector<string> players = {"A", "B", "C", "D"};
    vector<bool> used(players.size(), false);
    vector<string> order;

    //순열 표시 함수, 재귀적으로 확인
    void MakePermutation(int count, int pick)
    {
        //기저 케이스
        if (count == pick)
        {
            for (const string& name : order)
                cout << name << ' ';
            cout << '\n';
            return;
        }

        //재귀 호출, 선택한 선수 재외한 배열과 함께 함수 반복
        for (int i = 0; i < static_cast<int>(players.size()); ++i)
        {
            if (used[i]) continue;

            used[i] = true;
            order.push_back(players[i]);
            MakePermutation(count + 1, pick);
            order.pop_back();
            used[i] = false;
        }
    }

    int main()
    {
        MakePermutation(0, 3); // 4P3 = 24가지
    }
    // `used` 배열로 이미 고른 사람을 다시 선택하지 않는다. 고르는 순서가 결과에 남으므로 순열이다.
    ```

* 조합(Combination) 
    * 순서가 필요 없다.
        * ABC와 CAB는 같다. 구성요소 종류만 같으면 된다.
    * 경우의 수 탓에 $O(2^n)$이고 n이 27만 되어도 1.3억으로 1억 회를 넘어서며 쓰기 어렵다.
    * 예: 5명 중 3명을 팀으로 고른다. `A, B, C`와 `C, B, A`는 같은 팀이다.
    ![image](assets/img/260714-combination.png)
    
    ```c++
    #include <iostream>
    #include <vector>
    using namespace std;

    vector<string> players = {"A", "B", "C", "D", "E"};
    vector<string> team;

    //조합 표시 함수
    void MakeCombination(int start, int count, int pick)
    {
        //기저 케이스
        if (count == pick)
        {
            for (const string& name : team)
                cout << name << ' ';
            cout << '\n';
            return;
        }

        //재귀 호출, 선택한 선수 제외 후 함수 반복
        for (int i = start; i < static_cast<int>(players.size()); ++i)
        {
            team.push_back(players[i]);
            MakeCombination(i + 1, count + 1, pick);
            team.pop_back();
        }
    }

    int main()
    {
        MakeCombination(0, 0, 3); // 5C3 = 10가지
    }
    //다음 탐색 시작 위치를 `i + 1`로 넘긴다. 앞에서 선택한 원소를 다시 고르거나 순서만 다른 결과를 만들지 않는다.
    ```

## 활용
* 게임
    * 최선의 수를 찾아야 하는 경우
        * ex) 체스, 장기등의 게임, 경우의 수 탓에 백트래킹으로 최적화 하는편.
    * 절차적 생성(Procedural Generation) - 모든 경우의 수 -> 검증 -> 선별
    * 퍼즐 게임 솔버 - 해법 자동 검증
    * 밸런싱 시뮬레이션 - 캐릭터/성능 조합의 시뮬레이션을 통한 승률 예측
* 웹서버
    * 조합을 찾아야하는 문제들
    * A/B 테스트 - 버튼/색상/디자인등을 비교하며 탐색
    * 설정 최적화 - 몇 안되는 설정을 빠르게 최적으로 적용
    * 광고 입찰 - 노출 빈도/시간 시뮬레이션
* CS
    * 코딩테스트의 출발점 - 적용가능한가? 바로 안되면 무엇으로 최적화 가능한가?
    * 보안/암호학 - 무차별 대입, 비밀번호 크래킹. 암호강도의 측정 기준.
    * 형식 검증(Formal verification) - 시스템이 형식/속성을 만족하는지 확인.
* 정확성이 무엇보다 중요한 경우, 표본이 충분히 작은 경우.
