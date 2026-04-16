---
title: "TrickyDroneDelivery: 드론 움직임 개선하기"
author: Jaeseong Kim
date: 2026-04-16 12:00:00 +0800
categories: [Unreal, Project]
tags: [Unreal, Project, NBC, PawnMovementComponent, FloatingPawnMovement]
---
## 기존 코드의 문제점
NBC 7번 과제에서 드론의 움직임은 손수 구현했었다. 직접 방향을 계산하고 내부에 속도를 저장해놓고 관성처럼 Lerp를 이용해 보간하는 방식으로 구현했는데, 이는 추후의 기능 추가 과정에서 몇가지 순차적인 문제를 만들었다.
1. 바닥 충돌의 모호함: 기존 코드는 lineTrace를 이용해 바닥면과의 거리를 확인 후 속도를 조절하는 방식을 사용한다. 때문에 LineTrace지점인 드론의 중심 아래가 아니라 모서리, 옆면등에 있는 사물은 인식을 못하고 지나간다.
	* Sweep으로 바꾸면 단순 선이 아닌 도형으로 Trace를 진행하기에 모서리나 다른 부위에서의 하단 충돌도 체크할 수 있다.
2. 바닥이 아닌 옆면 충돌: 1번 문제는 어느정도의 해소 여지가 있지만, 여전히 벽면/물체의 옆으로 충돌할 때는 이를 검사할 수 없다. 
	* AddActorLocalOffset()을 실행할 때, bSweep값을 켜서 바닥만이 아닌 모든 진행방향에 대한 충돌 검사로 변경한다. 기존에 z축만 검사하던 코드에서 드론이 이동하고 있는 모든 방향에 대한 검사가 가능하다.
3. 사물을 장착했을 때의 Sweep 액터 목록 관리: 
	* AddActorLocalOffset()을 실행할 때, bSweep값을 켜서 바닥만이 아닌 모든 진행방향에 대한 충돌 검사로 변경한다. 기존에 z축만 검사하던 코드에서 드론이 이동하고 있는 모든 방향에 대한 검사가 가능하다.
4. BoxComponent사이즈에 의존하지 않는 충돌 검사여전히 추이는 드론의 충돌체인 BoxComponent의 구조를 이용하는게 아니다보니 다른


근본적으로 이 많은 문제들은 물리충돌 연산을 필요로 하는 게임에 물리연산에 기반하지 않는 움직임을 사용함에서 발생한다. 직접하노라면 LineTrace/SweepTrace를 바탕으로 다양한 거리계산/충돌체크/이벤트 호출등을 구현하면 됬겠지만, 다행히도 그리고 당연하게도, 언리얼 엔진에는 이런 움직임 연산을 담당해줄 수 있는 컴포넌트가 있는데, **PawnMovementComponent**이다. 내부적으로는 MovementComponent도 Trace와 물리에 기반하여 움직임을 계산하고 월드에 반영하도록 도와주니 저 개선해나가는 흐름 자체가 MovementComponent의 필요성과 유사하다고 할 수 있다. Character에 기본으로 달려있는 CharacterMovementComponent도 PawnMovementComponent 기반으로 구현되어 있는 기초적인 컴포넌트이다.
이번엔 드론이동의 구현이니, PawnMovementComponent 의 비행 버전 구현체인 **FloatingPawnMovement**를 사용해보자. PawnMovementComponent가 모든걸 물리적인 이동으로 구현하지는 않고 강제하지도 않는다. 오히려 아케이드스런 즉각적인 움직임에는 어울리지 않으니 내가 사용하는 이유는 충돌, 기존의 물리 시스템과 보다 조화로운 구조를 만들기 위함이다.

