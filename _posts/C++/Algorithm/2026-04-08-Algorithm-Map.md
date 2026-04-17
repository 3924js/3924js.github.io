---
title: "C++: 맵(Map)/세트(Set)"
author: Jaeseong Kim
date: 2026-04-08 12:00:00 +0800
categories: [C++, Algorithm]
tags: [C++, Algorithm, Map, Set]
---
* ## 맵(Map)
	* Key와 그에 대응되는 value, **데이터 쌍의 집합으로 이루어진 컨테이너**다.
		* 내부 작동은 std::pair를 자동 정렬 이진트리의 한 종류인 red-black tree에 넣어놓은 구조로, Key값의 크고 작음에 따라 자동으로 정렬된다.
	* 여러 기본적인 사용법이 존재한다.
		* map[key]: key값에 해당하는 value를 반환한다. 없으면 기본값인 0을 반환한다. 배열의 인덱스 접근자처럼 값의 수정도 가능하다.
			* 단순 탐색이 목적이고 map을 변형시키면 안된다면, 이 방법은 [key, 0]라는 쌍을 새롭게 추가해 길이를 늘리게 되니 조심하자. 그때는 count를 사용하는게 더 좋다.
			* 근데 카운팅이 목적이라면 map[n]++; 처럼 사용할 때 별도의 코드 없이 map으로 카운팅 할수있어 오히려 편리하다.
		* insert({key, value}): key, value 쌍을 추가한다.
		* erase(key): key에 해당하는 쌍을 map에서 제거한다.
		* find(key): key에 해당하는 위치의 반복자를 반환한다.
		* count(key): key가 있으면 1, 없으면 0을 반환한다.
		* clear(): 내용물을 모두 없애 비운다.

```c++
#include <string>
#include <map>
#include <iostream>

using namespace std;

int main()
{
    //map<key자료형, value 자료형>, 선언과 동시에 초기화도 가능
    map<string, int> HR = {% raw %}{{"CEO", 1}}{% endraw %};

    //[]를 이용한 값 추가, 뒤죽박죽 넣어도 순서대로 정렬된다.
    HR["IT"] = 3;
    HR["Cook"] = 5;
    
    
    //반복문을 이용한 출력
    for (auto Resources : HR) {
        cout << Resources.first << ": " << Resources.second << endl;
    }
    cout << endl;

    //insert를 사용한 추가
    HR.insert(pair <string, int>("Security", 3));
    HR.insert({"Inbound", 2});

    //반복자를 이용한 출력
    for (auto i = HR.begin(); i != HR.end(); i++) {
        cout << i->first << ": " << i->second << endl;
    }
    cout << endl;

    //erase를 이용한 삭제
    HR.erase("IT");

    //반복문을 이용한 출력
    for (auto Resources : HR) {
        cout << Resources.first << ": " << Resources.second << endl;
    }
    cout << endl;

    //find를 이용한 검색
    auto temp1 = HR.find("IT"); //IT 존재하지 않음
    if (temp1 != HR.end()) {
        cout << temp1->first << ": " << temp1->second << endl;
    }
    else {
        cout << "Not Found!" << endl;
    }
    temp1 = HR.find("Cook");   //Cook 검색 결과 출력
    if (temp1 != HR.end()) {
        cout << temp1->first << ": " << temp1->second << endl;
    }
    else {
        cout << "Not Found!" << endl;
    }
    cout << endl;

    //count를 이용한 존재여부 확인
    cout << "Inbound: " << HR.count("Inbound") << endl;
    cout << "CS: " << HR.count("CS") << endl;
    cout << endl;

    //없는 키값 []호출시 size증가
    cout << "size: " << HR.size() << endl;
    cout << "CS: " << HR["CS"] << endl;
    cout << "size: " << HR.size() << endl;
    
		//clear를 통해 비울 수 있음
		HR.clear();
		for (auto Resources : HR) {	//출력할 값 존재하지 않음.
        cout << Resources.first << ": " << Resources.second << endl;
    }
}
```

* **key와 value는 모든 자료형을 쓸 수 있다.** 다만 key값에 사용자가 정의한 클래스나 비교 연산자가 기본정의되있지 않은 자료형을 사용하고자 한다면, **비교연산자 '<'를 정의** 해주어야한다. 어떤 값을 바탕으로 내부 트리를 정렬해야할 지 알려줘야 하기 때문이다.

