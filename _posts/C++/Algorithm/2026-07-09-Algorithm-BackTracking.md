---
title: "C++: 백트랙킹(Back Tracking)"
author: Jaeseong Kim
date: 2026-07-09 8:00:00 +0800
categories: [C++, Algorithm]
tags: [C++, Algorithm, Backtracking]
---
## 백트래킹(Backtracking)
![alt text](/assets/img/260709-backtracking.png) 
* 완전 탐색에서 방향에 대한 조건 확인과 포기가 들어간 알고리즘이다.
	* 조건에 맞지 않는다면 더이상 탐색하지 않아 탐색 시간을 줄일 수 있다.

## 구조 
1. 유망성 판단 (Promising)
	* 탐색하고 있는 방향이 찾고자 하는 답의 방향성에 부합하는지 확인한다.
2. 가지치기(Pruning)
	* 유망성 판단에서 실패한 부분에 대한 추가적인 탐색을 중단하고 **되돌아간다.**
	* 주어지는 자료가 정렬되어 있으면 가지치기의 효율성이 증가한다. 다만 항상 정렬이 필요한 것은 아니며, 정렬되어야만 백트래킹을 적용할 수 있다는 것도 아니다.
* 완전탐색의 선택-재귀-되돌림 패턴에서 선택-검증-재귀-되돌림으로 1개 단계만 추가되는 것.

## 예시문제
  * 부분 합 문제
  ![alt text](/assets/img/260709-subset_sum.png) 
	  * 배열에서 원소의 합이 특정 값이 되는 조합 찾기
	  * 1차원 배열 조합에 대한 가지치기

	 ```c++
	#include <vector>
	#include <iostream>
	using namespace std;

	void subsetSum(vector<int>& nums, int target, int idx, vector<int>& path, int total) {
	    if (total == target) {
	        for (int v : path) cout << v << " ";
	        cout << "\n";
	        return;
	    }
	    if (idx == (int)nums.size() || total > target) return;  // 가지치기

	    path.push_back(nums[idx]);
	    subsetSum(nums, target, idx + 1, path, total + nums[idx]);  // 선택함
	    path.pop_back();                                            // 되돌림
	    subsetSum(nums, target, idx + 1, path, total);               // 선택 안 함
	}
	```
  * N-Queen
  ![alt text](/assets/img/260709-n_queen.png)
	  * 체스판에 N개의 퀸이 서로의 경로가 겹치지 않게 배치할 수 있는 방법 찾기
	  * 2차원 배열 조합에 대한 가지치기

	 ```c++
	#include <vector>
	using namespace std;

	int n, count = 0;
	vector<bool> cols, diag1, diag2;  // diag1: row-col+n, diag2: row+col

	void solve(int row) {
	    if (row == n) { count++; return; }          // 완성된 배치 하나 발견

	    for (int col = 0; col < n; col++) {
	        if (cols[col] || diag1[row - col + n] || diag2[row + col]) continue;  // 유망성 판단 실패

	        cols[col] = diag1[row - col + n] = diag2[row + col] = true;
	        solve(row + 1);
	        cols[col] = diag1[row - col + n] = diag2[row + col] = false;  // 되돌림
	    }
	}
	```
	
## 활용
* 게임
	* Minimax 알고리즘 - 체스, 바둑 AI 기초
		* 알파-베타 가지치기(Alpha-Beta Pruning) - 턴 기반 AI 제작
	*	퍼즐 생성
		*	스도쿠
	*	레벨 생성
* 웹/서버
	* 제약 충족 문제(Constraint Satisfaction Problem, CSP)
		* N-Queen
		* 시간표 자동 생성
			* 스케쥴 자동 생성
			* 물류, 배차 계획
	* 정규식 매칭도(Regular Expression Matching)
		* 에디터 - 찾기
		* 쉘 - grep
		* 텍스트 검색
		* 웹 라우팅 매칭
* CS 전반
	* 논리식 충족(satisfiability problem, SAT)
	* 그래프 색칠 문제 - 인접노드는 다른색, 안되면 돌아가기
	* 컴파일러 - 정규식 매칭, 파서
	* 패스워드 크래킹 - 무차별 대입 업그레이드
	* 이외에도 있는 NP문제의 실전 적용 케이스들
* 완전 탐색이 필요는 하지만 범위를 좁혀나갈 수 있는 문제 전반.