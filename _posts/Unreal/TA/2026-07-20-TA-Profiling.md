---
title: "TA: 언리얼 프로파일링"
author: Jaeseong Kim
date: 2026-07-20 08:00:00 +0800
categories: [Unreal, TA]
tags: [Unreal, TA, Profiling]
---
## 프로파일링(Profiling)
프로그램의 실행속도, 안정성을 보장하기 위해 메모리 사용량이나 프로세서 점유율등 성능 최적화의 목적으로 행해지는 프로그램 분석을 프로파일링이라 부른다.

게임을 엔진없이 DirectX같은 그래픽 언어로 스크래치부터 구현하고 물리, 상호작용등을 손수 만들어야 했던 시절에는 구현 방식과 사용 기술에 있어서 성능에 대한 고려와 검증은 자연스럽게 따라오는 플로우의 일부였다. 

지금도 일부라고는 할 수 있지만, 과거와 다른 점이라면 엔진에서 모든 기초적인 기능을 제공해주기 때문에 "먼저 만들고 나중에 최적화한다"라는 방식이 통하게 된다는 점이다. 더군다나 전반적인 게임플레이 기기들의 성능이 상향평준화 되면서, 과거라면 돌아가는게 이상할 수준의 그래픽/연산 부하는 감당 가능한 수준으로 들어선 감이 강하다. 이는 인디게임이나 중소게임사에서 유저들의 기기 성능을 덜 신경써도 되는, 흔히 말하는 발적화인 상태로 나오더라도 문제가 덜한 상황을 만들어냈다.

다만 이는 최적화가 필요하지 않다는 말은 아니다. 최적화가 개발의 일부로 유기적으로 포함되는 구조에서, 독립적으로 가져갈 수 있는 단계로 분화했다고 봐야 할 것이다. 즉 프로파일링의 중요도가 가볍고 소규모인 게임 프로젝트에선 중요도가 크게 낮아지긴 했어도, 동작하고 사용가능한 프로그램으로서 게임을 개선시켜주는 중요한 단계이다. 평소 랙이나 성능 문제로 게임을 즐기지 못했던 기억들을 떠올려 본다면 이를 체감할 수 있을 것이다.

