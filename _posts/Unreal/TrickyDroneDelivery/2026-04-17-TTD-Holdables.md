---
title: "TrickyDroneDelivery: 픽업상자 구현"
author: Jaeseong Kim
date: 2026-04-17 12:00:00 +0800
categories: [Unreal, Project]
tags: [Unreal, Project, NBC, PhysicalHandleComponent, Material]
---
이전엔 드론의 움직임을 자연스럽게 개선하는 과정을 거쳤다. 이제 본격적인 게임의 핵심인 택배배달을 구현해야 한다. 먼저 픽업할 상자를 구현해보자.

## Holdable 선언
액터 자체는 움직임이 필요 없이 StaticMesh기반에 기능만 붙일 것이다. 때문에 Actor를 기반으로 선언하고 StaticMeshComponent를 붙여주자.
```c++
//Holdable.h
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "Holdable.generated.h"

class UStaticMeshComponent;
UCLASS()
class TRICKYDRONEDELIVERY_API AHoldable : public AActor
{
	GENERATED_BODY()

	// Called every frame
	virtual void Tick(float DeltaTime) override;
	
public:	
	// Sets default values for this actor's properties
	AHoldable();

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;

	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = Holdable)
	UStaticMeshComponent* MeshComp;
};
```
```c++
//Holdable.cpp


#include "Holdable.h"
#include "Drone.h"
#include "Components/StaticMeshComponent.h"
#include "Components/BoxComponent.h"

// Sets default values
AHoldable::AHoldable()
{
 	// Set this actor to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;

	MeshComp = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("MeshComp"));
	MeshComp->SetSimulatePhysics(true);
	MeshComp->SetCollisionResponseToChannel(ECC_Camera, ECR_Ignore);
	RootComponent = MeshComp;
}
// Called when the game starts or when spawned
void AHoldable::BeginPlay()
{
	Super::BeginPlay();
}
// Called every frame
void AHoldable::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);
}
```
선언까진 기본적인 엑터들의 선언 방식과 크게 다르지 않다. 다만 스프링암에 걸려 카메라 거리에 영향을 주는 것이 불편하기에, 카메라에 대한 충돌 설정만 무시로 기본설정해주었다.

## 잡기 구현
이제 Drone이 들수 있도록 구현해보자. F를 눌러서 잡을 수 있도록 상호작용할 IA를 만들고 IMC와 Drone 클래스에서 연결해주자. PlayerContoller에서도 추가해주고, Drone 클래스에서 IA에 Grab함수를 바인딩해준다.
<div  style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
    <figure>
        <img  src="/assets/img/260417-Interact-IA.png"  alt=""  style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    </figure>
    <figure>
        <img  src="/assets/img/260417-Interact-IMC.png"  alt=""  style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    </figure>
</div>
  
