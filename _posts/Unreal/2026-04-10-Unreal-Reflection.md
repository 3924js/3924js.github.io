---
title: "Unreal: 리플렉션(Reflection)/UHT/UBT/CDO"
author: Jaeseong Kim
date: 2026-04-10 12:00:00 +0800
categories: [Unreal, Basics]
tags: [Unreal, Basics, Reflection, UHT, UBT, CDO]
---
* ## 리플렉션(Reflection)
	* **실행중에 프로그램이 자기자신의 상태를 인식하고 동작에 반영할 수 있는 능력**, 메타 프로그래밍의 일종으로 대신해서 불리기도 함.
	* 언리얼 리플렉션은 기본 **C++프로젝트를 언리얼 엔진이 인식**할 수 있도록 메타데이터로 이어주는 역할도 겸함.
		* 런타임 타입 정보 확인
		* 언리얼 에디터 연동
		* GC 대상으로 등록(기본 C++와 다르게 언리얼 엔진에서 GC 지원, 메모리 누수 방지)
			* UObject로 등록되지 않은 자료에 대해서도 안전을 보장하기 위해 TSharedPtr, TWeakPtr같은 기능 제공, UE5부턴 표준적으로 사용이 권장됨.
* ## 언리얼 빌더 툴(Unreal Builder Tool, UBT)
	* 가장 먼저 실행되는 빌드 도구로, **프로젝트의 구조를 파악하고 플랫폼에 따라 환경을 설정**한다.
		* 플러그인/모듈 검색
		* .target.cs 확인
		* .build.cs 처리
		* 플랫폼 호환성 확인
		* 개발 환경 자동 구성
* ## 언리얼 헤더 툴(Unreal Header Tool, UHT)
	* 컴파일 전에 미리 파일들을 전처리해주는 툴
	* 메크로로 달아놓은 메타데이터를 수집하고 그에 따른 파일들을 자동으로 생성/수정한다.
		* GENERATED_BODY()
		* UCLASS()
		* UPROPERTY()
		* UFUNCTION()
* ### 빌드과정
	* **UBT -> UHT -> VS -> 리플렉션 완성**
	* 위 빌드 프로세스 때문에 라이브코딩 만으로는 프로젝트에 수정사항이 제대로 반영되지 않는 때가 있다.
	* **.h 변경시엔 전체 리빌드**
	* **.cpp파일 변경시엔 라이브 코딩**
		* generated.h파일 재생성 필요 없으니 UHT 처리와 컴파일 건너뜀)
	* 회사나 대규모 프로젝트라면, 빌드 자동화와 테스트 자동화등 제작 과정을 도와주는 언리얼 호드(Unreal Horde)를 구축해 사용하기도 한다. 언리얼 빌드 엑셀러레이터(Unreal Build Accelerator)를 이용해 빌드과정도 다른 컴퓨터에서 진행하게 해주는등 원격 실행 기능도 들어있다.