## 방법
* ### [Stat: 언리얼 통계 명령어](https://dev.epicgames.com/documentation/unreal-engine/stat-commands-in-unreal-engine?lang=ko)
    ![Stat](assets/img/260720-stat.png)

    * 엔진의
    * 언리얼 엔진 기준으론 인게임중에 그레이브(`, 탭키 위에 문자)를 눌러 명령어를 입력하면 확인할 수 있다.
	* stat fps
        ![Stat](assets/img/260720-fps.png)

		* **프레임이 얼마나 나오는지**를 확인한다.
		* 성능의 기본적인 지표로 활용할 수 있다.
	* stat unit
        ![Unit](assets/img/260720-unit.png)

		* 한 프레임을 계산하는데 **어느 부분에서 얼마나 걸리는지**를 확인한다.
		* 어느 구간에서 많이 성능을 잡아먹는지 큰 범주에서 좁힐 수 있다.
			* game에서 시간이 많이 먹는다 -> Tick 호출, 로직등 CPU 연산이 필요한 작업들을 점검한다.
            * draw에서 시간을 많이 먹는다 -> 드로우콜 횟수, 컬링의 효율등 그래픽 명령 제출 과정에 관련된 것들을 확인한다.
			* gpu에서 시간이 많이 먹는다 -> 애니메이션, 텍스처, 쉐이더등 GPU연산이 필요한 작업들을 확인한다.
	* stat GPU
        ![Unit](assets/img/260720-gpu.png)

		* **GPU에서 무엇을 연산하는지, 렌더링 패스(Rendering Path)**를 확인한다. stat unit에서 GPU에 뜨는 시간이 유독 크다면 확인해볼 법하다.
* ### [언리얼 인사이트](https://dev.epicgames.com/documentation/unreal-engine/unreal-insights-in-unreal-engine)
    ![Insight](assets/img/260720-insight.png)

	* 게임 플레이중에 어떤일이 있었는지 기록을 분석할 수 있게 도와주는 통합 프로파일링 툴이다.
    * 사용하기 위해선 원하는 게임 구간을 트레이스로 저장해야 한다.
	    * 트레이스를 시작하면 게임 플레이 도중 일어아는 일들을 로그와 통계 형태로 저장하기 시작한다.
        * 트레이스를 종료하면 따로 확인할 수 있는 하나의 정리된 트레이스로 저장된다.
        * 언리얼 인사이트에서 이 트레이스를 열면 그래프, 통계 형태로 게임에서 어떤 일이 있었는지 파악할 수 있다.
    
    ![Graph](assets/img/260720-trace.png)

	* 그래프에서 순간적으로 튀는 구간이 병목 지점일 확률이 높으며, 그 때 어떤 일들이 있었는지 확인하면 병목인 부분을 좁혀나갈 수 있다.
* ### [렌더독(RenderDoc)](https://renderdoc.org/)
	*	[언리얼 엔진 공식  RenderDoc 가이드](https://dev.epicgames.com/documentation/unreal-engine/using-renderdoc-with-unreal-engine?lang=ko)
	* 오픈소스 그래픽 디버거로, 어떤 순서로 그래픽이 그려지는지 순서와 레이어별로 분해해서 볼 수 있는 기능을 제공한다.
    * 언리얼에 기본 플러그인으로 내장되어 있으며, 그래픽 최적화 작업에서 렌더링 과정을 확인하기 위해 사용할 수 있다.
		* 어떤 Draw Call이 실행됐는지
		* 어떤 메시·머티리얼·텍스처가 사용됐는지
		* 셰이더에 어떤 값이 전달됐는지
		* 깊이, 스텐실, 블렌딩 상태가 무엇인지
		* 특정 픽셀이 어떤 렌더링 이벤트에 의해 만들어졌는지
		* 중간 렌더 타깃이 어떤 순서로 변했는지 등등

* ### ShowDebug: 동작 확인 명령어

    ![ShowDebug](assets/img/260720-showdebug.png)

    * 성능의 병목을 확인하는 Stat과는 다르게, 입력이나 동작을 확인한다.
    * ShowDebug EnhancedInput

        ![ShowDebug](assets/img/260720-enhanced.png)

        * EnhancedInput 기반 입력 정보를 보여준다.
    * ShowDebug Input

        ![ShowDebug](assets/img/260720-input.png)

        * 기존 입력 시스템에 기반해 어떤 키 입력이 있었는지등을 확인할 수 있다.
    * 외에도 애니메이션, 콜리전, AI동작등 동작이 어떤지 확인할 수 있다.

* ### 치트 만들기 - 게임이름 - 명령어 형식으로
    * 게임 테스트중 기존에 설계된 플로우를 따라가면 모든 상황을 테스트하기엔 오래 걸릴 수 있다. 혹은 필요한 정보를 디버깅을 위해 원하는 형태로 정리해서 확인해야 할 때도 있다. 때문에 필요한 아이템을 주거나 위치를 이동하거나, 몬스터 스폰 수나 처치 수 확인등 테스트에 필요한 기능들을 치트 명령어로 만들어주면 편리하다. 구현에는 다양한 방법이 있다.
    * `UCheatManager`
        * 가장 정석적인 방법으로 치트 기능을 하나의 함수로 모아서 클래스로 관리할 수 있다.
            ```c++
            // MyCheatManager.h

            #pragma once

            #include "CoreMinimal.h"
            #include "GameFramework/CheatManager.h"
            #include "MyCheatManager.generated.h"

            UCLASS()
            class MYGAME_API UMyCheatManager : public UCheatManager
            {
                GENERATED_BODY()

            public:
                UFUNCTION(Exec)
                void AddGold(int32 Amount);

                UFUNCTION(Exec)
                void SetInvincible(bool bEnable);

                UFUNCTION(Exec)
                void TeleportTo(float X, float Y, float Z);
            };
            ```

        * 콘솔에서 호출하기 위해서는 게임에서 사용할 플레이어 컨트롤러에 치트매니저 클래스로 등록해줘야 한다. C++나 블루프린트 에디터에서 설정할 수 있다.
            ```c++
            // MyCheatManager.cpp

            #include "MyCheatManager.h"
            #include "GameFramework/PlayerController.h"
            #include "GameFramework/Pawn.h"

            void UMyCheatManager::AddGold(int32 Amount)
            {
                UE_LOG(LogTemp, Display, TEXT("Gold added: %d"), Amount);

                // GameState, PlayerState, InventoryComponent 등에 전달
            }

            void UMyCheatManager::SetInvincible(bool bEnable)
            {
                UE_LOG(
                    LogTemp,
                    Display,
                    TEXT("Invincible: %s"),
                    bEnable ? TEXT("true") : TEXT("false")
                );
            }

            void UMyCheatManager::TeleportTo(float X, float Y, float Z)
            {
                APawn* Pawn = GetOuterAPlayerController()->GetPawn();

                if (IsValid(Pawn))
                {
                    Pawn->SetActorLocation(FVector(X, Y, Z));
                }
            }
            ```

        * 멀티플레이에선 서버 RPC로 호출하도록 구성해야 한다. 클라이언트에서 바뀐 정보는 서버에서까지 바뀌지 않는다.
        * Shipping 빌드에선 자동으로 배제된 후 빌드된다. 다만 안전을 위해 추가적으로 전처리기를 붙여 확실히 하는게 좋다.
            ```c++
            //MyCheatManager.h
            #if !UE_BUILD_SHIPPING

            UFUNCTION(Exec)
            void AddGold(int32 Amount);

            #endif
            ```
            ```c++
            //MyCheatManager.cpp
            #if !UE_BUILD_SHIPPING

            void UMyCheatManager::AddGold(int32 Amount)
            {
                // Debug cheat
            }

            #endif
            ```
    * `Exec` 키워드
        * 앞에 예시에서 보인 `Exec`키워드는 치트 매니저 클래스 외부에 일반적인 `UFUNCTION`에도 붙일 수 있다. 이 경우 이미 구현하고 사용하는 일반 함수를 콘솔에서 호출할 수 있게 한다.
        * 이미 구현해놓은 기능을 콘솔에서 쓸 수 있도록 노출하는 방식이기 때문에 간단하게 추가할 수 있는 방법이지만, 명령어가 많아지면 관리하기 어려울 수 있다.

    * `CVar`
        * 콘솔 변수로, 콘솔해서 조회/수정할 수 있는 전역값이다.
        * 데미지 배율 조절, 쿨타임 조절등 실시간으로 값을 변경하는데 적합하며, 이를 행동의 판단/사용값으로 연결해 치트처럼 행동 제어에 사용할 수 있다.
            ```c++
            //선언 예시
            #include "HAL/IConsoleManager.h"

            static TAutoConsoleVariable<int32> CVarEnemyInvincible(
                TEXT("MyGame.EnemyInvincible"),
                0,
                TEXT("Controls enemy invincibility.\n")
                TEXT("0: Disabled\n")
                TEXT("1: Enabled"),
                ECVF_Cheat
            );
            //위 예시는 콘솔에선 MyGame.EnemyInvincible 1 처럼 호출할 수 있다.
            //필요에 따라 분류/계층으로 명령어를 구성하면 호출하기 편하다.
            ```
    * 이외에도 IConsoleManager에 콘솔 명령어로 등록하거나, 디버그용 UI를 따로 제작/추가해 버튼등으로 호출하는등 여러 방법이 있다.

* ### [위젯 리플렉터](https://dev.epicgames.com/documentation/unreal-engine/using-the-slate-widget-reflector-in-unreal-engine?lang=ko)
    ![Widget](assets/img/260720-widget.png)
    * 사용하고 있는 위젯의 계층구조와 상태를 확인할 수 있다.
    * 화면에 많은 UI를 각각을 확인하기 힘들 때, 겹친 UI등을 확인해야 할 때 등에 편리하다.

* ### [렌더 리소스 뷰어](https://dev.epicgames.com/documentation/unreal-engine/render-resource-viewer-in-unreal-engine?lang=ko)
    ![Resource](assets/img/260720-resource.png)

	* 그래픽 메모리에 로드되어있는 리소스의 종류를 보여준다.
    * 어디서 VRAM이 많이 잡아먹히는지 확인할 수 있다.
	* 만약 가볍고 툰쉐이딩에 가까운 간단함 게임을 생각한다면 나나이트도 적합하지 않을 수 있다. 많은 폴리곤과 디테일을 연산하는데는 효율적이지만 기본적으로 잡아먹는 성능이 어느정도 있어 차라리 없이 만드는게 성능과 퀄리티를 모두 잡는 경우도 있으니 참고할 것.
    * 마찬가지의 이유로 언리얼에 기본으로 활성화 되어있는 루멘도 필요 없을 수도 있다. 광원이 많이 없고 그림자나 반사광도 간단한 툰 스타일의 경우 불필요할 수 있다.

* ### 기타
    * 언급된 디버깅 툴이나 방법들 외에도 다양한 디버거를 언리얼에서 기본적으로 제공한다. 필요에 따라 활용해 성능과 실행을 분석하고 최적화하면 된다.
    * 뷰포트 엔진 퀄리티
        ![Viewport](assets/img/260720-viewport.png)

        * 에디터에서 게임을 돌려볼 때 뷰포트에 렌더링되는 퀄리티를 조절할 수 있다.
        * 단순히 기능의 구동만 테스트할 때나 컴퓨터 성능탓에 에디터 조작에 랙이 걸릴때 설정해보면 도움될 수 있다.
    * 엔진 디버깅
        ![Engine](assets/img/260720-engine.png)

        * 디버깅을 할 때 엔진 코드까지 들어가고 싶으면 `디버깅을 위한 에디터 심볼`을 선택해놓아야 한다. 에디터 외부, 에픽 게임즈 런처에서 설정할 수 있다.

## TA 분반 마무리 보너스 - 취업은 타이밍, 선택지는 다양
* 노력한다고 꼭 취업할 수 있는건 아니다. 시장이 차가울때는 노력해도 안되고, 풀릴때는 안될 사람들까지 취업된다.
* 공부하면서 기다리는 것도, 취직에 더 집중하는 것도, 창업으로 노선을 트는 것도 모두 선택지가 될 수 있다. 이는 개인에 달려있다.
    * 개인/인디 개발이라면 AtoZ 다 할줄 아는게 좋고, 취업 목적이면 한 분야에 특화되는게 좋다.
* 확실한건 **기회가 왔을 때 잡을 수 있을 만큼 준비**는 되어 있어야 한다.