```c++
//DronePlayerController.h
//...
UCLASS()
class TRICKYDRONEDELIVERY_API ADronePlayerController : public APlayerController
{
	GENERATED_BODY()
public:
	//...
	UPROPERTY(EditAnywhere, Category = Input)
	UInputAction* IA_Interact;
	//...
};
```
```c++
// Drone.h
//..
UCLASS()
class TRICKYDRONEDELIVERY_API ADrone : public APawn
{
	//...
	UFUNCTION()
	void Grab(const FInputActionValue& Value);
```
```c++
//Drone.cpp
// Called to bind functionality to input
void ADrone::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);
	if (UEnhancedInputComponent* EnhancedInput = Cast<UEnhancedInputComponent>(PlayerInputComponent)) {
		if (ADronePlayerController* PlayerController = Cast<ADronePlayerController>(GetController())) {
			//...
			if (PlayerController->IA_Interact) {
				EnhancedInput->BindAction(PlayerController->IA_Interact, ETriggerEvent::Started, this, &ADrone::Grab);
			}
		}
	}
}
void ADrone::Grab(const FInputActionValue& Value)
{
	UE_LOG(LogTemp, Warning, TEXT("Try Grabbing"));
}
```
실행해보면 F키를 눌렀을 때 로그가 잘 출력된다. 이제 실제로 드론이 아래 사물이 있음을 감지하도록 해보자. Drone 아래에 HoldingPoint라는 BoxComponent를 추가하고 위치를 조정 후에 충돌 이벤트를 추가해준다.
```c++
//Drone.h
//...
class AHoldable;
//...
UCLASS()
class TRICKYDRONEDELIVERY_API ADrone : public APawn
{
	GENERATED_BODY()

public:

	//...
protected:
	//...
	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = Drone)
	UBoxComponent* HoldPoint;
	//...
	
	AHoldable* HoldableActor;
	bool IsHolding;
	UFUNCTION()
	void MakeReadyToHold(UPrimitiveComponent* OverlappedComp, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult);

	UFUNCTION()
	void MakeNotReadyToHold(UPrimitiveComponent* OverlappedComp, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex);
}
```
```c++
//...
ADrone::ADrone()
{
	//...
	HoldPoint = CreateDefaultSubobject<UBoxComponent>(TEXT("HoldPoint"));
	HoldPoint->SetupAttachment(RootComponent);
	//...
}
// Called when the game starts or when spawned
void ADrone::BeginPlay()
{
	Super::BeginPlay();
	CurrentDirection = { 0, 0, 0 };
	DesiredDirection = { 0, 0, 0 };
	IsRolling = false;
	IsMoving = false;
	HoldableActor = nullptr;
	IsHolding = false;

	HoldPoint->OnComponentBeginOverlap.AddDynamic(this, &ADrone::MakeReadyToHold);
	HoldPoint->OnComponentEndOverlap.AddDynamic(this, &ADrone::MakeNotReadyToHold);
}

//...
void ADrone::MakeReadyToHold(UPrimitiveComponent* OverlappedComp, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
	if (OtherActor->IsA(AHoldable::StaticClass())) {
		HoldableActor = Cast<AHoldable>(OtherActor);
		UE_LOG(LogTemp, Warning, TEXT("Holdable: %s"), *(HoldableActor->GetName()));
	}
}

void ADrone::MakeNotReadyToHold(UPrimitiveComponent* OverlappedComp, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex)
{
	if (!IsHolding) {
		HoldableActor = nullptr;
		UE_LOG(LogTemp, Warning, TEXT("Not Overlapped!"));
	}
}
```
<div  style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
    <figure>
        <img  src="/assets/img/260417-detection.png"  alt=""  style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    </figure>
    <figure>
        <img  src="/assets/img/260417-detection.gif"  alt=""  style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    </figure>
</div>
  
