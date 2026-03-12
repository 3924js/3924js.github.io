---
title: "C++: 벡터(Vector)"
author: Jaeseong Kim
date: 2026-03-11 12:00:00 +0800
categories: [C++, Algorithm]
tags: [C++, Algorithm, Vector, Cache, iterator]
---
* ## 벡터(Vector)
	```c++
	#include<vector>
	using namespace std;
	int main(){
		//vector<자료형> 변수이름;
		vector<int> Nums;
		return 0;
	}	
	```
	* 고정적인 기존 배열과는 다르게, 필요에 따라 **자동으로 크기가 늘어나는 동적 배열**로, size와 capacity를 가짐.
		* size: 실제 들어있는 원소의 갯수.
		* capacity: 확보되어있는 메모리 공간의 크기.
		* size > capacity이면 넘어가게 될 때 2배 크기로 새로운 공간에 재할당하며, O(n)의 시간복잡도를 가짐(비주얼 스튜디오의 MSVC에선 1.5배이며, 컴파일러마다 다를 수 있음.)
	* C++에선 vector 라이브러리를 포함시킴으로서 사용할 수 있다. 매우 다양한 초기화 방법이 존재.
		```c++
		#include <iostream>
		#include <vector>
		using namespace std;
		int main(){
			vector<int> A; //빈 벡터 생성
			vector<int> B(3); //3 크기의 벡터 생성
			vector<int> C(3,1);	//3 크기의 벡터의 모든 원소를 1로 초기화
			vector<int> D = {1,2,3}; //초기화 리스트 이용
			vector<int> E{1,2,3};	//초기화 리스트 이용
			vector<int> F(E); //다른 벡터를 이용한 초기화
			vector<int> G(E.begin(), E.begin() + 2); //다른 벡터의 일부만 사용하여 초기화
			int Array[] = {1,2,3};
			vector<int> H(Array, Array + 3); //다른 배열을 이용해 초기화
			H.assign(3,1); //이미 선언된 벡터를 초기화, 3 크기의 벡터의 모든 원소를 1로 채움
		}
		```
	* vector와 함께 쓸 수 있는 함수들이 다양하게 존재한다.
		* ### 접근
			* vector[i]
				* 배열에 인덱스를 통한 방법과 같은 무작위 접근법
				* 마찬가지로 접근만 하면 되기에, $O(1)$의 시간복잡도를 가짐.
			* at(i)
				* 위의 무작위 접근과 같으나 인덱스가 범위에 맞지 않으면 std::out_of_range 예외를 발생시킴.
			* front()
				* 처음 원소를 반환
			* back()
				* 마지막 원소를 반환
			* ### 반복자(iterator)
				* 인덱스 대신 쓰일 수  있는 포인터처럼 작동하는 객체로, 역참조를 통해 바로 원소 접근 가능
				* begin(): 시작지점 주소
				* end(): 벡터가 끝나는 주소 (마지막 원소 X)
				* rbegin(): 백터 반대에서 볼때 시작지점의 주소, 즉 마지막 원소의 주소.
				* rend(): 백터 반대에서 볼때 벡터가 끝나는 지점에 주소, 즉 시작 원소 이전 주소
				* base(): rbegin/rend등으로 얻는 reverse iterator에서 자료형을 기본적인 일반 iterator로 변환. 일반 iterator기준으로 1 큰 값으로 변환되니 유의.
				* cbegin()/cend(): 읽기전용 상수 반복자, 가리키는 주소는 begin()/end()와 같음.
				* distance(A,B): 두 반복자 A,B의 차이를 반환.
		* ### 수정
			* push_back()
				* 벡터 마지막에 새로운 원소를 추가
				* 끝에만 접근하면 되기에, O(1)의 시간복잡도로 빠르게 처리 가능
				* 추가로 인해 size가 capacity를 넘어가게 되면 capacity를 증가시키게 되며 이때는 $O(n)$의 시간복잡도를 가짐
					* push_back마다 새로 공간을 할당해야 했다면, (n개 삽입) * (n번의 복사를 통한 capacity 증가) => $O(n^2)$의 시간복잡도였을 것.
			* pop_back()
				* 벡터 마지막에 있는 원소를 벡터에서 삭제
				* 마찬가지로 접근만 하면 되기에, $O(1)$의 시간복잡도를 가짐.
				* size가 이 작업으로 인해 capacity보다 줄어들어도 별도의 capacity 조정 작업은 없음.
			* insert(v.begin() + 1, val)
				* 인덱스 특정 위치에 값을 삽입
				* 인덱스 뒤의 자료들을 주소상 연속되게 한칸씩 뒤로 민 후 삽입해야되서 $O(n)$으로 느려짐.
				* push_back()처럼 capacity를 초과한 삽입은 추가적인 $O(n)$을 발생.
			* erase(v.begin() + n)
				* 인덱스 특정 위치의 값을 삭제
				* 삭제 후 뒤의 자료들을 주소상 연속되게 한칸씩 옮겨야되기에 $O(n)$으로 느려짐.
			* insert/erase를 많이 반복해야하는 작업이면, 경우에 따라 새로운 vector에 특정 값들을 선택적으로 push하는게 더 빠를 수 있음.
		* ### 정보
			* size(): 벡터의 크기를 반환
			* capacity(): 벡터의 용량을 반환
			* empty(): 벡터가 비었는지 여부를 반환
			* max_size(): 벡터의 이론적인 한계 용량, 시스템 메모리를 기준으로 한다.
			* data(): 내부 배열의 시작 주소를 반환한다. 
			```c++
			#include <iostream>
			#include <vector>
			using namespace std;
			int main(){
				vector<int> A(10, 10); //10이 10개 있는 벡터 생성
				cout << "최대 가능 용량: " << A.max_size() << endl;
				//at()을 이용한 10번째 원소, 마지막 값 출력
				cout<<"마지막 원소:" << A.at(9)<<", 길이: " << A.size() << ", 용량: " << A.capacity() << endl;
				
				A.push_back(3);	//마지막에 3 추가
				//rbegin() 역참조를 이용한 마지막 값 출력
				//용량 초과로 벡터 재할당, 용량 증가
				cout<<"마지막 원소:" << *(A.rbegin())<<", 길이: " << A.size() << ", 용량: " << A.capacity() << endl;
				
				A.pop_back(); //마지막 값 삭제
				//end() 역참조를 통한 마지막 값 출력
				cout<<"마지막 원소:" << *(A.end() - 1)<<", 길이: " << A.size() << ", 용량: " << A.capacity() << endl;
				
				A.insert(A.begin(), 20);	//첫번째 자리에 20 삽입
				//begin() 역참조를 통한 첫 값 출력
				cout<<"첫 원소:" << *(A.begin())<<", 길이: " << A.size() << ", 용량: " << A.capacity() << endl;

				A.erase(A.begin());	//첫번째 원소 삭제
				//front()를 통한 첫 값 출력
				cout<<"첫 원소:" << A.front() <<", 길이: " << A.size() << ", 용량: " << A.capacity() << endl;

				A.assign(3,7); // 3 크기의 7로 찬 벡터로 재초기화
				//back()를 통한 마지막 값 출력
				cout<<"첫 원소:" << A.back() <<", 길이: " << A.size() << ", 용량: " << A.capacity() << endl;
			}
			```
	* ### 순회
		* 위 함수들과 접근법들을 이용해 다양한 순회 방법이 존재하겠으나, 벡터 전체 범위를 손쉽게 탐색하는데 있어서는 for문 혹은 향상된 for문(Enhanced for, foreach)를 이용하면 편하다.
			* 인덱스 기반 for문(Index-based for)
				* 가장 기본적인 for문을 사용한 방법.
				* 인덱스 값을 알면서 순회해야될 때 유용.
				* 벡터에 제공되는 반복자를 이용한 방식도 가능.
					* const iterator를 이용해 읽기 전용 탐색 가능.
					* rbegin(), rend()를 이용한 역방향 탐색 가능;
			* Foreach
				* 범위 기반 for 문(Range-based for)
					* 벡터 x에 있는 원소들을 하나하나 val로 반환받으며 모든 원소에 대해 반복문 실행
					* 참조가 아닌 복사하여 받기에 원래 배열에 수정사항이 반영되진 않음.
					* 인덱스를 추적할 수 없음.
				* 범위 기반 for 문  참조사용
					* 위 방식의 참조 버전.
					* 참조가 아닌 복사하여 받기에 원래 배열에 수정사항이 반영되진 않음.
					* 인덱스를 추적할 수 없음.
					
				```c++
				#include <iostream>
				#include <vector>
				using namespace std;
				int main(){
					vector<int> A = {1,2,3,4,5};
					//인덱스 기반, 1 2 3 4 5 출력
					for(int i = 0; i < A.size(); i++){
						cout << A[i]++ << " ";
					}
					cout << endl;
					
					//범위기반, 이전 반복에서 1씩 증가해 2 3 4 5 6 출력
					for(int i : A){
						cout << i++ << " ";
					}
					cout << endl;
					
					//범위참조기반, 이전 반복의 증가는 반영되지 않아 2 3 4 5 6 출력
					for(int& i : A){
						cout << i++ << " ";
					}
					cout << endl;
					
					//반복자 기반, 이전 반복에서 증가해 3 4 5 6 7 출력
					for(auto i = A.begin(); i != A.end(); i++){
						cout << *i << " ";
					}
					cout << endl;
				}
				```
	* ### 연속 메모리 사용 친화성
		* vector는 메모리상에서 끊기지 않고 연속된 주소에 데이터를 저장
			* 이는 insert와 erase가 더 큰 시간복잡도를 가지게 한다.
			* 대신 자료가 메모리에 흩어져있는 파편화를 일으키지 않는다.
			* 때문에 필요에 따라 다르지만 주된 사용법은 push/pop에 기반하면 빠른 작동속도를 보장한다.
		* CPU가 cache로 데이터를 불러올 때(캐싱할때) 메모리 주소상 주변에 있는 데이터도 함께 가져오며, 메모리 상의 연속성은 캐싱에 유리함을 만들어 성능을 향상
			* 캐쉬는 CPU가 쓸 자료를 미리 저장해놓는 매우 빠르고 작은 저장공간(SRAM)이다. 용량은 보통은 수십, 많게는 수백 KB 수준으로, 메모리(DRAM)보다 매우 작지만 훨씬 빠른 속도로 동작해 CPU의 클럭 단위 작동을 보장한다.
			* 때문에 함께 캐쉬로 불려오는 데이터가 함께 쓰일것이란 연관성을 바탕으로 함께 캐싱이 이루어진다.
			* 캐싱된 데이터가 실제로 사용된다면, 메모리까지 다시 접근하지 않아도 되기에 성능이 향상되며, 이를 캐쉬 히트(Cache hit)라고 표현한다.
			* 사용할 데이터가 캐쉬에 존재하지 않으면, 동작속도가 상대적으로 느린 메모리까지 접근해야 하며, 이는 성능 하락으로 이어진다. 이를 캐쉬 미스(Cache miss)라 표현한다.
			* 파편화되지 않은 메모리는 더 많은 사용가능공간을 확보시키는 한편, 캐쉬 히트 비율을 높여 성능 향상에도 기여한다.
