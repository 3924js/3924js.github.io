---
title: "C++: 문자열(String)"
author: Jaeseong Kim
date: 2026-03-25 12:00:00 +0800
categories: [C++, Algorithm]
tags: [C++, Algorithm, String, String Stream]
---
* ## 문자열(String) 클래스
	* C++에서 char* 대신 사용할 수 있는 문자열 자료형으로, string 라이브러리에 클래스로 구현되어 있다.
	* 내부는 벡터와 유사하며, 차이점은 모든 자료형 대신 char만 담을 수 있다는 점이다.
		```c++
		#include <iostream>
		#include <string>
		using namespace std;
		int main() {
			string A; //String 생성
			string B = "Hello"; //생성과 함께 초기화
			char temp[] = "Hi"; //Char 배열을 이용한 초기화
			string C = temp;
			char* ptr = temp;	//Char 포인터를 이용한 초기화
			string D = ptr; 
			
			cout << D << endl;
		}
		```
* ## 지원 함수
	* find()
		* 문자열 안에 해당 검색어가 포함됬는지 검색해 그 위치에 해당하는 인덱스를 반환한다. 두번째 인자로 검색을 시작할 index를 지정할 수 있다.
		* 없을 경우 string::npos를 반환한다.
			* string::npos는 찾지 못한 상태를 나타낸다.
			* npos가 나올 수 있는 기능을 구현할 경우 예외 처리를 해주지 않으면 런타임 에러가 날 수 있다.
		*  해당 범위에 대해 모든 위치에서 검색어가 존재하는 지 확인한다. 즉, 원본의 길이를 n, 검색어의 길이를 m이라 하면 $O(n *m)$의 시간 복잡도를 가진다.
			```c++
			#include <iostream>
			#include <string>
			using namespace std;
			int main() {
				string A = "On the other hand"; //생성과 함께 초기화

				size_t temp = A.find("hello");	//없으니 string::npos 반환
				if (temp != string::npos) {
					cout << temp << endl;
				}
				else {
					cout << "does not exists in the string!" << endl;
				}
				temp = A.find("other");	//other이 시작하는 인덱스 반환
				if (temp != string::npos) {
					cout << temp << endl;
				}
				else {
					cout << "does not exists in the string!" << endl;
				}
				temp = A.find("n");	//On에 있는 첫번째 n의 인덱스 반환
				if (temp != string::npos) {
					cout << temp << endl;
				}
				else {
					cout << "does not exists in the string!" << endl;
				}
				temp = A.find("n", 3);	//인덱스 3 이후로 검색, hand에 있는 첫번째 n의 인덱스 반환
				if (temp != string::npos) {
					cout << temp << endl;
				}
				else {
					cout << "does not exists in the string!" << endl;
				}
			}
			```
	* substr()
		* s.substr(시작위치, 길이)
		* 일부분을 **새 문자열**로 반환
		* 길이가 없으면 시작위치부터 끝까지
			```c++
			#include <iostream>
			#include <string>
			using namespace std;
			int main() {
				string A = "On the other hand";

				string temp = A.substr(7);	//인덱스 7이후의 문자열 반환, he other hand
				cout << temp << endl;

				temp = A.substr(7, 5);	//인덱스 7에서 5개 길이의 문자열 반환, other
				cout << temp << endl;

				temp = A.substr(A.find("hand"));	//find와 조합하여 사용, hand
				cout << temp << endl;
			}
			```
		* find와 substr를 사용하여 파일 확장자 분리나에 많이 쓰임
			```c++
			#include <iostream>
			#include <string>
			using namespace std;
			int main() {
				string A = "260326_SamplePost_Subtitle";

				size_t index = A.find('_');
				string token = A;
				while (index != string::npos) {
					cout << token.substr(0, index) << endl;	//토큰 출력
					token = token.substr(index+1);	//남은 문자열로 이동
					index = token.find('_');
				}
				cout << token.substr(0, index) << endl;	//마지막 토큰 출력
			}
			```
	* replace()
		* s.replace(시작위치, 길이, "새문자열")
		* 원본을 **직접** "하나만 수정"
			```c++
			#include <iostream>
			#include <string>
			using namespace std;
			int main() {
				string A = "On the other hand";
				cout << A << endl;	//on the other hand
				A.replace(0,5, "qqqq");	//인덱스 0부터 5문자를 qqqq로 변환
				cout << A << endl;	//qqqq other hand
			}
			```
		* find와 조합하여 npos반환할때까지 반복하면 전부 대체 가능
			```c++
			#include <iostream>
			#include <string>
			using namespace std;
			int main() {
				string A = "left right left up down left";
				while (A.find("left") != string::npos) {
					A.replace(A.find("left"), 4, "FOUND!");
					cout << A << endl;
				}
			}
			```
	* stringstream
		* string을 스트림처럼 다룰 수 있게 해주는 자료형으로, sstream 라이브러리를 포함하여 사용할 수 있다.
			* 읽기 전용인 istringstream, 쓰기 전용인 ostringstream도 존재한다.
		* 문자열을 cin, cout을 사용할 때 처럼 스트림(Stream)으로 다룰 수 있게 해준다.
			```c++
			#include <iostream>
			#include <string>
			#include <sstream>
			using namespace std;
			int main() {
				string str = "100 200 300 400";
				stringstream ss(str); //다른 문자열로 스트링스트림 초기화
				cout << "ss: " << ss.str() << endl;

				//스트링 스트림에서의 토큰 출력
				int A[4];
				ss >> A[0] >> A[1] >> A[2] >> A[3];
				cout << "ss: " << ss.str() << endl;
				cout << "A[0]: " << A[0] << endl;
				cout << "A[1]: " << A[1] << endl;
				cout << "A[2]: " << A[2] << endl;
				cout << "A[3]: " << A[3] << endl;

				//스트링 스트림으로 토큰 입력
				ss.str("");	//버퍼 초기화, 내용물 없음
				ss.clear();	//플래그 초기화, 입력 가능
				ss << 500 << " " << 600;
				cout << "ss: " << ss.str() << endl;

				//반복문으로 끊어서 토큰 읽기
				string token;
				while (getline(ss, token, ' ')) {
					cout << "token: " << token << endl;
				}

			}
			```
	* 변환
		* stoi: string => int
		* stof: string  => float
		* stol: string => long
		* stoll: string => long long
		* stod: string => double
		* to_string: 다른 자료형 => string
	* string에 저장되는 문자는 ascii수준의 문자와 한글같이 유니코드를 사용하는 문자의 사이즈가 다르다. (ASCII: 7비트, 유니코드 UTF-8: 최대 4바이트)