아래에 물체가 있음을 확인할 수 있게 되었다. 아래 HoldingPoint가 겹치는지에 따라 로그를 띄우고 있다. 이제 물체를 실제로 장착하도록 해보자.
간단한 방식으로는 AttachToComponent를 쓰는 방법이 있다. 이 경우 내가 원하던 것처럼 완벽히 특정 컴포넌트/소켓에 장착되어 하나의 몸체처럼 움직이게 되는데, 대신 SimulatePhysics, 물리연산을 사용할 수 없다. 키게되면 물리연산이 소켓에 장착되어 Pawn을 따라다니는 판정보다 우선시되어 따라다니지 않게된다. 즉 Holdable이 Drone의 움직임을 여전히 방해하고 싶으면 Sweep으로 별도로 충돌여부를 체크해야되는데 이는 이동은 개선하던 저번과 마찬가지로 원하는 방식은 아니다.
또 하나의 방법은 SetActorLocation(), SetActorRotation()으로 Holdable의 위치를 강제로 조작하는 방법이 있다. 충돌도 살아있고 벽을 뚫거나 하지는 않겠지만, 물리 연산을 보장해주지는 않는다. 벽에 해당 액터만 걸렸다가 나온다면 부드럽게 끌려오기보단 텔레포트하는 느낌이 들며, 이를 보정하는것은 사용자의 몫이 된다.
그리고 지금 쓸 방법인 PhysicalHandleComponent/PhysicalConstraintComponent를 사용하는 방법도 있다. 장착할 액터를 물리적으로 연결하게 해주는 컴포넌트인데 전의 방법과 결과적으로는 유사하지만, 부착된 액터가 끌려오는 과정이 더 자연스럽고 물리연산이 자동적으로 이루어진다. Handle은 느슨하게 연결되어 장착된 Pawn의 Tick()에서 위치의 업데이트를 해주어야된다. 단순히 SetActorLocation()등의 방법으로 불러오는 것과는 다르게, 물리 연산을 끄지 않아도 되고, 미리 handle에 주어진 damping등의 수치에 따라 물리적으로 자연스럽게 딸려오도록 구현된다. 그에 비해 Constraint방식은 관절에 달린 것처럼 Pawn과 한몸체가 되는 수준으로 단단하게 결합한다.
물리 충돌이 있으면서도 잘 부착되있을 수 있는 PhysicalConstraint방식으로 구현해보자.
```c++
// Drone.h
//...
class UPhysicsConstraintComponent;
//...
UCLASS()
class TRICKYDRONEDELIVERY_API ADrone : public APawn
{
	GENERATED_BODY()

public:
	//...

protected:
	//..
	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = Drone)
	UPhysicsConstraintComponent* PhysicsConstraint;
	//...

	UFUNCTION()
	void Hold();
	UFUNCTION()
	void Unhold(AActor* DestroyedActor);
};
```
```c++
//Drone.cpp
//...
ADrone::ADrone()
{
	//...
	PhysicsConstraint = CreateDefaultSubobject<UPhysicsConstraintComponent>(TEXT("PhysicalConstComp"));
	PhysicsConstraint->SetupAttachment(RootComponent);
}
//...
void ADrone::Grab(const FInputActionValue& Value)
{
	if (IsValid(HoldableActor) && !IsHolding) {
		Hold();
	}
}
void ADrone::Hold()
{
	if (IsHolding || !IsValid(HoldableActor)) {
		return;
	}
	IsHolding = true;
	HoldableActor->SetHeld(this);
	//Set PhysicalConstraint
	UPrimitiveComponent* Target = HoldableActor->FindComponentByClass<UPrimitiveComponent>();
	HoldableActor->SetActorLocation(GetActorLocation() - GetActorUpVector() * HoldingDistance);
	PhysicsConstraint->SetConstrainedComponents(Cast<UPrimitiveComponent>(BoxComp), NAME_None, Target, NAME_None);

	PhysicsConstraint->SetLinearXLimit(LCM_Locked, 0.f);
	PhysicsConstraint->SetLinearYLimit(LCM_Locked, 0.f);
	PhysicsConstraint->SetLinearZLimit(LCM_Locked, 0.f);
	PhysicsConstraint->SetAngularSwing1Limit(ACM_Locked, 0.f);
	PhysicsConstraint->SetAngularSwing2Limit(ACM_Locked, 0.f);
	PhysicsConstraint->SetAngularTwistLimit(ACM_Locked, 0.f);
	
	//Add listener on HoldableActor events
	HoldableActor->OnDestroyed.AddDynamic(this, &ADrone::Unhold);
}

void ADrone::Unhold(AActor* DestroyedActor)
{
	IsHolding = false;
	HoldableActor = nullptr;
	//Unset PhysicalConstraint
	PhysicsConstraint->BreakConstraint();
}
```
 새로운 컴포넌트로 PhysicalConstComp를 등록해줬고, 이를 생성자에서 부착해준다. 이전에 준비된 액터 감지를 확인 후 활성화되던 Grab()에서 로그출력 대신 Hold()를 호출하도록 변경했다, 하단에 있는 액터의 정보를 불러오고 드론과의 위치 HoldingDistance로 맞춘 후에 PhysicalConstranint를 설정한다. Drone측만 아니라, Holdable액터에도 추후에 있을 기능을 위해 누군가 잡고있는지 내부변수로 저장하도록 설정했다. 해당 액터가 위치가 멀어지거나 각도가 틀어지지 않도록 제한을 걸고, 잡고있는 액터가 파괴되면 Unhold()를 호출해 연결을 해제하도록 설정한다.
 ```c++
//Holdable.h
//...
class ADrone;
UCLASS()
class TRICKYDRONEDELIVERY_API AHoldable : public AActor
{
	GENERATED_BODY()

	// Called every frame
	virtual void Tick(float DeltaTime) override;
	
public:	
	//...
	void SetHeld(ADrone* WhoGrabs);
	//...
}
```
```c++
//Holdable.cpp
#include "Drone.h"
//...
void AHoldable::SetHeld(ADrone* WhoGrabs)
{
	MakeInvurnerable();
	IsHeld = true;
	this->Holder = WhoGrabs;
}
```
![image](/assets/img/260417-constraint.gif)
가까이가면 F키를 눌러 잡을 수 있고, 움직임을 따라 잘 움직인다.
  

