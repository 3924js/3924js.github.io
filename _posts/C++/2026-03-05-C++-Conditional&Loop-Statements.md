---
title: "C++: 조건문(if, else), 반복문(for, while)"
author: Jaeseong Kim
date: 2026-03-05 12:00:00 +0800
categories: [C++, C++ Basics]
tags: [C++, C++ Basics]
---
* ## 조건문(Conditional Statement)
	* 조건에 따른 명령 수행을 구현, 다양한 상황에 대해 올바른 동작을 보장
	* ### if: 주어진 조건이 참이면 실행
		```c++
		int main(){
			int a = 1;
			//if(condition) {statement;}
			if(a > 0){
				cout << "hello world" << endl;
			}
		}
		```
	* ### else: 앞선 if의 조건이 거짓이면 실행
		```c++
		int main(){
			int a = 1;
			if(a < 0){	//실행되지 않음!
				cout << "hello world" << endl;
			}
			else{	//if 조건 불만족했으니 실행!
				cout << "else!" << endl;
			}
		}
		```
	* ### else if: else와 if의 조합, 앞선 if가 조건을 불만족하고 else if의 조건이 만족되면 실행.
		* 여러개의 else if 가 있어도 상관 없음
		```c++
		int main(){
			int a = 1;
			if(a < 0){	//실행되지 않음!
				cout << "hello world" << endl;
			}
			else if(false){	//실행되지 않음!
				cout << "first else if!" << endl;
			}
			else if(a > 0){	//조건 맞으니 실행!
				cout << "second else if!" << endl;
			}
			else{	//else if 조건 성립으로 실행 안됨!
				cout << "else!" << endl;
			}
		}
		```
	* if - else if - else 구조중 단 하나만 실행됨
	* if 안에 다른 if가 있어도 됨(중첩 반복문, nested if)
	* &&(and) 와 ||(or)등을 이용해 조건문 안에 여러 조건을 넣을 수 있음.
	* 간단한 계산기 코드 예시(사칙연산 선택 기능 추가)
	```c++
	int main()
	{
		int val1;
		cout << "첫번 째 값을 입력하세요: ";
		cin >> val1; // val1에 입력값 저장

		char opChar;
		cout << "수행할 계산을 입력하세요(+,-,*,/): ";
		cin >> opChar; // val1에 입력값 저장

		int val2;
		cout << "두번 째 값을 입력하세요: ";
		cin >> val2; // val2에 입력값 저장
		if(opChar == '+') cout << val1 << " + " << val2 << " = " << val1 + val2 << endl; //입력값들의 더하기 출력
		else if(opChar == '-') cout << val1 << " - " << val2 << " = " << val1 - val2 << endl; //입력값들의 빼기 출력
		else if(opChar == '*') cout << val1 << " * " << val2 << " = " << val1 * val2 << endl; //입력값들의 곱하기 출력
		else if(opChar == '/') cout << val1 << " / " << val2 << " = " << (float) val1 / val2 << endl; //입력값들의 나누기 출력
		else cout << "입력에 오류가 있습니다!" << endl;
	}
	```
	* ### switch
		* 여러개의 조건문의 늘어지는 코드 구조를 간결하게 바꿔주는 문법, 주어진 값의 상태에 따라 case를 나누어 작동.
		* break를 통해 case 실행 후 switch문에서 이탈시키지 않으면 뒤이어 오는 다른 case와 명령어들까지 수행함.
		* switch의 상수 조건값이 무엇인지에 따라 분기하기 때문에 범위조건(ex- 양수, 3보다 크다)은 if를 사용해야함.
		```c++
		//위 계산기 코드의 switch버전
		int main()
		{
			int val1;
			cout << "첫번 째 값을 입력하세요: ";
			cin >> val1; // val1에 입력값 저장

			char opChar;
			cout << "수행할 계산을 입력하세요(+,-,*,/): ";
			cin >> opChar; // val1에 입력값 저장

			int val2;
			cout << "두번 째 값을 입력하세요: ";
			cin >> val2; // val2에 입력값 저장
			switch(opChar){
				case '+':
				cout << val1 << " + " << val2 << " = " << val1 + val2 << endl; //입력값들의 더하기 출력
				break;
			case '-':
				cout << val1 << " - " << val2 << " = " << val1 - val2 << endl; //입력값들의 빼기 출력
				break;
			case '*':
				cout << val1 << " * " << val2 << " = " << val1 * val2 << endl; //입력값들의 곱하기 출력
				break;
			case '/':
				cout << val1 << " / " << val2 << " = " << (float) val1 / val2 << endl; //입력값들의 나누기 출력
				break;
			default:
				cout << "입력에 오류가 있습니다!" << endl;
				break;
			}
		}
		```
