---
title: "C++: 탬플릿(Template), STL(Standard Template Library)"
author: Jaeseong Kim
date: 2026-03-12 12:00:00 +0800
categories: [C++, C++ Basics]
tags: [C++, C++ Basics, Overloading, Template, STL, Vector, Map, Find, Sort, Iterator]
---
* ### 함수 오버로딩(Function Overloading)
	* **동일한 이름의 함수**를 여러개 정의하는 행위.
		* C언어에서는 함수 이름만으로 구분 
		* C++에서 매개변수까지 함께 고려하기에, **매개변수 자료형이나 갯수가 다르면 다른 함수**로 인식.
		* 클래스에서 여러 생성자로 초기화하는 법을 구현한 것과 동일한 개념.
		```c++
		#include <iostream>
		using namespace std;
		void ShowValue(){
			cout << "No given value!" << endl;
		}
		void ShowValue(int val){
			cout << val << endl;
		}
		void ShowValue(double val){
			cout << val << endl;
		}
		int main(){
			ShowValue();		//No given value!
			ShowValue(3);		//3
			ShowValue(1.23);	//1.23
		}
		```
	* 오버로딩이 호출되지 않는 경우들도 존재.
		* 타입변환이 되는 매개변수로 인해 모호한 경우
			```c++
			#include <iostream>
			using namespace std;
			
			void ShowValue(float val){
				cout << val << endl;
			}
			void ShowValue(double val){
				cout << val << endl;
			}
			int main(){
				//float 과 double로 모두 형변환 가능.
				//무엇을 호출해야할 지 모호한 상태.
				//ShowValue(3);
			}
			```
		* 디폴트 매개변수로 인해 모호한 경우
			```c++
			#include <iostream>
			using namespace std;
			
			void ShowValue(float val1, float val2 = 3.33f){
				cout << val1 + val2 << endl;
			}
			void ShowValue(float val){
				cout << val << endl;
			}
			int main(){
				//디폴트 매개변수로 인해 첫번째 함수도 1개 매개변수로 실행 가능
				//2개 함수 다 호출 가능해 모호한 상태.
				//ShowValue(3.15);
			}
			```
		* 타입이 포인터와 배열인 점만 다른 경우
			```c++
			#include <iostream>
			using namespace std;
			
			void ShowValue(float val[]){
				cout << val[0] << endl;
			}
			void ShowValue(float* val){
				cout << *val << endl;
			}
			int main(){
				//매개변수에서 배열과 포인터는 구분되지 않음.
				//2개 함수 다 호출 가능해 모호한 상태.
				float x[3] = {1.11,1.12,1.13};
				//ShowValue(x);
			}
			```
		* 반환 타입만 다른경우
			```c++
			#include <iostream>
			using namespace std;
			
			int ShowValue(float val){
				cout << val << endl;
				return static_cast<int>(val);
			}
			/* 반환 타입으로는 구분되지 않음!
			float ShowValue(float val){
				cout << val << endl;
				return val;
			}*/
			int main(){
				float x = 3.185;
				ShowValue(x);
			}
			```
	* 컴파일러가 오버로딩 함수의 우선순위를 정하는데는 순서가 있다.
		1. 정확히 매칭되는 함수
		```c++
		#include <iostream>
		using namespace std;
		void ShowValue(int val){
			cout << val << endl;
		}
		void ShowValue(double val){
			cout << val << endl;
		}
		int main(){
			//ShowValue(int val) 호출, 3출력
			ShowValue(3);
		}
		```
		2. 타입 승격(손실이 없는 형변환)했을 때 매칭되는 함수
		```c++
		#include <iostream>
		using namespace std;
		void ShowValue(int val){
			cout << val << endl;
		}
		void ShowValue(double val){
			cout << val << endl;
		}
		int main(){
			//ShowValue(float val)없음
			//float->double 승격 가능, ShowValue(double val) 호출, 3.15 출력
			ShowValue(3.15f);
		}
		```
		3. 손실 있는 타입 변환했을 때 매칭되는 함수
		```c++
		#include <iostream>
		using namespace std;
		void ShowValue(int val){
			cout << val << endl;
		}
		int main(){
			//ShowValue(float val)없음
			//float->int 변환 가능, ShowValue(int val) 호출, 3 출력
			ShowValue(3.15f);
		}
		```
		4. 사용자 정의 타입 변환을 했을 때 매칭되는 함수.
		```c++
		#include <iostream>
		using namespace std;
		class temp{
		public:
			//int 형변환 연산자 오버로딩
			operator int() const {return 111;}
		};
		void ShowValue(int val){
			cout << val << endl;
		}
		int main(){
			//ShowValue(temp val)없음
			//temp->int 변환 가능, ShowValue(int val) 호출, temp-int 변환 값인 111출력.
			temp x;
			ShowValue(x);
		}
		```
