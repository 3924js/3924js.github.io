---
title: "C++: 클래스(class) & 객체 지향 프로그래밍(Object Oriented Programming)"
author: Jaeseong Kim
date: 2026-03-09 12:00:00 +0800
categories: [C++, C++ Basics]
tags: [C++, C++ Basics]
---
* ## 1-5 클래스(Class)
	* C++에서 도입된 객체 지향 프로그래밍의 객체(Object)를 구현하기 위한 구조로, 필요한 데이터와 기능을 서로 관련된 덩어리/집합인 객체로 묶고 세부적인 구현을 외부로부터 감추기 위해 사용.
		* 사용자가 알아야 할 내용이 감소함으로서 보다 쉽고 간편한 사용 유도.
		* 권한에 따른 제한적인 접근을 구현함으로서 의도되지 않은 동작 예방
		* C 언어에서의 구조체와 유사하지만, 멤버 함수 가능, 상속, 접근제어등의 기능에 있어서 차이가 있음. (C++의 구조체는 기능적으로 클래스와 거의 동일하며 상속, 접근제어등도 가능, 기본 접근 지정자만 public으로 차이있음.)
		```c++
		//class 이름{}
		class A{
		public:
			int a;	//멤버 변수
			void printNum(){	//멤버 함수(struct에서 안되던 개념)
				cout << a << endl;
			}
		};
		int main(){
			//class를 자료형처럼 사용
			//클래스이름 오브젝트이
			A temp;
			temp.a = 10;	//A내부 변수 수정
			temp.printNum();	//10 출력
		}
		```
	* ### 접근제어(Access Control)
		* 키워드를 통해 멤버 함수와 변수에 대한 접근 권한을 설정할 수 있음.
		* public: 외부에서도 자유롭게 접근 가능.
		* private: 외부에서 접근 불가능. 별도의 지정자가 없는 멤버들의 경우 기본적으로 private으로 인식.
		* protected: 클래스의 자식 클래스에서만 접근가능. (상속에 대해 알아야 이해 가능)
		```c++
		class A{
		public:	//외부 접근 가능
			int a;
			void SayHello(){
			cout << "Hello" << endl;
			}
		private:	//외부 접근 불가
			int b;
			void SayBye(){
			cout << "Bye" << endl;
			}
		};
		int main(){
			A temp;
			//public 변수 접근
			temp.a = 10;	
			//private 변수 접근(inaccesible 오류 발생!)
			//temp.b = 100;
			
			//public 함수 호출
			temp.SayHello();
			//private 함수 호출 (inaccesible 오류 발생!)
			//temp.SayBye();
		}
		```
	* ### 접근자(Getter)/설정자(Setter)
		* 위 코드에서처럼 변수에 직접 접근하여 값을 획득하고 수정하는 행위는 예기치 않은 동작을 야기할 가능성 존재.
		* 맴버 변수들은 private로 설정하고 이에 값을 획득하는 함수인 접근자(Getter), 값을 설정하는 함수인 설정자(Setter)를 별도로 사용하여 안정적인 프로그램 구현 가능
			* Getter로 얻은 변수값을 수정해도 맴버 변수에 영향 없음. (참조로도 반환할 수 있지만 데이터 보호를 위해 필요시에만 사용.)
			* Setter로 맴버 변수가 올바른 값으로 수정되도록 설계 가능.
			```c++
			class A{
			public:
				int GetNum(){	//Num 값을 반환
					return Num;
				}
				void SetNum(int Val){	//Num 값을 설정
					if(0 < Val && Val < 100){
						Num = Val;
					}
				}
			private:
				int Num;	//class 외부에서 직접 접근 불가능
			};
			int main(){
				A temp;
				temp.SetNum(10);
				cout << temp.GetNum() << endl; // 10출력
				temp.SetNum(1000);	//100초과, 수정 안됨.
				cout << temp.GetNum() << endl; // 10 출력.
			}
			```
	* ### 생성자(Constructor)
		* 클래스가 생성되어 오브젝트화될 때 자동으로 호출되어 처음 수행되야될 동작들을 보장하는 맴버 함수. 클래스 이름과 동일한 함수 명으로 선언함.
		* 여러개의 매개변수에 따라 여러 종류의 생성자를 만들어 놓을 수 있음. 같은 이름의 함수를 매개변수의 종류에 따라 여러 형태로 구현해놓는 것을 오버로딩(Overloading)이라고 함.
		```c++
			class A{
			public:
				//클래스 이름(매개변수){ 생성자 내용;}
				A(){	//매개변수 없는 생성자
					Num = -1;
				}
				A(int Val){	//정수를 받는 생성자
					Num = Val;
				}
				A(float Val){	//실수를 받는 생성자, 정수로 형변환 후 저장
					Num = (int)Val;
				}
				int GetNum(){
					return Num;
				}
			private:
				int Num;
			};
			int main(){
				A a;
				cout << a.GetNum() << endl; // -1출력
				A b(3);
				cout << b.GetNum() << endl; // 3 출력
				A c(1000.315f);
				cout << c.GetNum() << endl; // 1000 출력
				//A d("qqq");	//오류! 해당되는 생성자 없음!
			}
			```
	*  ### 소멸자(Destructor)
		* 객체가 소멸될 때 호출되는 함수. 생성자의 반대 역할. 객체의 소멸을 알리거나 소멸시 수행되야할 동작들을 보장.
		```c++
			class A{
			public:
				A(){
					cout << "Class Constructed!" << endl;
				}
				~A(){
					cout << "Class Destructed!" << endl;
				}

				int GetNum(){
					return Num;
				}
			private:
				int Num;
			};
			int main(){
				A a;	//class Constructed! 출력
				//함수 종료, a 객체 메모리 해제되며 소멸자 호출
				//class Destructed! 출력
				return 0; 
			}
			```
	* ### 헤더파일 활용
		* 위 예시들에선 맴버 함수들도 클래스 내부에서 구현했지만, 기본적으로는 선언과 구현은 분리해야 함. 선언부들은 헤더파일에 저장하며, 소스파일은 그 헤더파일을 불러와 구현하거나 사용하는 구조로 분리.
			* 선언부만 분리시킴으로서 여러 클래스와 파일들을 프로젝트에서 다룰 때 구현부의 중복 포함을 방지.
			* 선언부만 독립적으로 여러 파일에서 불러올수 있게 함으로서 재사용성 향상.
			* 선언부와 구현부의 독립성을 통한 모듈화 기여.
	* 간단한 성적 관리 프로그램.
		```c++
		//Student.h
		#ifndef STUDENT_H	//STUDENT가 정의되어있지 않으면 실행
		#define STUDENT_H	//STUDENT 정의
		class Student{
		public:
			//입력이 없으면 0으로 초기화
			Student(int History = 0, int Math = 0, int Science = 0){
				this->History = History;
				this->Math = Math;
				this->Science = Science;
			}
			//구현 없이 선언만 마무리
			float GetAverage();
			int GetHighest();
			int GetTotal();
		private:
			int History;
			int Math;
			int Science;
		};
		#endif //STUDENT_H ifdef 구문 종료
		
		```
		```c++
		//Student.cpp
		#include "Student.h"
		#include <algorithm>
		float Student::GetAverage(){
			return static_cast<float>(this->History + this->Math + this->Science)/3;
		}
		int Student::GetHighest(){
			return std::max(std::max(History, Math), Science);
		}
		int Student::GetTotal(){
			return this->History + this->Math + this->Science;
		}
		```
		```c++
		//main.cpp
		#include <iostream>	//표준 라이브러리등 시스템 경로에서 검색할 때 <> 사용.
		#include "Student.h"	//프로젝트 내의 파일을 검색할 때 ""사용.
		using namespace std;
		
		int main(){
				Student Kim(30, 20, 10);
				cout << Kim.GetAverage() << endl; //20
				cout << Kim.GetHighest() << endl;	//30
				cout << Kim.GetTotal() << endl;	//60
				return;
		}
		```
	* 배터리 사용 시뮬레이션
		```c++
		//Battery.h
		#ifndef BATTERY_H
		#define BATTERY_H
		class Battery{
		public:
			//100으로 초기화
			Battery(){
				Power = 100;
			}
			int GetRemainings();
			void Consume();
			void Charge();
		private:
			int Power;
		};
		#endif //BATTERY_H
		
		```
		```c++
		//Battery.cpp
		#include "Battery.h"
		#include <algorithm>
		int Battery::GetRemainings(){
			return this->Power;
		}
		void Battery::Consume(){	//7만큼 감소, 0보다 내려갈 수 없음
			this->Power = std::max(0, this->Power - 7);
		}
		void Battery::Charge(){	//5만큼 증가, 100보다 올라갈 수 없음
			this->Power = std::min(100, this->Power + 5);
		}
		```
		```c++
		//main.cpp
		#include <iostream>	
		#include "Battery.h"
		using namespace std;
		
		int main(){
				Battery Samsung;
				cout << Samsung.GetRemainings()<< endl; //100
				Samsung.Charge();
				cout << Samsung.GetRemainings()<< endl; //100
				Samsung.Consume();
				cout << Samsung.GetRemainings()<< endl; //93
				return;
		}
		```
	*  분수(Fraction) 구현 클래스
		```c++
		//Fraction.h
		#ifndef FRACTION_H
		#define FRACTION_H
		class Fraction{
		public:
			//100으로 초기화
			Fraction(int Num = 0, int Denom = 1){
				Numerator = Num;
				Denominator = Denom;
			}
			//구현 없이 선언만 마무리
			void Display();
			void Simplify();
		private:
			int Numerator;
			int Denominator;
		};
		#endif //FRACTION_H
		
		```
		```c++
		//Fraction.cpp
		#include "Fraction.h"
		#include <algorithm>
		#include <iostream>
		using namespace std;
		void Fraction::Display(){
			cout<< Numerator << "/" << Denominator << endl;
		}
		void Fraction::Simplify(){
			//둘중 작은 수부터 내려가며 먼저 나오는 공약수로 약분
			//빠른 성능을 원하면 std::gcd도 사용 가능
			for (int i = std::min(Numerator, Denominator); i >= 1; i--){
				if(Numerator % i == 0 && Denominator % i == 0){
					Numerator /= i;
					Denominator /= i;
					return;
				}
			}
		}
		```
		```c++
		//main.cpp
		#include <iostream>	
		#include "Fraction.h"
		
		
		int main(){
				Fraction A;
				Fraction B(6,10);
				A.Display(); // 0/1
				B.Display(); // 6/10
				B.Simplify();
				B.Display();	//3/5
				B.Simplify();
				B.Display();	//3/5
		}
		```
		 