* ##  활용
	* 온라인 게임이나 채팅 앱에서 흔히 쓰일법한 **채팅 필터링**에 쓰일 수 있다. 다만 위 find() 함수는 시간복잡도 탓에 적합하지 않고, 트라이(Trie, 이진 트리의 3갈래 버전)을 사용하는 방식인 아호코라식(Aho-Corasick)을 사용하면 더 빠르게 구현할 수 있다.
	* **언리얼 엔진**에서는 string에 대응되는 자료형인 **FString**이 존재한다.
		* 위에 나왔던 함수들에 대응되는 Find(), Mid(), Replace() 외에도 Left(), Right()등 다른 기능 함수들도 지원한다.
	* **JSON,XML**등으로 저장된 게임 세이브 데이터, 작업용 데이터, 혹은 **URL**을 통해 전달되는 웹사이트 정보나 쿼리등을 저장하고(**직렬화**) 불러오는데(**파싱**) 사용될 수 있다.
	* **서버의 로그 분석**에서는 find, substr, stringstream을 이용한 로그 탐색과 분석이 기초적인 패턴이다.
	* 운영체제와 네트워크, DB등에서도 **문자의 인코딩**은 기본적인 기능으로서 위치한다. 인코딩이 깨져 상황을 이해할 수 없는 상황은 안좋은 사용자 경험을 초래하고 에러에 취약하게 만든다.
	* 해쉬(Hash)와도 문자를 숫자로 일대일 대응하여 사용한다는 점에서 비슷할 수 있으나, 목적이 문자열 저장인지 아니면 빠른 시간복잡도의 보장인지에서 갈린다.