## 충돌 이벤트 연동
구현은 완료되었어도 방금 보았듯이, 상자를 드론이 눌러서 땅속으로 넣어버리거나 물리 상호작용에서 지나친 떨림이 발생하는 등의 문제가 있다. Drone 이 완전히 물리 기반으로 이동하는게 아니다보니 그렇다. 물리 수치 조절과 충돌시 드론의 이동값 보정으로 상쇄해주는 코드를 추가하겠다. 박스가 충돌시에 충돌 대상이 장착된 Drone이 아니라면, 측 벽이나 다른 사물과 충돌했다면 NormalImpulse의 일부를 Drone의 속도보정치로 적용해 뚫고 지나가는등의 행위를 제한한다.
```c++
// Drone.h
//...
UCLASS()
class TRICKYDRONEDELIVERY_API ADrone : public APawn
{
	GENERATED_BODY()

public:
	//...

protected:
	//..
	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = Movement)
	float CollisionImpulseTakingRatio;		//How much from NormalImpulse to take on movement.
	UFUNCTION()
void OnHoldableCollision(UPrimitiveComponent* HitComp, AActor* OtherActor, UPrimitiveComponent* OtherComp, FVector NormalImpulse, const FHitResult& Hit);
};
```
```c++
//Drone.cpp
//...
void ADrone::Hold()
{
	//...
	//Add listener on HoldableActor events
	HoldableActor->OnDestroyed.AddDynamic(this, &ADrone::Unhold);
	Target->OnComponentHit.AddDynamic(this, &ADrone::OnHoldableCollision);
}

void ADrone::Unhold(AActor* DestroyedActor)
{
	//remove collision dynamic
	if (IsValid(HoldableActor))
	{
		UPrimitiveComponent* Target = HoldableActor->FindComponentByClass<UPrimitiveComponent>();
		if (Target)
		{
			Target->OnComponentHit.RemoveDynamic(this, &ADrone::OnHoldableCollision);
		}
	}
	IsHolding = false;
	HoldableActor = nullptr;
	//Unset PhysicalConstraint
	PhysicsConstraint->BreakConstraint();
}

void ADrone::OnHoldableCollision(UPrimitiveComponent* HitComp, AActor* OtherActor, UPrimitiveComponent* OtherComp, FVector NormalImpulse, const FHitResult& Hit)
{
	if (OtherActor->IsA(ADrone::StaticClass())) {
		return;
	}
	else {
		MovementComp->Velocity += NormalImpulse * CollisionImpulseTakingRatio;
	}
}
```

CollisionImpulseTakingRatio를 일단 0.0001로 설정해놨다. 튕겨나가는정도까진 아니고, 뚫고 들어가는 것만 막는 수준을 원하는데, Impulse가 벡터 크기가 1000은 기본적으로 넘고 크면 10만도 넘는 경우는 있기에 충분히 작아도 잘 작동한다. 또한 잡고있지 않을때도 상자를 벽이나 바닥 속으로 밀어버릴 수 있는 현상을 막기위해, 잡고있지 않을 때는 드론이 닿을때 항상 밀려나는 속도만큼 반대로 밀어내도록 충돌 이벤트를 추가해줬다.
```c++
//Holdable.h
#pragma once
//...
UCLASS()
class TRICKYDRONEDELIVERY_API AHoldable : public AActor
{
//...
	GENERATED_BODY()

	// Called every frame
	virtual void Tick(float DeltaTime) override;
	
public:	
	///...
protected:
	//...
	UFUNCTION()
	void OnHit(UPrimitiveComponent* HitComp, AActor* OtherActor, UPrimitiveComponent* OtherComp, FVector NormalImpulse, const FHitResult& Hit);
};
```
```c++
//Holdable.cpp
//...
void AHoldable::OnHit(UPrimitiveComponent* HitComp, AActor* OtherActor, UPrimitiveComponent* OtherComp, FVector NormalImpulse, const FHitResult& Hit)
{
	if (OtherActor->IsA(ADrone::StaticClass())) {
		if (!IsHeld) {
			OtherActor->AddActorWorldOffset(-GetVelocity() * GetWorld()->GetDeltaSeconds());
		}
		return;
	}
}
```
![image](/assets/img/260417-collision.gif)
반대로 드론이 충돌할 때 상자로 속도 영향이 있어야 하나도 처음엔 고민했었는데, PhysicalConstraint로 워낙 강하게 묶여있고 드론이 충돌해 멈출정도면 상자도 당연히 멈추기에 별 상관 없을 것 같다. 전체적으로는 의도대로 작동하니, 세부적인 충돌감, 조작감 구현은 폴리싱의 영역인 것 같다.

