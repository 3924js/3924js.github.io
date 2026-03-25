---
title: "C++: 문자열(String)"
author: Jaeseong Kim
date: 2026-03-25 12:00:00 +0800
categories: [C++, Algorithm]
tags: [C++, Algorithm, String]
---
## 문자열(String) 클래스
* C++에서 char* 대신 사용할 수 있는 문자열 자료형으로, string 라이브러리에 클래스로 구현되어 있다.
* 내부는 벡터와 유사하며, 차이점은 모든 자료형 대신 char만 담을 수 있다는 점이다.
* 내부적으로는 std::pair를 이진 트리(Binary Search Tree)형태로 엮어놓은 형태이다.
	* 정확히는 Red-Black 트리라는 자가 균형 트리(Self-Balancing BST)로, 필요에 따라서는 트리 구조를 구현하는 대신 map컨테이너를 사용하는 것도 좋은 선택이다.
## 지원 함수
* find()
	* 문자열 안에 해당 검색어가 포함됬는지 검색해 그 위치에 해당하는 인덱스를 반환한다.
	* 없을 경우 npos를 반환한다.
		* string::npos로 찾지 못한 상태를 나타낸다.
		* npos가 나올 수 있는 기능을 구현할 경우 예외 처리를 해주지 않으면 런타임 에러가 날 수 있다.
	find("a", 7) //인덱스 7부터 검색, O(nxm) 원본 길이, 검색어 길이
* substr()
	* s.substr(시작위치, 길이)
	* 일부분을 새 문자열로 반환
	* 길이가 없으면 시작위치부터 끝까지
* find와 substr를 사용하여 파일 확장자 분리에 많이 쓰임
* replace
	* s.replace(시작위치, 길이, "새문자열")
	* 원본을 직접 "하나만 수정"
* find에서 npos반환할때까지 반복하면 전부 대체 가능
* stringstream
	* 문자열을 여러 조각으로 분리하기 위한 자료형
	* 문자열을 cin처럼 처리해 하나씩 꺼낼 수 있음
* 변환
	* stoi
	* stof
	* stol
	* stod
	* to_string
	* 
	* 글자당 사이즈 다름
	* 채팅필터 간단히 find/replace로 가능하지만 실제로는 그러면 너무 느림 -> 아호코라식
	* parser
	* 문자는 숫자: ascii수준의 문자와 한글 문자는 사이즈가 다름 (ascii 7비트, 유니코드 utf8 최대 4바이트
* 활용
	* 게임개발
		* 채팅 필터링(금칙어 -> 별표), 세이브로드(직렬화 -> 파싱)
		* UE5 의 FString - find(), mid(), Replace()
	* 웹/서버
		* URL 파싱(호스트, 경로, 퀘리 분리), JSON/XML 파싱
		* 서버 로그 분석 - find + substr + stringstream 이 기초 패턴
	* 일반
		* 문자 인코딩 = 운영체제 네트워크 데이터베이스 모두 영향
		* 나중에 배울 해쉬에서도 문자 = 숫자가 핵심