## 컴포넌트 선언/부착
가장 먼저 **선언 후에 Pawn에 장착**해줘야 한다. 로직컴포넌트로 실체가 없기 때문에, 별도로 SetupAttachment()로 RootComponent에 붙이는 과정은 없지만, 대신 어떤 컴포넌트를 조종하게 될지 **SetUpdatedComponent()**를 이용해 정해줘야 한다. BoxComponent를 조종하는 것으로 만들어보자.
```c++
// Drone.h
#pragma once

//...
class UFloatingPawnMovement;

UCLASS()
class TRICKYDRONEDELIVERY_API ADrone : public APawn
{
	GENERATED_BODY()

public:
	//

protected:
	//...
	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = Drone)
	UFloatingPawnMovement* MovementComp;
	//...
```
```c++
// Drone.cpp
//...
#include "GameFramework/FloatingPawnMovement.h"

// Sets default values
ADrone::ADrone()
{
 	//...

	MovementComp = CreateDefaultSubobject<UFloatingPawnMovement>(TEXT("MovementComp"));
	MovementComp->SetUpdatedComponent(BoxComp);
}
```
```c++
// Drone.cpp
//...
#include "GameFramework/FloatingPawnMovement.h"

// Sets default values
ADrone::ADrone()
{
 	//...

	MovementComp = CreateDefaultSubobject<UFloatingPawnMovement>(TEXT("MovementComp"));
	MovementComp->SetUpdatedComponent(BoxComp);
}
```
이 상태로 빌드해 실행해보면 상속받은 BP_Drone에도 잘 장착이 되어있다.
## 이동 개선
이제 본격적으로 이동을 대신 만들 차례다. 기존에는 입력이 들어오면 입력에따라 움직일 방향을 설정하고, 그 방향과 이전 이동 방향을 계산해 Tick()에서 보정하는, 즉 실제 이동은 Tick()에서 이루어지는 구조였다. MovementComponent도 이와 유사하게 독립적인 Tick()을 가지며, AddMovementInput()을 추가하면 주어진 입력에 따라 이동을 계산, 조종의 대산이 되는 컴포넌트를 움직인다. 
때문에 AddMovementInput()로 속도에 반영해도 되겠지만, 그러면 급격한 방향전환에서 밀리는 느낌이 이전보다 덜하다. 이는 입력값 자체를 보간하면서 적용하던 이전 방식과, 주어지는 입력에 보다 즉각적으로 반응할 수 있는 MovementComponent의 차이 때문이다. 이는 Acceleration과 Deceleration을 낮췄을 때 가감속이 더 부드러워지긴 하지만, 조작에 대한 반응성까지 희생시키면서 부드러움을 챙기는 느낌이고, 이는 플레이 난이도의 상승으로 이어질 것 같다.
내가 원하는 드론을 원하는 방향으로 움직일 수 있으면서도 관성의 영향을 받는 상태와는 다르기에, 속도가 아닌 입력, 즉 방향만 보정해주는 기능은 유지를 할것이다. 이를 위해 이전과 같이 입력에 따라 를 별도로 계산/추적하고, 이를 Tick()에서 보간하면서 MovementComponent에 넘겨주는, 혼합적인 방식으로 구현해보자. 오히려 이전과 비슷해 코드가 바뀐건 많이 없다.
```c++
// Drone.cpp
//...
void ADrone::ChangeDesiredVelocity(const FInputActionValue& Value)
{
	if (!Controller) return;
	//Set desired movement direction to unit vector of local coordinate
	FVector Input = Value.Get<FVector>();
	FVector Direction(0.0);
	if (!FMath::IsNearlyZero(Input.X)) {
		Direction += GetActorForwardVector() * Input.X;
	}
	if (!FMath::IsNearlyZero(Input.Y)) {
		Direction += GetActorRightVector() * Input.Y;
	}
	if (!FMath::IsNearlyZero(Input.Z)) {
		Direction += GetActorUpVector() * Input.Z;
	}
	DesiredDirection = Direction.GetSafeNormal();
}
void ADrone::ResetDesiredVelocity(const FInputActionValue& Value)
{
	if (!Controller) return;
	//Reset desired movement velocity when the input disaapears
	DesiredDirection= { 0, 0, 0 };
}
//...
// Called every frame
void ADrone::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

	//Calculate velocity and apply to the movement
CurrentDirection = FMath::VInterpTo(CurrentDirection, DesiredDirection, DeltaTime, MovementLerpRate);
AddMovementInput(CurrentDirection);

	//...
}

//...
```
의미의 변화에 따라 Current/DesiredVelocity에서 CurrentDesiredDirection으로 이름을 바꿔주었다. Speed를 사용하지 않는점, Lerp대신 VInterpTo를 사용하는 점 정도를 제외하면 구조 자체는 이전과 유사하다.
내기준 MaxSpeed: 1000, Acceleration: 4000, Deceleration: 2000, 보정값이던 MovementLerpRate: 0.8이 원하던 느낌과 유사하게 날라다니는 것 같다.

## 카메라 회전
기존의 드론은 카메라를 회전하면 드론이 따라서 움직였다. 나는 별로 이게 조작감이 좋다고 생각하지 않는다. 첫번째로는 드론의 진행방향과 별개로 플레이어가 원하는 위치를 볼 수 있어야 정보를 얻는데 좋고, 둘째로 롤링을 할때 카메라가 회전하는건 꽤나 시야를 어지럽힌다. 때문에 카메라 조작을 드론으로부터 분리해보겠다. 언리얼은 이미 카메라 조작을 제공하기는 한다. AddControllerPitch/Yaw/RollInputer을 이용해 카메라 각도를 조정할 수 있고, 이 각도를 폰과 연동될지, 이동하면서 따라올지 선택할 수 있다.
```c++
//Drone.cpp
//...
ADrone::ADrone()
{
	//...
	SpringArmComp->bUsePawnControlRotation = true;
	//...
}
//...
void ADrone::Look(const FInputActionValue& Value)
{
	if (!Controller) return;
	//Rotate view using mouse input
	FVector2D Input = Value.Get<FVector2D>();
	if (!FMath::IsNearlyZero(Input.X)) {
		AddControllerYawInput(Input.X);
	}
	if (!FMath::IsNearlyZero(Input.Y)) {
		//limit from -90 to 90 degrees

		float CurrentPitch = FRotator::NormalizeAxis(GetControlRotation().Pitch);
		if ((Input.Y < 0 && CurrentPitch + Input.Y > -90) || (Input.Y > 0 && CurrentPitch + Input.Y < 90.0)) {
			AddControllerPitchInput(-Input.Y);
		}
	}
}
//...
```
기존에 GetActorRotation, AddActorLocalRotation()을 이용해 Drone의 로테이션을 조작하던 방식에서, 컨트롤러의 회전을 조작하는 방식으로 변경했다. 한가지 확인해놔야 할게 있다면 USpringArmComponent에서 UsePawnControlRotation이 true여야 한다는 것인데, 해제되어 있으면 장착된 액터/컴포넌트의 회전만 사용하고 컨트롤러의 회전값을 반영하지 않는다. 반대로 설정되어 있으면 지금 원하는 것처럼 액터의 회전과 관계없이 돌아간다.

## 상자 집기/배달
지금 프로젝트의 핵심 목표인 택배상자의 배달이다. 

