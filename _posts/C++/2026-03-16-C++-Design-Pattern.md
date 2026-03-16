---
title: "C++: 디자인 패턴(Design Pattern)"
author: Jaeseong Kim
date: 2026-03-16 12:00:00 +0800
categories: [C++, C++ Basics]
tags: [C++, C++ Basics, Design Pattern, Singletone, Decorator, Observer]
---
* ## 소프트웨어 디자인 패턴(Software Design Pattern)
	* 소프트웨어 개발에 있어서 **반복적인 문제를 해결**하기 위한 해법으로서 이미 만들어 놓은 패턴들이다. 짧게 디자인 패턴이라고 부를 때가 더 많다.
	* 큰 범주에 따라 3가지 큰 가지로 나뉜다.
	* ### 생성 패턴(Creational Pattern)
		* 새로운 객체를 만드는 것에 관한 패턴이다. 즉 **생성**에 관여하는 방법을 제시한다.
		* ### 싱글톤(Singleton)
			*	문맥상 오직 **하나의 객체**만 존재하도록 보장하는 패턴이다.
			*	여러 객체로부터 하나의 객체에 접근하고 수정이 빈번히 일어날 때, 대상을 하나로 통일 시키는데 용이하다.
			*	전역변수와 유사한 개념이지만 단순 데이터인지, 전역적인 객체에 대한 접근제어인지에 따라 차이점을 가진다.
			*	사용 언어와 상황에 따라 여러가지의 구체적인 구현법이 존재하지만, 보통 다음은 공통적으로 들어간다.
				1. 생성자를 private으로 설정하여 외부에서의 직접적인 생성을 막는다.
				2. 인스턴스를 static으로 선언하여 클래스 스코프 안에서 1개만 생성되도록 한다.
				3. Getter를 static으로 선언하여 인스턴스를 얻을 때 오직 하나의 함수만 있도록 보장한다.
				4. (권장되지만 필요에 따라) 복사 생성자와 대입 생성자를 삭제하여 객체 복사를 막는다.
				
				```c++
				#include <iostream>
				using namespace std;
				
				class BasketBall {
				private:
					static BasketBall* Instance; //정적 인스턴스
					int BounceTime;
					BasketBall() {
						BounceTime = 0;
					}
					//연산자 삭제를 통한 복사방지
					BasketBall(const BasketBall&) = delete;
					BasketBall& operator=(const BasketBall&) = delete;
				public:
					static BasketBall* GetInstance() {	//Getter 함수
						if (Instance == nullptr){
							Instance = new BasketBall();
						}
						return Instance;
					}
					void ShowBounceTime() {	//Bounce 출력 함수
						cout << BounceTime << endl;
					}
					void Bounce() {	//Bounce 증가 함수
						BounceTime++;
					}
				};
				BasketBall* BasketBall::Instance = nullptr;	//정적 변수 초기화, C++ 17부턴 정적 변수도 선언부에서 초기화 가능
				class Player {	//농구선수 클래스
				public:
					void BounceBall() {
						BasketBall::GetInstance()->Bounce();
					}
				};
				int main() {
					Player A;
					Player B;
					//A와 B는 동일한 BasketBall을 호출, 총 2 증가
					A.BounceBall();
					B.BounceBall();
					// 2 출력
					BasketBall::GetInstance()->ShowBounceTime();
				}
				```
	* ### 구조 패턴(Structural Pattern)
		* 객체들의 **결합**에 관여하는 패턴이다. 연결이 어떤식으로 구성되는지에 대한 방법을 제시한다.
		* ### 데코레이터(Decorator)
			* **기존 클래스의 수정 없이 기능의 추가**를 구현하기 위한 패턴으로, 데코레이터가 기존 클래스를 감싸면서(Wrapper class) 기능을 추가하게 된다.
			* 상속으로 구현하기엔 클래스가 지나치게 많아지는 경우인 클래스 폭팔(Class Explosion)을 방지하고, 조합을 바탕으로 간결하게 다양한 경우의 수를 감당하지만, 구조가 복잡해지고 객체의 수가 증가하며 디버깅이 어려워 질 수 있다.
				```c++
				#include <iostream>
				#include <string>
				using namespace std;
				class Sandwich {	//샌드위치 클래스
				public:
					virtual ~Sandwich() {}
					virtual string GetInfo() = 0;
				};
				class BasicSandwich : public Sandwich {	//기본 샌드위치
				public:
					string GetInfo() override {
						return "Basic BLT\n";
					}
					~BasicSandwich() {
						cout << "BasicSandwich destroyed!" << endl;
					}
				};
				class SandwichDecorator : public Sandwich {	//샌드위치 데코레이터
				protected:
					Sandwich* sandwich;
				public:
					SandwichDecorator(Sandwich* sandwich) : sandwich(sandwich){}
					string GetInfo() override {
						return sandwich->GetInfo();
					}
					~SandwichDecorator() {
						cout << "Decorator destroyed!" << endl;
						delete sandwich;
					}
				};
				class Ham : public SandwichDecorator {// 햄 토핑
				public:
					Ham(Sandwich* sandwich) : SandwichDecorator(sandwich) {}
					string GetInfo() override {
						return sandwich->GetInfo() + ">> +Ham\n";	//햄 추가
					}
					~Ham() {
						cout << "Ham destroyed!" << endl;
					}
				};
				class Cheese : public SandwichDecorator {// 치즈 토핑
				public:
					Cheese(Sandwich* sandwich) : SandwichDecorator(sandwich) {}
					string GetInfo() override {
						return sandwich->GetInfo() + ">> +Cheese\n";	//치즈 추가
					}
					~Cheese() {
						cout << "Cheese destroyed!" << endl;
					}
				};
				int main() {
					Sandwich* A = new BasicSandwich();
					Sandwich* B = new Ham(A);
					Sandwich* C = new Cheese(B);
					Sandwich* D = new Ham(C);
					cout << D->GetInfo() << endl; //Basic sandwich, ham, cheese, ham 출력
					//순차적으로 해제: Ham -> decorator -> cheese -> decorator
					//-> ham ->decorator -> basic sandwich
					delete D;  
				}
				```
	* ### 행동 패턴(Behavioral Pattern)
		* 객체들 간의 **상호작용** 패턴이다. 상태가 변화할 때 그 변화를 어떻게 다루고 전달할지에 대한 방법을 제시한다. 앞서 STL과 백터에서 보았을 반복자(Iterator)도 행동 패턴의 구현체라고 볼 수 있다.
		* ### 옵저버(Observer)
			* 대상(Subject)를 **관찰**하는 관찰자(Observer)가 대상으로부터 **알림**을 받고(Notify) 그에 따른 동작을 하는 패턴이다.
			```c++
				#include <iostream>
				#include <vector>
				using namespace std;

				class Observer {	//옵저버 클래스 (동작)
				public:
					virtual void Update() = 0;
				};
				class DriveThru { //드라이브스루 클래스(대상)
				private:
					vector<Observer*> Observers;
				public:
					void Attach(Observer* Observer) {	//옵저버 추가 함수
						this->Observers.push_back(Observer);
					}
					void Detach(Observer* target){	// 옵저버 제거 함수
						for(auto i = Observers.begin(); i != Observers.end(); i++){
							if(*i == target){
								Observers.erase(i);
								return;
							}
						}
					}
					void notify() {	//알림 함수
						for (Observer* X : Observers) {
							X->Update();
						}
					}
					void EnteringCar() {	//알림 트리거 동작
						cout << "Order Received!" << endl;
						notify();
					}
				};
				class KitchenRoom : public Observer { //주방 클래스(동작)
				public:
					void Update() override {
						CookFood();
					}
					void CookFood() {
						cout << "Preparing the Order!" << endl;
					}
				};
				class Pickup : public Observer { //픽업 카운터 클래스
				public:
					void Update() override {
						GreetingCustomer();
					}
					void GreetingCustomer() {
						cout << "Welcome!" << endl;
					}
				};

				int main() {
					DriveThru* Gate = new DriveThru();
					KitchenRoom* Kitchen = new KitchenRoom();
					Pickup* Counter = new Pickup();
					
					//옵저버 부착
					Gate->Attach(Kitchen);
					Gate->Attach(Counter);
					
					//Notify
					Gate->EnteringCar();
					//Order Received! => Preparing the Order! => Welcome!

					//옵저버 제거
					Gate->Detach(Counter);

					//Notify
					Gate->EnteringCar();
					//Order Received! => Preparing the Order!

				}
				```
	* 지금 소개된 패턴 외에도 팩토리(Factory), 빌더(Builder), 어댑터(Adapter), 파사드(Facade)등 꽤나 많은 패턴들이 존재한다. 
	* 디자인 패턴 자체를 하나의 언어처럼 다루며 복잡한 문제를 해결하는 의사 코드(Pseudo code) 차원의 설계법으로 접근할 수도 있다.