* ## CDO(Class Default Object)
	* **UObject 클래스에 하나씩 존재하는 기본 객체**로, 재사용 가능한 설계도로서 엔진이 **인스턴스를 생성할 때 사용**된다.
	* 컴파일 후 엔진이 켜질 때, 모듈 로딩이 끝난 후 엔진에 등록되면서 생성된다.
	* CDO 생성자는 한번만 호출되고, 이후의 객체는 CDO의 복사를 통해 인스턴스를 생성한다.
		* 메모리 절약: 공통되는 데이터는 CDO에 저장하고 다른 데이터만 각 인스턴스에 저장
		* 빠른 초기화: 생성자 호출 없이 메모리만 복사
		* 델타 직렬화: 차이점만 기록/전달하여 저장/네트워크 효율 향상
	* CDO의 값을 실행 중에 변경할 일은 별로 없고, 권장되지도 않는다. 런타임에 CDO가 변경되었을 때 실행 결과에 어떤 일을 미칠지는 환경과 구조에 따라 달라서 예측하기 힘들다. 다음코드는 MyActor에서 UPROPTER() int 로 선언된 맴버 변수 Health의 변화를 보여준다.

	```c++
	#include "MyGameModeBase.h"
	#include "MyActor.h"

	void AMyGameModeBase::BeginPlay()
	{
		Super::BeginPlay();
		//CDO 바탕으로 ActorA 생성, 초기값 100 출력
		AMyActor* ActorA = GetWorld()->SpawnActor<AMyActor>();
		UE_LOG(LogTemp, Warning, TEXT("ActorA Init Health: %d"), ActorA->Health);
	
		//ActorA 의 체력 수정, 변경된값 50 출력
		ActorA->Health = 50;
		UE_LOG(LogTemp, Warning, TEXT("ActorA mod Health: %d"), ActorA->Health);
		
		//CDO 접근, 초기값 100출력
		AMyActor* ActorCDO = GetMutableDefault<AMyActor>();
		UE_LOG(LogTemp, Warning, TEXT("CDO Init Health: %d"), ActorCDO->Health);
		
		//CDO 값 수정, 변경된 200출력
		ActorCDO->Health = 200;
		UE_LOG(LogTemp, Warning, TEXT("CDO mod Health: %d"), ActorCDO->Health);

		//CDO를 바탕으로 새로운 객체 ActorB 생성, CDO에서 변경됬어도 100출력
		AMyActor* ActorB = GetWorld()->SpawnActor<AMyActor>();
		UE_LOG(LogTemp, Warning, TEXT("ActorB Init Health: %d"), ActorB->Health);

		//기존 객체 ActorA의 값도 이후 수정된 CDO의 영향을 받지 않음, 50출력
		UE_LOG(LogTemp, Warning, TEXT("ActorA Health: %d"), ActorA->Health);
	}
	//첫번째 실행시 100 50 100 200 100 50
	//종료후 두번째 실행시 100 50 "200" 200 100 50
	```
	다만 엔진을 수정하거나 특수한 기능을 만드는 일이 아닌, 일반적인 기능을 구현하는데 있어서 CDO를 직접적으로 수정할 일은 없을것이다. 보통 그런상황까지 간다면, 설계 구조가 무언가 잘못되었다는 말일텐데 런타임에 값에 따라 클래스가 달라지도록 재설계로 충분히 해결될 일이지, CDO를 수정하여 엔진 동작의 잠재적인 오류의 여지를 남기는건 좋지 않을 것이다.
	
## 나쁜 코딩 습관
1. 기이한 이름
	* 코드를 명료하고 직관적으로 표현하는 것은 이름인데, 변수/클래스/함수등이 이름으로 판단하기 힘들어지면 설계가 잘못된것일 수 있다.
	* 특히 명확한 이름이 떠오르지 않는 설계 과정이라면 방향성이 잘못된 것일 수 있다.

	```c++
	//어떤 목적을 가지는지 알 수 없음
	void DoSomething(string x);	
	float A;
	int x;

	//이름만으로 기능과 목적성을 판단할 수 있음
	void PrintString(string Message);
	float ColorValue;
	int RepeatTimes;
	string EnemyName;
	```
2. 중복 코드(Duplicated Code)
	* 중복된 구조로 인한 단순 복사-붙여넣기는 개발시간을 더 많이 소비하게 된다.
	* 부분적으로 비슷한 코드들은 공통되는 부분 먼저 정리해서 차근차근 분리하는게 좋다.
		* 클래스/함수/구조체/상속 관계 형성/템플릿 등등
3. 긴 함수(Long Function)
	* 짧은 함수는 쉽게 파악 가능하지만, 긴 함수는 이해하기도 힘들고 공유하기 편함.
	* 주석이 필요하다고 생각되면 별도의 함수로 만들려고 해보자. 길어지면 주석과 내용이 일치하지 않을 수 있다.