## 충돌 파괴 구현
단순히 배달만 하면 재미가 없을 것이다. 일단은 들고있는 택배상자가 세게 3번 부딪히면 파괴되도록 할 생각이다 이를 위해 OnHit이벤트에 기능을 추가해주자.  누군가에게 들려있고(IsHeld), 데미지를 받을 수 있으며(IsVulnerable), 충격이 일정 크기 이상이면(ImpulseDamageThreshold), AddDamage()를 호출해 데미지 처리를 진행한다. 체력을 깎고, 체력이 0이면 파괴하지만 아니면 무적상태 타이머를 작동시킨다. 처음 상자를 잡을때나, 지속적으로 바닥에 끌고가는 상황에서 추가적인 데미지를 쌓고싶진 않으니, 타이머를 이용한 무적상태도 추가해보자.
간단하게 무적상태와 아닐때를 구별할 머티리얼도 설정해줬다.OnHit에서 AddDamage()를 호출해서 피해 처리를 하도록 추가했다.
```c++
//Holdable.h
//...
UCLASS()
class TRICKYDRONEDELIVERY_API AHoldable : public AActor
{
	//...
public:	
	float GetImpulseDamageThreshold();
	//...

protected:
	//...
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Holdable)
	float VulnerabilityCooldown;

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Holdable)
	UMaterialInterface* VulnerableMaterial;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Holdable)
	UMaterialInterface* InvulnerableMaterial;

	int32 Health;
	bool IsVulnerable;

	void AddDamage();
	void MakeInvurnerable();
	void MakeVurnerable();
	FTimerHandle InvulnerableTimer;
```
```c++
//Holdable.cpp
//...
// Sets default values
AHoldable::AHoldable()
{
 	// Set this actor to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;

	MeshComp = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("MeshComp"));
	MeshComp->SetSimulatePhysics(true);
	MeshComp->SetCollisionResponseToChannel(ECC_Camera, ECR_Ignore);
	RootComponent = MeshComp;
	MaxHealth = 3;
	VulnerabilityCooldown = 3.0;
}

void AHoldable::AddDamage()
{
	Health--;
	if (Health <= 0) {
		Destroy();
	}
	else {
		MakeInvurnerable();
	}
}
float AHoldable::GetImpulseDamageThreshold()
{
	return ImpulseDamageThreshold;
}
void AHoldable::MakeInvurnerable() {
	IsVulnerable = false;
	//MeshComp->SetMaterial(0, InvulnerableMaterial);
	GetWorld()->GetTimerManager().SetTimer(InvulnerableTimer, this, &AHoldable::MakeVurnerable, VulnerabilityCooldown, false);
}
void AHoldable::MakeVurnerable()
{
	//MeshComp->SetMaterial(0, VulnerableMaterial);
	UE_LOG(LogTemp,Warning, TEXT("Timer Finished"));
	IsVulnerable = true;
}

void AHoldable::OnHit(UPrimitiveComponent* HitComp, AActor* OtherActor, UPrimitiveComponent* OtherComp, FVector NormalImpulse, const FHitResult& Hit)
{
	if (OtherActor->IsA(ADrone::StaticClass())) {
		if (!IsHeld) {
			OtherActor->AddActorWorldOffset(-GetVelocity() * GetWorld()->GetDeltaSeconds());
		}
		return;
	}
	else if (IsHeld && IsVulnerable && NormalImpulse.Length() > 	else if (IsHeld && IsVulnerable && NormalImpulse.Length() > ImpulseDamageThreshold) {
) {
		AddDamage();
	}
}
```

다만 상자의 충돌만 피해를 입는게 아니라, Drone의 충돌도 피해를 입게 만들고 싶다. 그래서 드론에도 충돌 처리를 넣어줬다. 드론은 특유의 이동방식때문에 NormalImpulse가 항상 0으로 나온다. 그래서 충돌 방향과 속도의 내적, 그리고 들고있는 물체의 질량을 이용해 충격량을 구했다. 다만 그래도 부드러운 이동과 속도 한계치 탓에 충격량이 크게 나오진 않기에, 피해 임계점을 HoldableActor가 가지는 값의 1/10로 기준치를 잡았다.
```c++
//ADrone.h
#pragma once
//...
UCLASS()
class TRICKYDRONEDELIVERY_API ADrone : public APawn
{
//...
	GENERATED_BODY()

	// Called every frame
	virtual void Tick(float DeltaTime) override;
	
public:	
	///...
protected:
	//...
	float HoldableActorMass;
	UFUNCTION()
void CollideWhileHolding(UPrimitiveComponent* HitComp, AActor* OtherActor, UPrimitiveComponent* OtherComp, FVector NormalImpulse, const FHitResult& Hit);
};
```
```c++
//Drone.cpp
void ADrone::Hold()
{
	//...
	//...
	HoldableActorMass = HoldableActor->FindComponentByClass<UStaticMeshComponent>()->GetMass();
}
void ADrone::CollideWhileHolding(UPrimitiveComponent* HitComp, AActor* OtherActor, UPrimitiveComponent* OtherComp, FVector NormalImpulse, const FHitResult& Hit)
{
	float ImpulseStrength = FVector::DotProduct(GetVelocity(), -Hit.ImpactNormal) * HoldableActorMass;
	UE_LOG(LogTemp, Warning, TEXT("DroneCollided! - %f "), ImpulseStrength);
	if (HoldableActor->GetIsVulnerable() && ImpulseStrength > HoldableActor->GetImpulseDamageThreshold() / 10) {
		UE_LOG(LogTemp, Warning, TEXT("DamageFromDrone!"));
		HoldableActor->AddDamage();
	}
}
```
![image](/assets/img/260417-final.gif)
상자를 잡은 후의 움직임 자체는 잘 구현되었다. 상자와 드론, 어느쪽에서 충돌하든 데미지가 누적되며, 상자만 충돌하더라도 드론의 움직임에 영향을 줄 수 있다. 기초적인 조작은 이정도면 다 구현한 것 같다. 추가적인 기능들 구현 후에 폴리싱하면서 세부조절하면 될 것 같다.

