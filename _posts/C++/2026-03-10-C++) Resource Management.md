---
title: "C++: 자원 관리(Resource Management)"
author: Jaeseong Kim
date: 2026-03-10 12:00:00 +0800
categories: [C++, C++ Basics]
tags: [C++, C++ Basics, Resource Management, Smart Pointer, Memory]
---
* ## 2-1 자원 관리(Resource Management)
	* 프로그램이 실행되는 과정에서 연산되고 참조되는 데이터는 모두 메모리 상에 존재.
	* 데이터는 생성 시점이나 방식에 따라 운영체제의 관리 하에 수명에 따라 특정 메모리 영역(Segment)에 할당 되며, 크게 4개로 구별된다. 주소값이 낮은 곳 부터 차례대로 코드, 데이터, 힙, 스택으로 나뉜다.
	* ### 코드(Code)
		* 텍스트(Text) 세그먼트, 코드(Code) 세그먼트로도 불린다.
		* 기계어로 컴파일된 명령어들이 저장되는 영역으로 프로그램의 작성 코드들이 보관된다.
		* 안정성을 위해 보통은 읽기 전용(Read-Only)로 설정된다.
		* 프로그램 시작시에 후술할 스택, 힙에 덮어씌워지는걸 방지하고자  낮은 주소값에 배치된다.
	* ### 데이터(Data)
		* 전역변수(Global Variable)과 정적변수(Static Variable)이 저장된다.
		* 초기값이 있는 데이터영역(Initialized Data Segment)과 초기값이 없는 영역(Block Started By Symbol, BSS)로 나뉘며, 전자가 더 낮은 주소값에 위치한다. BSS에 있는 변수들은 처음 시작시 0으로 초기화된다.
		```c++
		//Initialized Data에 저장
		int GlobalNum = 10;
		static int Num = 15;
		//BSS, Uninitialized Data에 저장
		char Globalchar;
		static bool Boolean;
		int main(){
			int a = GlobalNum;
		}
		```
	* ### 힙 메모리(Heap Memory)
		* 실행 도중 입력되거나 생성되는 데이터가 저장되는 메모리 공간.
		*	new 키워드를 이용한 할당이나 malloc, calloc등 메모리 할당 함수를 통해 생성되는 데이터들이 저장된다.
		*	필요에 따라 사이즈가 더 커질 수 있으며, 기본적으로 스택보다 많은 영역을 할당받는다.
			*	매우 큰 데이터를 다루거나 할당해야 하는 경우, 스택 메모리에서는 불가능해도 힙 메모리에선 가능한 경우가 있다.
		```c++
		int main(){
			//힙에 생성
			int* a = new int;
			int* b = (int*)malloc(sizeof(int));
			//힙에서 삭제
			delete a;
			free(b);
		}
		```
	* ### 스택 메모리(Stack)
		* 컴파일 과정에서 이미 생성되는 데이터들이 저장되는 메모리 공간.
		* 상수, 리터럴, 함수등 변하지 않을 값들이 배치된다.
		* 함수가 호출되거나 종료됨에 따라 자동적으로 생성 및 삭제되며,  힙 메모리보다 빠르게 동작한다.
	* 힙이 증가할때는 주소값이 높아지는 영역으로 증가하며, 스택이 증가할 때는 주소값이 낮아지는 영역으로 커진다. 일정량 이상 사용하여 사용 가능한 공간이 남지 않으면 힙과 스택의 영역이 겹쳐 충돌하게 된다.
		* 이는 스택 오버플로우(Stack Overflow)를 발생시키며, 함수의 비정상적인 작동을 야기할 수 있다.
		* 이를 방지하기 위해 운영체제는 힙과 스택의 충돌 전에 감지할 수 있는 기법들을 운영체제가 운용하며, 프로그램을 강제 종료할 권한을 가진다.
		```c++
		void Temp(){
			int a = 1;
		}
		int main(){
			//함수 호출과 함께 a 스택에 생성
			Temp();
			//함수 종료와 함께 스택에서 삭제
		}
		```
	* 언리얼 엔진에서는 프로그래밍 할때 여러 이유로 힙 메모리를 주로 사용하게 된다.
		* 함수의 수명을 넘어 프레임간의 유지되는 수명의 유연한 관리
		* 후술할 리플렉션 시스템의 반영
		* 독자적인 가비지 컬렉션(Garbage Collection)
	* ### 스마트 포인터(Smart Pointer)
		* 기존 포인터에는 소유권 개념이 없어 메모리가 실제 사용중인지 아닌지 추적하는데 설계와 노력을 필요로 한다.
		* 여러 포인터가 가리키는 메모리 주소가 어떤 한 포인터에 의해서 해지되었을 때, 다른 포인터들은 해당 주소에 있는 정상적이지 않은 값을 읽게 될 수 있으며, 이렇게 정상적이지 않은 주소를 가진 포인터를 허상 포인터(Dangling Pointer)라고 한다.
			```c++
			int main(){
				int* a = (int*)malloc(sizeof(int));
				int* b = a;
				*a = 10;
				//둘다 10 출력
				cout << *a << endl;
				cout << *b << endl;
				
				free(a);
				
				//이상한 값 출력
				cout << *b << endl;
			}
			```
		* 또한 사용하지 않는데 정상적으로 해지되지 않는 메모리가 계속 쌓이게 되면, 사용가능한 메모리 용량이 줄어들며 성능을 잡아먹고 메모리 누수(Memory Leak)가 일어난다.
		* C++에서는 메모리 공간을 보다 안정적으로 관리하고, new/delete를 사용하지 않는 자동적인 방식을 구현하기 위해 스마트 포인터(Smart Pointer)의 개념을 도입했다. 이 3가지 종류가 있으며, memory 라이브러리에 저장되어 있다.
			* ### 유니크 포인터(Unique Pointer, unique_ptr)
				* 오직 유니크 포인터 자신만 소유할 수 있다. 다른 유니크 포인터가 소유하려고 한다면 컴파일 에러를 보인다..
				* 복사가 불가능하며, std::move를 이용해 소유권을 다른 유니크 포인터로 이동할 수 있다.
					```c++
					#include <iostream>
					#include <memory>	//스마트 포인터 라이브러리
					using namespace std;
					class X{
					public:
						X(int val = 0){
							cout << "created!" << endl;
							num = val;
						}
						~X(){
							cout << "destroyed!" << endl;
						}
						void ShowNum(){
							cout << num << endl;
						}
					private:
						int num;
					};
					int main(){
						unique_ptr<X> A = make_unique<X>(15);	//Unique pointer 선언, created! 출력
						A->ShowNum(); //15 출력
						unique_ptr<X> B;
						//B = A; 오류! 유니크 포인터는 복사 불가능
						B = move(A);	//A에서 B로 소유권 이동, A는 nullptr로 바;
						B->ShowNum();	//15 출력
						return 0;
						//범위 이탈로 B 자동으로 소멸, 관리하던 객체 X도 소멸로 destroyed! 출력
						//일반포인터라면 자동 해제되지 않고 delete 로 직접 삭제해줘야함.
					}
					```
			* ### 공유 포인터(Shared Pointer, shared_ptr)
				* 소유권 대신 메모리가 참조되고 있는 횟수인 레퍼런스 카운트를 추적한다.
				* 레퍼런스 카운트가 0으로 떨어지면, 아무도 참조하지 않으면 자연스럽게 메모리 해제.
				* use_count()를 사용해 레퍼런스 카운트를 확인.
				* reset()으로 객체 소유를 해제하거나 다른 객체로 변경.
					```c++
					#include <iostream>
					#include <memory>
					using namespace std;
					class X{
					public:
						X(int val = 0){
							cout << "created!" << endl;
							num = val;
						}
						~X(){
							cout << "destroyed!" << endl;
						}
						void ShowNum(){
							cout << num << endl;
						}
					private:
						int num;
					};
					int main(){
						shared_ptr<X> A = make_shared<X>(15);	//shared pointer 선언, created! 출력, 레퍼런스 카운트 1
						shared_ptr<X> B = A;	//shared pointer는 unique랑 다르게 복사 가능, 레퍼런스 카운트 2
						A->ShowNum(); //15 출력
						B->ShowNum();	//똑같이 15 출력
						cout << A.use_count() << endl; //2
						A.reset();
						cout << B.use_count() << endl; //1
						B.reset();//레퍼런스 카운트 0, 객체 소멸, destroyed! 출력
						return 0; 
					}
					```
			* ### 약한 포인터(Weak Pointer, weak_ptr)
				* 공유포인터는 서로 참조하게 되었을 때 순환 참조로 인한 메모리 누수, 오류를 발생 시킬 수 있음.
					```c++
					#include <iostream>
					#include <memory>
					using namespace std;
					class Y;
					class X{
					public:
						X(){
							cout << "X created!" << endl;
						}
						~X(){
							cout << "X destroyed!" << endl;
						}
						shared_ptr<Y> ptr;
					};
					class Y{
					public:
						Y(){
							cout << "Y created!" << endl;
						}
						~Y(){
							cout << "Y destroyed!" << endl;
						}
						shared_ptr<X> ptr;
					};
					int main(){
						//포인터 2개 생성, X created!, Y created! 출력
						shared_ptr<X> A = make_shared<X>();
						shared_ptr<Y> B = make_shared<Y>();
						//순환참조
						A->ptr = B;
						B->ptr = A;

						cout << A.use_count() << endl; //레퍼런스 카운트 2
						
						return 0; //서로 참조중이여서 메모리 사용으로 인식, 객체 소멸 안됨. Destroyed! 출력 없음.
					}
					```
				* 이를 해결하기 위해, 공유포인터처럼 참조는 하지만, 레퍼런스 카운터를 증가시키지 않는 포인터인 약한 포인터를 사용.
					```c++
					#include <iostream>
					#include <memory>
					using namespace std;
					class Y;
					class X{
					public:
						X(){
							cout << "X created!" << endl;
						}
						~X(){
							cout << "X destroyed!" << endl;
						}
						shared_ptr<Y> ptr;
					};
					class Y{
					public:
						Y(){
							cout << "Y created!" << endl;
						}
						~Y(){
							cout << "Y destroyed!" << endl;
						}
						weak_ptr<X> ptr;	//약한 포인터로 교체
					};
					int main(){
						//포인터 2개 생성, X created!, Y created! 출력
						shared_ptr<X> A = make_shared<X>();
						shared_ptr<Y> B = make_shared<Y>();
						//순환참조
						A->ptr = B;
						B->ptr = A;

						cout << A.use_count() << endl; //레퍼런스 카운트 1
						return 0; //순환 참조 없음. 객체 삭제, Destroyed! 출력
					}
					```
	* ### 얕은 복사(Sallow Copy)
		* 포인터의 주소값만 복사하는 방식. 참조 형식의 전달과 유사하며, 복사 비용이 적다.
		* 스마트 포인터 시작 부분에서 언급했듯이, 포인터의 주소값으로 가리키는 위치의 객체가 원본 객체에 의해서 소멸되면 유효하지 않은 값을 가리키는 허상 포인터가 될 수 있음.
	* ### 깊은 복사(Deep Copy)
		* 포인터가 가리키는 클래스 객체의 값들을 새로운 메모리 영역에 완전히 복제하는 방식.
		* 복사 이후에는 완전히 다른 객체인 상태라, 원본의 삭제 여부와 상관없이 객체를 유지할 수 있지만, 한 객체에서의 수정 사항은 다른 쪽에 전달되지 않음.
		```c++
			#include <iostream>
			using namespace std;
			int main(){
				int* a = new int(3);
				int* b = new int(*a);
				*a = 10;
				cout << *a << endl; //10
				cout << *b << endl; //3
				
				delete a; //a 객체 삭제
				
				cout << *b << endl; //3
				delete b; //b 객체 삭제
			}
			```
	* ### 가비지컬렉션(Garbage Collection)
		* 메모리 사용을 주기적으로 체크하여 사용되지 않는 데이터, 메모리를 자동으로 해제해주는 기능으로, Java, Python같은 다른 언어에서는 내장되어있는 기능이다.
		* C++에는 이 기능이 내장되어있지 않으며, 동적 할당된 메모리의 수동 해제, 스마트 포인터 사용, GC의 개별적인 구현 등을 통해 메모리를 관리한다. GC 구현에 쓸 수 있는 알고리즘은 여러가지가 있다. 장단점과 상황에 따라 다양한 GC 방식을 구현/사용할 수 있다.
			* 참조 카운팅(Reference Counting): 공유 포인터에서 쓰인 기능과 동일한 방식이다. 참조된 횟수를 추적하여 참조되지 않을 때 메모리를 해제한다.
			* 마크 앤 스윕(Mark & Sweep)
				* 객체들을 훑어 사용되지 않는 객체들을 표시(Mark)한 후 해당 객체들을 청소, 쓸어버리는 방식(Sweep)으로 작동한다.
				* 속도는 다른 알고리즘에 비해 느린편이나 순환 참조 문제가 없고 비교적 간단하며 보편적인 방식이다.
				* 마크 앤 컴팩트(Mark & Compact)
					* 기존 마크앤 스윕에서 스윕 단계 대신 계속 쓰는 객체들의 주소를 옮겨서 정리하는 방식이다.
					* 메모리 파편화(Fragmentation) 문제가 없으나, 주소를 옮기는데 비용이들어 마크 앤 스윕보다 느릴 수 있다.
			* 복사 GC(Copying GC)
				* 2개의 힙 영역으로 나누고, 살아있는 객체들을 골라 다른 쪽 힙 영역으로 복사한 후 기존 힙 영역을 통째로 초기화하는 방식.
				* 작동 속도가 빠른 편이지만, 이 경우 힙 영역을 절반밖에 쓰지 못한다.
			* 이 외에도 
		* 언리얼 엔진에서는 독자적인 GC 기능이 구현되어 있으며 이는 마크 앤 스윕 메커니즘으로 구동한다.
			1. 루트 셋(Root Set)에 포함된 객체들을 확인한다.
			2. 루트 셋에서 직간접적으로 도달 가능한 객체들을 마크해놓는다. (그래프 탐색방법과 같다.)
			3. 마크되지 않은 개체들의 메모리를 회수한다.
		* 언리얼 엔진에서 인식하는 객체인 UObject에는 GC의 동작을 제어하기 위해 플래그(Flag)를 사용할 수 있다.
			* RF_RootSet: 해당 UObject가 루트셋의 일부로 관리됨을 알리는 플래그. AddToRoot()/RemoveFromRoot() 함수로 설정/해제.
			* RF_BeginDestroyed: 메모리에서 실제로 해제되기 전까지의 작업을 담당하는 함수인 BeginDestroy()의 호출을 나타내는 플래그.
			* RF_FinishedDestroyed: 메모리가 해제됬음을 알리는 FinishDestroy() 함수의 호출을 알리는 플래그.
	* ### 언리얼 리플렉션(Unreal Reflection)
		* 리플렉션(Reflection)
		* 언리얼 엔진은 C++로 쓰여진 객체들 자체를 인식할 수 없으며, 일부 기능은 C++의 기본적인 기능을 넘어서서 작동하고 통합되야 함.
			* 스코프 단위가 아닌 프레임 단위의 데이터 보존/관리.
			* 블루프린트, 에디터등 작업 환경의 통합.
		* 언리얼 엔진은 이 모든 작동하는 객체들을 UObject라는 단위로 다루게 되며, 
		* 