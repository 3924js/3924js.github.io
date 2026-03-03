---
title: "C Language: 동적할당(Dynamic Allocation,"
author: Jaeseong Kim
categories: [C++, C Intro]
tags: [C++]
---

* 8-1 힙 메모리(Heap Memory)
	* 메모리 레이아웃(Memory Layout): 컴퓨터 운영체제에서 메모리의 관리, 할당을 위해 가상으로 나누어놓은 메모리 공간
		* 스택 메모리(Stack Memory)
			* 컴파일 시점에 고정되는 지역변수(배열등)을 저장
			* 할당과 해제의 속도가 상수시간(O(1))로 빠름
			* 컴파일 이후 변경 불가능
			* 저장되는 정보들의 수명이 프로그램 수명과 같음.(불필요한 메모리 점유 가능)
		* 힙 메모리(Heap Memory)
			* 동적으로 할당과 해제 가능한 영역
			* 스택 메모리에 비해 할당과 해제가 느린편 (운영체제 개입, 메모리 탐색 등)
			*  
		* 코드 섹션(Code Section): 소스코드들의 빌드 결과물들이 저장
		* 데이터 색션(Data Section): 전역변수, 정적변수등 데이터가 저장
* 8-2 동적할당(Dynamic Allocation)
	* 프로그래머가 임의로 메모리 공간을 사용하겠다 선언하여 변수에 지정하는 방식
		* 메모리 할당(대여) (Memory Allocation) 
			* 힙 메모리 관리자에게 필요한만큼 메모리를 요청
			* 해당 크기만큼 연속된 메모리 위치의 주소를 찾아서 반환
		* 메모리 사용 (Memory Use)
			* 필요한 작업 수행 가능
			* 메모리에 기본으로 들어있는 값은 무작위 값(이전에 메모리에 쓰였던 값 등등)
				* 보안 영역으로 들어갈 경우 데이터 유출 가능
		* 메모리 해제(Memory deallocation, Memory delete)
			* 사용이 끝난 후, 메모리 할당 해제를 요청
			* 메모리 위치를 점유되지 않은 상태로 바꿈
	* 프로그램에 의한 자동 할당 해제가 되지 않기 때문에 해제하지 않으면 사용가능한 공간이 계속 줄어드는 메모리 누수(Memory Leak)가 생김
	* 사용 함수
		* malloc()	(Memory Allocate)
			* 필요한 공간만큼 메모리를 할당
				* 어떤 자료형을 할당할 지 모르기에 void*로 주소를 반환, 필요한 형태로 변환해줘야함.
			```c
			int* Num;
			Num = (int*)malloc(sizeof(int) * 10); //10개 짜리 int 배열
			Num[9] = 12;
			printf("%d", Num[9]); //12 출력
			```
		* calloc()		(Clear and Memory Allocate)
			* malloc()과 동일한 기능이나, 할당되는 공간의 값을 0으로 모두 초기화시켜 랜덤한 쓰래기 값으로 차있는 것을 방지.
			 ```c
			int* Num;
			Num = (int*)calloc(10, sizeof(int)); //10개 짜리 int 배열
			Num[9] = 12;
			printf("%d", Num[9]); //12 출력
			```
		* realloc() (Re-Allocate)
			* 이미 할당된 메모리를 다른 사이즈로 재할당하는 함수.
			```c
			int* Num;
			Num = (int*)calloc(10, sizeof(int)); //10개 짜리 int 배열
			Num[9] = 12;
			printf("%d", Num[9]); //12 출력
			realloc(Num, sizeof(int) * 14)// 14개 배열로 재할당
			Num[13] = -9;
			printf("%d", Num[13]); //-9 출력
			```
		* free()
			* 할당된 메모리를 해제해 사용 가능한 메모리로 되돌림
		* 
		* memset()
			* 메모리를 특정한 값으로 초기화
		* memcpy()
			* 다른 메모리의 내용을 복사해옴
		* memcmp()
			* 2개의 다른 메모리 값을 비교
* ## 9-1 구조체 (Structure)
	* 여러 자료형을 묶어 새로운 자료형으로 사용할 수 있게 해주는 기능
		```c
		struct Car
		{
			int Wheels;
			char Color;
			int Year;
		};	//세미콜론 필요
		
		int main(){
			struct Car Truck;
			Truck.Wheels = 4;
			Truck.Color = 'W';
			Truck.Year = 2016;
			printf("%d", Truck.Year); //2016 출력
		}
		```
		* 함수의 반환형, 매개변수형으로 사용하여 여러 값을 한번에 전달 가능
		```c
		struct Car
		{
			int Wheels;
			char Color;
			int Year;
		};	//세미콜론 필요
		void printInfo(struct Car tempCar){
		printf("%d %c %d", tempCar.Wheels, tempCar.Color, tempCar.Year);
		}
		int main(){
			struct Car Truck;
			Truck.Wheels = 4;
			Truck.Color = 'W';
			Truck.Year = 2016;
			printInfo(Truck); //4 W 2016 출력
		}
		```
* 9-2 Typedef
	* 기존 선언된 자료형을 새로운 이름으로 다른 이름을 추가
		* ex) site_t => typedef unsigned long long size_t
		* 구조체의 경우 불필요하게 struct 키워드를 반복해서 쓰지 않아도 됨
		```c
		typedef struct
		{
			int Wheels;
			char Color;
			int Year;
		}Car_t;	//세미콜론 필요
		
		int main(){
			Car_t Truck = {0,};	//struct 없이 선언
			printf("%d", Truck.Year); //2016 출력
		}
		```
		* . 과 -> 둘다 접근에 사용 가능
		* 일반적인 자료형처럼
			* 함수 반환형과 매개변수형에 사용 가능
			* 포인터 사용 가능
			* 배열 사용 가능
			* 구조체 안 맴버 변수 선언 가능
* 9-3 구조체와 클래스
	* 객체 지향 프로그래밍(Object Oriented Programming)의 class 개념과 유사
			* 필요한 여러 자료나 개념을 하나의 객체로 묶어서 사용.
			* OOP class의 상속(Inheritance), 접근 제한 지정자(public, private, protected)등 차이점 탓에 완벽히 같은 개념은 아님.
* 9-4 enum (열거형, Enumeration)
	* 특정한 값을 다른 이름으로 재선언
		* 단순 값이 아닌 이름 표현을 통한 가독성 증가
		* const같은 수정 방지 효과
	* 