## 이번의 경험: 비물리와 물리의 공존
이동을 먼저 구현하고 사물의 물리 시뮬레이션과 공존하기 위해선 크게 3가지 방법이 있는 것 같다.
1. 이동과 사물의 움직임 모두 비물리
	* 모두 비물리에 기반하여 구현하고 충돌과 상호작용을 직접 정의한다.
	*  SetActorLocation(), AddActorLocalOffset()등을 활용하여 위치를 강제로 이동시키는데, 즉각적이고 사실성이 덜 필요한 게임에선 오히려 좋을 수 있다. 
	* 다만 이는 충돌/겹침등의 상호작용에 대해 직접 구현할 부분이 많아진다는 말이기도 하다. 
	* 내가 처음에 구현한 관성같은 이동을 위해 Lerp를 AddActorLocalOffset()과 조합한 경우도 비슷하다 볼 수 있겠다.
2. 모든 움직임을 물리에 기반
	* 모두 물리 연산을 이용하여 만드는 방법이다. 
	* AddForce(), Velocity의 수정등을 통해 위치와 회전같은 transform정보가 간접적으로 조절된다. 
		* 수치를 적절히 조정하면 비물리기반의 움직임과 유사하게도 만들 수 있다. 
	* 엔진에 사전에 정의된 물리 상호작용들을 기반하기에 기능적으로 충돌할 여지는 더 적은편이다.
		* 다만 물리 시뮬레이션을 필요로 하기에 연산 부하가 1번 방법에 비해 많고, 정밀한 조절이 힘들 수 있고, 네트워크 동기화가 비교적 어려울 수 있다.
3. 1번과 2번의 조합
	* 비물리 기반 이동과 물리 기반 사물이동, 혹은 그 반대로 두 방법을 조합한다. 
	* 필요에 따라 방법을 선택해 구현한다는 점에서 양 측의 장점을 모두 흡수할 수도 있지만, 물리법칙을 벗어난 움직임과 물리량에 기반한 움직임이 혼재한다는 점에서 오히려 의도치 않거나 자연스럽지 못한 그림이 더 많이 보이는 것 같다. 즉 신경쓸 부분이 더 늘어난다.
	* 지금 구현한 Drone과 Holdable도 여기에 속한다. 드론의 움직임이 충돌이나 각종 상호작용에 지나치게 휘둘린다면 조작감이 좋지 않을것이다. 그런점에서 비물리에 기반하는 움직임을 사용하는 Drone은 불가피한 선택이고, 그런 드론과 자연스럽게 떨어지고, 걸리고, 쓰러지는등의 모습을 보이는 상자들은 더 자연스럽고 매력적인 게임을 만들어 줄 것이다.

처음엔 1번 방법으로 해보다가 드론과 상자의 물리 충돌 연동이라는 목적때문에 3번 방법으로 선회했다. 상자르 짓이겨버리는듯한 드론의 움직임을 충돌시 반대로 속도를 추가해주는 코드로 이를 상쇄했는데, 더 좋은 방법이 있을지는 탐구의 영역인 듯 하다. 드론처럼 자유로운 이동과 회전을 필요로하는 경우엔 적합하지 않을 수도 있지만, 조작감과 성능을 모두 챙기는 선택지기에 게임으로 가장 매력적이지 않을까. 