---
title: "C++: 알고리즘 기초"
math: true
author: Jaeseong Kim
date: 2026-03-04 12:00:00 +0800
categories: [C++, Algorithm]
tags: [C++, Algorithm]
---
* ## 알고리즘(Algorithm)
	* 문제를 해결하기 위한 **절차**
		* 튜링머신이 모든 유효한 입력에 대해 **유한한 시간**에 **정지**하게 만들기 위한 명령
		* 페르시아의 수학자인 알콰리즈미(al-Khwārizmī)에서 유래
		* 여러 요소에 대해 좋은 알고리즘인지 평가하는 방법이 다름
* ## 복잡도(Complexity)
	* 알고리즘의 성능을 평가하는 기준. 무엇을 평가기준으로 정하는지에 따라 몇가지 방법이 있다.
	* ### 표기법
		* Big-O: O(n),  배수 부분을 제외했을때 n 이하의 속도로 증가한다.(상한선), 최악의 경우를 표기할 때 보통 사용
		* Big-Omega: Ω(n), 적어도 n이상의 속도로 증가한다.(하한선), 최선의 경우를 표기할 때 보통 사용
		* Theta: Θ(n), O와 Ω의 공통되는 영역, 평균적인 증가율으로 해석되는 경우가 많은데, 엄밀히 모든 경우에 있어 평균 수행 시간인 것은 아님.
		* 표기법 안에 들어가는 식이 증가율, n값의 증가에 따라 얼마나 수가 커지는 지를 나타내다. 복잡도에서는 이를 통해 얼마나 처리하는데 오래걸리는지, 공간을 많이 차지하는지등 안좋은 상태인지를 나타낸다.
			* $O(1) -> O(log n) -> O(n) -> O(n^k) -> O(e^n) < ...$
				* 1: 상수(constance)
					* 단순히 1번 하는 함수들, hello world 1번 출력
				* $log n$: 로그(Log)
					* 퀵 정렬 알고리즘(Quick sort), 이진트리 탐색
				* $n$: 선형(Linear)
					* n번 더하기, n개 짜리 배열 탐색
				* $n^k$: 다항(Polynomial)
					* ex) 이중 배열 탐색: $n^2$
				* $k^n$: 지수(exponential)
					* 피보나치 수열(재귀함수), 하노이의 탑
				* $n!$: 팩토리얼(Factorial)
					* 대게 브루트 포스를 이용한 해법들
				* 이외에도 많은 경우에 따른 수식들이 존재하나 실질적으로 고차함수나 지수함수 수준까지만 가더라도 n이 클경우 쓰기 제약되는 알고리즘들임.
* ### 시간복잡도(Time Complexity)
	* 보편적인 평가 기준으로, 걸리는 시간에 따라 평가하며, 보통은 Big-O 표기법이 사용됨. 
		* 최선의 경우, 혹은 평균적으로 엄청 빠르다 하더라도, 최악의 경우에 엄청 오래 걸린다면 프로그램은 멈춘거나 다름없기에 쓸모가 없다.
	* 보통 $O(n^3)$만 되더라도 상당히 쓰기 힘들다. 다만 n값이 크지 않은것이 명백한 경우 Big-O가 별로 의미 없을 수도 있다.
	* 1억번의 연산이 대략적으로 1초가 걸린다고 생각하면, 연산이 얼마나 최적화되야할지 알 수 있다. 게임 개발에 있어서는 프레임 드랍에 직접적인 영향을 준다.
	```c
	int main()
	{
		int sum = 0;
		int num = 100; //n = 100

		//n번 반복, O(n)
		auto start = std::chrono::high_resolution_clock::now();
		for(int i = 0; i < num; i++)
		{
			sum += 1;
		}
		auto end = std::chrono::high_resolution_clock::now();
		auto diff1 = end - start;
		std::cout << "작동시간: " << diff1.count() << std::endl;

		//n^2번 반복, O(n^2)
		start = std::chrono::high_resolution_clock::now();
		for(int i = 0; i < num; i++)
		{
			for(int j = 0; j < num; j++){
					sum += 1;
			}
		}
		end = std::chrono::high_resolution_clock::now();
		auto diff2 = end - start;
		std::cout << "작동시간: " << diff2.count() << std::endl;
		std::cout << "시간 차이 " << (diff2 - diff1).count() << "배!" << std::endl;
	}
	```
	
* ### 공간복잡도(Space Complexity)
	* 알고리즘 구동을 위해 필요한 메모리 공간의 양에 따라 평가한다. 시간복잡도처럼 Big-O 표기법이 일반적으로 사용된다.
	* 현대에는 PC/디바이스들의 증가한 메모리 용량, 프로세서 성능으로 중요도가 덜 부각되는 편이나, 임베디드, 모바일, 빅데이터 처리, 레거시 아키텍처에서의 개발등 용량의 제약이 강한 분야에서 중요할 수 있다.
		* 언리얼엔진, 유니티엔진을 이용하는 수준의 게임이라면 코드와 프로그램보단 모델이나 애니메이션, 텍스처, 사운드등 리소스가 메모리에서 차지하는 비중이 더 큰편이다.