* ### 탬플릿
	* 자료형 상관없는 코드를 작성하기 위한 문법으로 임의의 자료형을 임시로 가정하여 사용한다.
	* 동일한 동작을 함수를 오버로딩으로 작성하려 한다면 다른 자료형의 함수 여러개를 작성했겠으나, 탬플릿은 이를 간단하게 구현 가능
		```c++
		#include <iostream>
		using namespace std;
		/*
		int GetDouble(int val){
			return val + val;
		}
		float GetDouble(float val){
			return val + val;
		}*/
		//위 오버로딩 함수와 동일한 탬플릿 함수
		template<typename T>
		T GetDouble(T val){
			return val + val;
		}
		int main(){
			cout << GetDouble(3) << endl;		//6
			cout << GetDouble(1.33) << endl;	//2.66
		}
		```
	* ### 탬플릿 클래스
		* 함수를 일반화하는 것처럼 클래스도 일반화가 가능하다.
		* 뒤에 나올 STL 컨테이너에서 많이 쓰이는 형태의 이용법이다.
		```c++
		#include <iostream>
		using namespace std;
		
		template<typename T>	//템플릿 클래스
		class temp{	
		private:
			T val;	//템플릿을 이용한 맴버 변수 선언
		public:
			temp(T input){	//템플릿 매개변수 활용
				val = input;
			}
			T GetValue(){	//템플릿 반환형 활용
				return val;
			}
			void Increment(){
				val++;
			}
		};
		int main(){
			temp<int> X(15);	//int형 템플릿 클래스 생성
			X.Increment();
			cout << X.GetValue() << endl;	//16
			temp<double> Y(3.14);	//double형 템플릿 클래스 생성
			Y.Increment();
			cout << Y.GetValue() << endl;	//4.14
		}
		```
* ## 표준 탬플릿 라리브러리 (STL, Standard Template Library)
	* C++에서 제공하는 많이 쓰이는 자료구조와 알고리즘, 기능들을 구현해놓은 라이브러리.
	* 다양한 모듈이 개별적으로 존재하기 때문에, 필요에 따라 필요한 모듈만 포함시켜 사용하며, 여러 카테고리로 분류된다.
		1. ### 컨테이너(Container)
			* 데이터를 담고 관리하는 객체로, 자료구조에 기반해 구현되어있다.
			* 여러가지 자료형을 담을 수 있도록 템플릿을 기반으로 구현되어 있으며, 탬플릿 클래스의 사용법처럼 사용자가 자료형을 지정한 후 사용한다.
			* 컨테이너 내부에서 동적으로 메모리를 관리하며, 사용자는 삽입, 수정, 삭제등에 대해 개별적인 할당과 해제를 하지 않아도 된다.
			* 반복자(Iterator)를 이용해 순회할 수 있다.
			* ### [벡터(Vector)](https://3924js.github.io/posts/Algorithm-Vector/) 
				(Algorithm에서 이미 한번 정리했다.)
				* vector 모듈에 정의된 배열과 유사한 컨테이너로, 최대 크기가 정적이지 않고 원소의 갯수에 따라 동적으로 늘어난다.
				* [] 를 이용한 무작위 접근이 가능하다.
				* 작동속도가 빠른 삽입/삭제 기능인 push_back(), pop_back()을 쓸 수 있다.
				* insert()/erase()를 통한 삽입/삭제가 가능하지만, 많이 써야할 때는 느리기에 push_back()/pop_back()을 이용하는게 좋다.
					* 무작위 위치에 대한 지속적인 삽입/삭제가 이루어질 경우 다른 자료구조가 더 효율적이다.
			* ### 맵(Map)
				* map 모듈에 정의된 컨테이너로, 키(key)와 값(value)의 쌍(Pair)로 이루어진다. Python의 dictionary와 유사하다.
				* 키 기준 오름차순으로 자동 정렬되어있다.
				* STL에 정의되어 있지 않은 이진트리(Binary Tree)구조를 부분적으로 대체할 수 있다.
				```c++
				#include <iostream>
				#include <map>
				using namespace std;
				int main(){
					map<int, string>
				}
				```
			* 이 외에도 리스트(list), 어레이(Array), 셋(set)등 다른 자료구조도 정의되어 있다.
		2. ### 알고리즘(Algorithm)
			* 동작하는 방법에 관한 객체
			* sort()
				* 조건에 따라 자료구조 안의 값들을 정렬한다.
				* 조건이 없으면 오름차순 정렬로 이루어지고, 필요에 따라 조건으로 쓸 bool 을 반환하는 함수를 사용할 수도 있다.
				```c++
				
				```
			* find()
				* 주어진 값을 찾아서 해당 인덱스의 반복자를 반환한다.
				* 주어진 값을 찾지 못하면 컨테이너 끝까지 탐색한 것으로 end()를 반환한다.
				```c++
				```
		3. ### 반복자(Iterator)
			* 컨테이너와 알고리즘에 대해 일반화된 접근법을 제공해주는 문법이다.
			* 기존의 포인터 개념과 유사하게 동작하며, 컨테이너에 사용하여 역참조를 통해 원소에 접근할 수 있다.
			* 순방향 반복자
				* 가장 기본적인 반복자로, 첫번째 원소에서 시작해서 마지막 원소 다음까지 움직인다.
				* 포인터처럼 역참조가 가능하다.
				* 
			* 역방향 반복자(reverse_iterator)
				* 컨테이너 마지막 원소에서 시작해 첫번째 원소 앞까지 움직이는 반복자이다. 방향이 반대로 바뀐 것이고 기능은 동일하다.
				* 움직일 때 감소시키는게 아닌 일반 반복자처럼 증가시키면서 끝으로 향한다.
				* 순방향 반복자와 클래스가 달라 호완되지 않는다. 대신 base()로 변환해 사용할 수 있다.
		4. 이외에도 입출력, 시스템 관련 기능, 리플렉션/메타 프로그래밍등 다양한 목적에 맞는 모듈을 포함시키게 된다. (ex-가장 처음부터 보았던 iostream도 입출력 관련 모듈이다.)
	* [C++Reference](https://cppreference.com/)라는 국제 표준화 기구(International Organization for Standization , ISO)에서 공식적으로 운영하는 C++에 관한 정보를 정리해놓은 사이트에 [STL](https://cppreference.com/w/cpp/standard_library.html)관련 내용도 잘 정리되어 있다. 워낙 방대하고 기본적인 라이브러리다 보니 필요에 따라 참고하면 좋다.
		