```c++
#include <string>
#include <map>
#include <iostream>

using namespace std;
class A {	
public:
    int num;
    A(int val) : num(val) {

    }

    bool operator < (const A& other) const {//비교연산자 정의
        return num < other.num;
    }
};
int main()
{
    //map<key자료형, value 자료형>
    map<A, int> Map;
		//뒤죽박죽 넣어도 3 5 13 50으로 정리
    Map[A(13)] = 1;
    Map[A(5)] = 1;
    Map[A(3)] = 1;
    Map[A(50)] = 1;

    for (auto x : Map) cout << x.first.num << ": " << x.second << endl;
}
```
* ## 셋(Set)
	* map에서 값이 존재하지 않는, 키값만으로 이루어진 컨테이너이다. 
	* 키에 대응되는 별도의 값이 존재하지 않기에, 같은 키값이 주어지면 중복으로 추가되지 않는다.
	* 더 자세히 들어가면 그냥 이진 트리, 그중에서도 Red-Black tree와 같다. 때문에 값이 추가되고 삭제되는 것에 따라 자동으로 정렬된다.
	* insert, count, erase등 **기본적인 사용은 map과 유사**하다. 위에 언급한 모든 함수들은 set에서도 작동한다.
	
	```c++
	#include <string>
	#include <set>
	#include <iostream>

	using namespace std;

	int main()
	{
	    //set<자료형> 이름, 선언과 함께 초기화 가능
	    set<string> Items = {"Potion", "Sword", "Apple"};


	    //반복문을 이용한 출력
	    for (auto item : Items) {
	        cout << item << endl;
	    }
	    cout << endl;

	    //insert를 사용한 추가
	    Items.insert("Bow");
	    Items.insert({ "Inbound", "Crossbow"});  //배열로 여러개 추가도 가능

	    //반복자를 이용한 출력
	    for (auto i = Items.begin(); i != Items.end(); i++) {
	        cout << *i << endl;
	    }
	    cout << endl;

	    //erase를 이용한 삭제
	    Items.erase("Sword");

	    for (auto item : Items) {
	        cout << item << endl;
	    }
	    cout << endl;

	    //find를 이용한 검색
	    auto temp = Items.find("Armor"); //Armor 존재하지 않음
	    if (temp != Items.end()) {
	        cout << *temp << endl;
	    }
	    else {
	        cout << "Not Found!" << endl;
	    }
	    temp = Items.find("Apple");   //Apple 검색 결과 출력
	    if (temp != Items.end()) {
	        cout << *temp << endl;
	    }
	    else {
	        cout << "Not Found!" << endl;
	    }
	    cout << endl;

	    //count를 이용한 검색
	    cout << "Potion: " << Items.count("Potion") << endl;
	    cout << "Boots: " << Items.count("Boots") << endl;
	    cout << endl;

			//clear를 통한 초기화
			Items.clear();
			for (auto item : Items) {	//출력할 내용 없음
	        cout << item << endl;
	    }
	}
	```
* ## Unordered Map/Set
	* 기존의 자동 균형 트리의 사용으로 있던 **정렬 기능이 빠진 map/set**이다.
	* Red-Black tree 대신 **해시 테이블**을 사용하는데, 정렬 상태를 보장하지 못하는 대신 랜덤접근의 속도가 트리의 $O(log n)$ 에서 해시의 **"평균" $O(1)$** 로 많이 좋아진다.
		* 하지만 같은 해시값이 존재하는 **해시 충돌시에는 최악의 케이스일 때 O(n)**으로  기존 트리구조보다 느리다. 때문에 성능이 더 중요하면 커스텀 해시함수를 작성해 사용하거나, 일반 map구조가 더 좋을 수 있다.
	* 정렬 상태가 필요 없고(순회시 순서가 상관이 없고) 자주 검색할 일만 많다면 오히려 좋은 선택지일 수 있다.
	* 사용법은 map/set과 동일하다.
* ## 활용
	* 게임의 아이템, 퀘스트 목록, 환경설정등 데이터를 관리하는데 유용하게 쓰인다.
		* 인벤토리를 구현할 때 아이템을 모두 보관하는 벡터가 아닌, 아이템의 ID와 갯수로 이루어진 map, 그리고 그 ID로 이루어진 아이템 정보 테이블로 구성하면 다양한 아이템을 필요로하는 MMORPG같은 환경에 잘 대응할 수 있다.
		* 언리얼에서는 TMap, TSet이라는 대응되는 컨테이너를 제공한다.
	* 웹/서버 개발에서도 key-value구조의 데이터를 저장할 때 쓰인다.
		* 여러 분야에서 보편적으로 데이터 저장의 쓰이는 JSON도 같은 구조를 가지고있다.
		* 서버의 캐시로 쓰이는 Redis도 해시 테이블 구조이다.
		* URL 쿼리, HTTP 헤더, 쿠키등 여러 데이터도 모두 key-value의 형태를 띄고있다.
	* 컴퓨터 과학 전반에 있어서도, 데이터와 숫자의 연결이라는 구조는 기초적인 자료구조로 쓰인다.
		* DB의 인덱스
		* 무결성 검증용 체크썸(Check Sum)
		* 암호
		* 블록체인