* ### 1-6 객체 지향 프로그래밍(Object Oriented Programming)
	* C언어로 대중화된 절차 지향 프로그래밍의 한계점인 유지보수성, 재사용성을  보완하기위해 제안된 프로그래밍 패러다임중 하나로, Java, C++, Python같은 고급 프로그래밍 언어의 보다 추상화되고 일상어에 가까운 개념을 제시.
	* 기존 C언어는 기계어(Machine Language)-어셈블리어(Assembly Language) -C언어(C Language)로 이어지는 언어의 고급화, 추상화를 통해 프로그래밍을 보다 쉽고 가독성있게 만들었지만, 함수, 구조체 구조로는 극복하지 못하는 코드 재사용성과 복잡함에 대한 관리 한계가 있었음.
	* 객체 지향 프로그래밍은 보다 나은 설계와 유지보수성을 위해 객체들의 상호작용을 중심으로 프로그래밍하는 방법을 제시하며, 다음 4가지 핵심 원리, 객체 지향의 4 기둥(4 Pillars of OOP) 를 포함. 
		* 캡슐화(Encapsulation)
			* 데이터와 함수를 클래스 단위로 묶어 외 접근을 제한.
			* 위에서 배운 Getter, Setter내용과 일치하며, 변수를 private으로 두고 필요한 함수들만 public으로 열어두어 간접적인 접근만 가능하게 함.
		* 상속(Inheritance)
			* 기존 클래스의 개념을 다른 클래스가 이어받을 수 있음.
			* 물려주는 클래스가 부모 클래스(Parent Class), 물려받는 클래스가 자식 클래스(Child Class)
			* 부모 클래스에 정의된 개념은 자식클래스에서 사용 가능하며, 자식 클래스는 추가적인 맴버 변수나 함수 정의가 가능.
				```c++
				#include <iostream>
				using namespace std;
				class Transportation{	//부모 클래스
				public:
					void Drive() {
						cout << "Going to the City Hall" << endl;
					}
				};
				class Taxi : public Transportation{//자식 클래스

				};
				int main() {
					Transportation* A = new Taxi();	//상속받은 자식 클래스 객체를 부모클래스 포인터로 가리킴(업캐스팅)
					A->Drive(); //시청으로 출발
				}
				```
		* 추상화(Abstraction)
			* 복잡한 내부 구현을 숨김으로서 필요한 기능만 드러나는 단순한 개념으로 변환
			* 캡슐화와 유사하지만 데이터 보호와 구조의 단순화중 어느 부분에 초점을 맞추느냐에 따라 다음.
			* 추상 클래스와 인터페이스를 이용해 구현.
				```c++
				#include <iostream>
				using namespace std;
				class Transportation{	//부모 추상 클래스
					public:
						virtual void Drive() = 0;	//순수 함수, 구현은 하위 클래스에서!
				};
				class Taxi1 : public Transportation{	//추상화된 함수 구현 1
					public:
						void Drive() override{
							cout << "Going to the City Hall" << endl;
						}
				};
				class Taxi2 : public Transportation{
					public:
						void Drive() override{
							cout << "Going to the Supermarket" << endl;
						}
				};
				int main(){
					Taxi1 A;
					Taxi2 B;
					A.Drive(); //시청으로 출발
					B.Drive(); //슈퍼마켓으로 출발
				}
				```

		* 다형성(Polymorphism)
			* 변수, 함수등의 요소들이 상황에 따라 다양하게 해석되는 것에 대한 허용.
			* 오버로딩(Overloading)
				* 같은 이름의 함수가 매개변수에 따라 다양한 형태로 구현하는 기능
				* 오버로딩으로 다양한 입력을 받는 생성자 코드 예시와 동일
				```c++
				#include <iostream>
				using namespace std;
				class Transportation{
					public:
						Transportation(int val = 0){	//int 생성자, 입력 있으면 val, 없으면 0
							Price = val ;
						}
						Transportation(float val){	//float 생성자
							Price  = (int) val;
						}
						void ShowPrice(){
							cout << "Price: " << Price << endl;
						}
					private:
						int Price;
				};
				int main(){
					Transportation A;
					Transportation B(30);
					Transportation C(5.689f);
					A.ShowPrice(); //0 출력
					B.ShowPrice(); //30 출력
					C.ShowPrice(); //5 출력
				}
				```
			* 오버라이딩(Overriding)
				* 자식 클래스에서 부모 클래스의 함수의 내용을 새로운 내용으로 덮어쓰는 행위
				* 부모에서 virtual 키워드를 사용함으로서 오버라이딩에 대한 허용을 표시.
				* 자식에선 overriding 키워드를 사용해 함수를 오버라이딩 할 것이라 선언.
				* 부모 클래스에서 함수를 호출하더라도, 자식 오브젝트에 구현된 오버라이딩 함수가 있으면 자식의 함수를 호출
				```c++
				#include <iostream>
				using namespace std;
				class Transportation{
					public:
						Transportation(){
							Price = 10;
						}
						virtual void ShowPrice(){
							cout << "Price: " << Price << endl;
						}
					protected:
						int Price;
				};
				class Taxi : public Transportation{
					public:
						void ShowPrice() override{
							cout << "Taxi Price: " << Price << endl;
						}
				};
				int main(){
					Transportation* A = new Transportation();
					Transportation* B = new Taxi();
					A->ShowPrice(); //Price: 10 출력
					B->ShowPrice(); //Taxi Price: 10 출력, Transportation* 이여도 자식 함수 호출
					delete A;
					delete B;
				}
				```
	* 클래스를 이용한 게임 직업 예시
		```c++
		//Adventure.h
		class Adventure{//추상화 클래스
			public:
				virtual void useSkill() = 0; //순수 가상 함수
		};
		class Warrior: public Adventure{
			public:
				void useSkill() override;
		};
		class Mage: public Adventure{
			public:
				void useSkill() override;
		};
		class Archer: public Adventure{
			public:
				void useSkill() override;
		};
		```

		```c++
		//Adventure.cpp
		#include <iostream>
		#include <Adventure.h>
		using namespace std;
		//각자 다른 스킬 텍스트 출력
		void Warrior::useSkill(){
			cout << "Warrior uses Slash!" << endl;
		}
		void Mage::useSkill(){
			cout << "Warrior casts Fairball!" << endl;
		}
		void Archer::useSkill(){
			cout << "Warrior shoots an arrow!" << endl;
		}
		```

		```c++
		//main.cpp
		#include <iostream>
		#include <Adventure.h>
		int main(){
			Adventure* A = new Warrior();
			Adventure* B = new Mage();
			Adventure* C = new Archer();
			A->useSkill();
			B->useSkill();
			C->useSkill();
		}
		```