* ## 반복문(Loop Statement)
	* 필요한만큼 명령어를 반복하여 실행하여 코드의 반복을 줄임
	* ### for
		* 반복자(iterator)의 값에 따라 수행되는 반복문
		* 반복해야 하는 횟수가 명확할 때(ex-배열 탐색등) 사용하면 좋음
		```c++
		int main(){
			//0부터 9까지 출력
			//for(초기화식;조건식;증감식)
			for(int i = 0; i< 10; i++){
				cout << i << endl;
			}
		}
		```
	* ### foreach
		* for의 변형으로, 배열, std::array, std::vector등을 순회할 때 반복자를 인덱스로 사용해야 하던 구조 없이, 각 요소를 순회하며 반복 수행
			* 범위 기반 for문
			```c++
			int main(){
				int a[5] = {1,2,3,4,5};
				//복사형
				//for(요소자료형 요소명: 컨테이너)
				for(int num: a){
					cout << num << endl;
				}

				//참조형(const & 사용)
				for(const int& num: a){
					cout << num << endl;
				}
			}
			```
			* std::for_each
				std에 정의되어있는 반복문으로, 주어진 메모리 영역의 값들에 대해 함수를 수행
			```c++
			int main(){
				std::vector<int> a = {1,2,3,4,5};
				//std::for_each(시작주소값, 종료주소값, 반복할 함수)
				//람다함수를 이용한 for_each 출력 예시
				std::for_each(a.begin(), a.end(), [](int num){cout << num << endl;});
			}
			```
	* ### while
		* 조건이 참인 동안 계속해서 실행되는 반복문, 반복자가 필요 없으며, 조건만 만족하면 계속해서 작동함.
		```c++
		int main(){
			int num = 0;
			//0부터 9까지 출력
			//while(조건문) {명령문;}
			while(num < 10){
				cout << num << endl;
				num++;
			}
		}
		```
	* ### do while
		* while문의 변형으로, 명령의 최소 1회 실행을 보장함.
		* 실행 후에 조건을 판단.
		```c++
		int main(){
			int num = 1;
			//이미 num이 1이여서 조건이 안맞아도 hello world 출력
			//do {명령문;} while(조건문);
			do{
				cout << "hello world"<< endl;
				num++;
			}while(num < 1);
		}
		```
	* 조건문과 마찬가지로 반복문 안에 반복문이 들어갈 수 있음(지나친 중첩 반복문은 성능에 악영향을 줄 수 있음)
	* 조건식이 항상 참이라면 무한 반복문이됨 (infinite loop)
		* for와 while 모두 구현 가능하지만 while로 보통 구현하는편
		* 중간에 반복 탈출을 위한 조건문이 필요함.(break) 탈출하지 못하는 무한 반복문은 망가진 프로그램.
		```c++
		int main(){
			int num = 0;
			//hello world를 5번 출력
			while(true){
				cout << "hello world"<< endl;
				num++;
				//5번 반복했으면 반복문 탈출
				if(num >= 5){
					break;	
				}
			}
		}
		```
		
	* 흔한 반복문 연습용 별 피라미드 만들기
	```c++
		int main(){
			for(int i = 0; i< 5; i++){	//5열
				//9행 출력
				//왼쪽 공백
				for(int j = 0; j < 4 - i; j++){
					cout << ' ';
				}
				//별
				for(int j = 0; j < i * 2 + 1; j++){
					cout << '*';
				}
				//오른쪽 공백
				for(int j = 0; j < 4 - i; j++){
					cout << ' ';
				}
				cout << endl;
			}
		}
		```