* ### Vector의 활용
	* 기존의 정적 배열 사용을 메모리의 유연성과 다양한 기본 지원되는 함수들의 편의성을 챙기며 대체 가능.
	* 언리얼에선 UObject 호완을 위해 Tarray라는 다른 구조 사용, 기본적인 기능은 vector와 동일.
	* 웹개발 및 서버 개발에서 상품,  목록, 리스트를 vector에 담아서 사용 가능.
	* 다른 언어에서도 이름은 다르나 비슷한 기능들 존재
		* Python: list
		* Java: ArrayList
		* Rust: Vec
	* ### 간단한 vector를 이용한 게임 인벤토리 예시
		```c++
		#include <iostream>
		#include <vector>
		using namespace std;
		class Inventory{
		private:
			vector<string> items;
		public:
			//아이템 출력
			void ShowInventory(){
				if(items.empty()){
					cout << "아이템이 없습니다!" << endl;
					return;
				}
				cout << "가방" << endl;
				cout << "====================" << endl;
				for(auto i = items.begin(); i != items.end(); i++){
					cout << ">> " << *i << endl;
				}
				cout << "====================" << endl;
			}
			//아이템 사용
			void UseItem(string item){
				//뒤에 가까운 아이템을 삭제하면 더 빠르니 rbegin/rend 사용
				for(auto i = items.rbegin(); i != items.rend(); i++){
					if(*i == item){
						items.erase(i.base()-1);
						cout << "아이템 " << item << " 을 사용했습니다!" <<endl;
						return;
					}
				}
				cout << "아이템이 존재하지 않습니다!" << endl;
			}
			//아이템 추가
			void AddItem(string item){
				items.push_back(item);
				cout << "아이템 " << item << " 이 추가되었습니다!" <<endl;
			}
			
			//아이템을 이름 오름차순으로 정렬
			void SortItem(){
				cout << "인벤토리를 정렬합니다!" <<endl;
				for(auto i = items.begin(); i != items.end() - 1; i++){
					auto lowest = i;
					for(auto j = i + 1; j != items.end(); j++){
						if(*j < *lowest) lowest = j;
					}
					if(i == lowest) continue;
					string temp = *i;
					*i = *lowest;
					*lowest = temp;
				}
			}
		};
		int main(){
			Inventory X;
			X.ShowInventory(); //아이템 없음!
			X.AddItem("Poisionous Potion");
			X.AddItem("Magic Scroll");
			X.AddItem("Poisionous Potion");
			X.ShowInventory(); //아이템 출력
			X.UseItem("Magic Scroll");	//독포션 1개 제거
			X.ShowInventory(); //아이템 출력
		}
		```