---
title: "TrickyDroneDelivery: 적 구현 - 까마귀, 물총꼬마"
author: Jaeseong Kim
date: 2026-04-21 8:00:00 +0800
categories: [Unreal, Project]
tags: [Unreal, Project, GameMode, GameState]
---
이전에는 물체를 들 수 있는 기능과, 들었을 때의 물리, 파괴 상호작용을 구현했다. 그 상자들을 들어서 단순히 배달만 하면 재미가 없을 것이다. 이번엔 배달과정을 방해할 적 2종류를 구현해보자. 가만히 앉아있다가 배달을 방해하러 올 까마귀와 드론을 향해 물을 발사할 개구진 꼬마이다.
## TakeDamage기반으로 변경
기존에는 Holdable안에서 데미지를 줄지 조건의 판단과 실제 데미지의 적용을 가지고 있었다. 그 조건이 모든 상황에 대해 잘 작동하면 모르겠지만, 충격력이나 속도를 기준값으로 가지고 이를 판단하기에 적이나 적이 쏘는 투사체의 속도가 충분치 않다면 조건이 불만족해 데미지가 적용되지 않는 상황이 있을 수 있겠다. 조건은 데미지를 적용하는 각 클래스에서 판별하는 것으로 하고, 데미지의 실제 적용 코드만 Holdable에 남기고자 한다. 근데 이렇게 바꾸는 김에 언리얼 엔진 기본 데미지 기능인 ApplyDamage/TakeDamage기반으로 변경하면 좋을 것 같다. 모든 액터에 기본적으로 존재하는 함수라 UGameplayStatics만 있으면 호출할 수 있고 전반적인 호환성을 올려준다.
```c++
//Holdable.h
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "Holdable.generated.h"

class UStaticMeshComponent;
class ADrone;
UCLASS()
class TRICKYDRONEDELIVERY_API AHoldable : public AActor
{
	GENERATED_BODY()

	// Called every frame
	virtual void Tick(float DeltaTime) override;
	
public:	
	//...
	virtual float TakeDamage(float DamageAmount, struct FDamageEvent const& DamageEvent, class AController* EventInstigator, AActor* DamageCauser) override;
protected:
	//...
};
```
```c++
//...
float AHoldable::TakeDamage(float DamageAmount, FDamageEvent const& DamageEvent, AController* EventInstigator, AActor* DamageCauser)
{
	if (!IsVulnerable) {
		UE_LOG(LogTemp, Warning, TEXT("Attack from %s, but not vulnerable!"), *(DamageCauser->GetName()));
		return 0.0f;
	}
	UE_LOG(LogTemp, Warning, TEXT("Damage from %s!"), *(DamageCauser->GetName()));
	Health--;
	if (Health <= 0) {
		Destroy();
	}
	else {
		MakeInvurnerable();
	}
	return DamageAmount;
}
//...
void AHoldable::OnHit(UPrimitiveComponent* HitComp, AActor* OtherActor, UPrimitiveComponent* OtherComp, FVector NormalImpulse, const FHitResult& Hit)
{
	if (OtherActor->IsA(ADrone::StaticClass())) {
		if (!IsHeld) {
			OtherActor->AddActorWorldOffset(-GetVelocity() * GetWorld()->GetDeltaSeconds());
		}
		return;
	}
	else if (IsHeld && IsVulnerable && NormalImpulse.Length() > ImpulseDamageThreshold) {
		FDamageEvent DamageEvent;
		TakeDamage(1, DamageEvent,nullptr, this);
	}
}
```
기존에 AddDamage()에 있던 코드를 TakeDamage()로 옮겨주는 한편, 무적상태의 판단도 TakeDamage()내부로 함께 옮겨줬다. 적들 입장에선 상자의 무적상태를 신경쓰지 않고 ApplyDamage()만 호출하면 되기에 호출 전에 확인할 요소가 하나 줄어들 것이다. OnHit()에서도 AddDamage()를 호출하는 대신 TakeDamage()를 호출한다. ApplyDamage()를 호출할 수도 있겠지만, 자기 자신의 데미지 판정이기에 TakeDamage()를 호출하는 편이 더 자연스럽겠다.
## 까마귀 구현
가만히 위치에 앉아있다가 플레이어를 추적해올 까마귀를 구현해보자. 처음에 배치된 위치를 기억했다가 플레이어가 일정 거리 안으로 상자를 가지고 들어오면 다가온다. 들이 받으면 데미지를 주고 잠시 멈췄다가 돌진하기를 반복한다. 거리를 벗어나면 제자리로 돌아가 플레이어가 다시 들어오기를 기다린다. 원래 Behavior Tree로 구현하려 했는데, 기껏해야 상태가 감시, 추적, 복귀정도밖에 없고, 감시와 추적상태 마저도 큰 차이가 없기에 이정도면 코드로 다 구현하는게 더 자연스러울 것 같다.
```c++
//Crow.h

#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Pawn.h"
#include "Crow.generated.h"

class USphereComponent;
class UStaticMeshComponent;
class UFloatingPawnMovement;
class ADrone;

UCLASS()
class TRICKYDRONEDELIVERY_API ACrow : public APawn
{
	GENERATED_BODY()

public:
	// Sets default values for this pawn's properties
	ACrow();

	// Called every frame
	virtual void Tick(float DeltaTime) override;

	// Called to bind functionality to input
	virtual void SetupPlayerInputComponent(class UInputComponent* PlayerInputComponent) override;

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;

	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = Crow)
	USphereComponent* SphereComp;

	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = Crow)
	UStaticMeshComponent* MeshComp;

	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = Crow)
	UFloatingPawnMovement* MovementComp;


	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Crow)
	float SearchRange;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Crow)
	float ChaseRange;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Crow)
	float MovementSpeed;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Crow)
	float ArrivalThreshold;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Crow)
	float DetectionInterval;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Crow)
	float AttackInterval;

	FVector Base;
	AActor* TargetDrone;
	bool bIsChasing;
	FTimerHandle DetectionTimer;
	bool CanAttack;
	FTimerHandle AttackTimer;

	UFUNCTION(BlueprintCallable, Category = Crow)
	void Charge(FVector Direction);

	UFUNCTION(BlueprintCallable, Category = Crow)
	void ReturnBase();

	UFUNCTION(BlueprintCallable, Category = Crow)
	void CheckTargetCondition();

	UFUNCTION(BlueprintCallable, Category = Crow)
	void SetBase();


	UFUNCTION()
	void StopAttack(UPrimitiveComponent* HitComp, AActor* OtherActor, UPrimitiveComponent* OtherComp, FVector NormalImpulse, const FHitResult& Hit);
	void ResumeAttack();

};
```
```c++
//Crow.cpp


#include "Crow.h"
#include "Components/SphereComponent.h"
#include "Components/StaticMeshComponent.h"
#include "GameFramework/FloatingPawnMovement.h"
#include "Engine/OverlapResult.h"
#include "CrowAIController.h"
#include "Drone.h"
#include "Holdable.h"
#include "Kismet/KismetSystemLibrary.h"
#include "Kismet/GameplayStatics.h"
#include "BehaviorTree/BlackboardComponent.h"
#include "Kid.h"

// Sets default values
ACrow::ACrow()
{
 	// Set this pawn to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;
	SphereComp = CreateDefaultSubobject<USphereComponent>(TEXT("SphereComp"));
	RootComponent = SphereComp;
	MeshComp = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("MeshComp"));
	MeshComp->SetupAttachment(RootComponent);
	MovementComp = CreateDefaultSubobject<UFloatingPawnMovement>(TEXT("MovementComp"));
	MovementComp->SetUpdatedComponent(SphereComp);
	AIControllerClass = ACrowAIController::StaticClass();
	AutoPossessAI = EAutoPossessAI::PlacedInWorldOrSpawned;
    AttackInterval = 2.0f;

}

// Called when the game starts or when spawned
void ACrow::BeginPlay()
{
	Super::BeginPlay();
	Base = GetActorLocation();

    AAIController* AIC = Cast<AAIController>(GetController());
    Super::BeginPlay();
    Base = GetActorLocation();
    bIsChasing = false;
    TargetDrone = nullptr;
    CanAttack = true;

    GetWorld()->GetTimerManager().SetTimer(DetectionTimer, this, &ACrow::CheckTargetCondition,DetectionInterval, true);
    SphereComp->OnComponentHit.AddDynamic(this, &ACrow::StopAttack);
}

void ACrow::Charge(FVector Direction){
    if (!MovementComp) return;

    FVector NewMovement = (Direction - GetActorLocation()).GetSafeNormal();// * MovementSpeed;
    MovementComp->AddInputVector(NewMovement);
}

void ACrow::ReturnBase(){
    if (!MovementComp) return;
    if (FVector::Dist(GetActorLocation(), Base) < ArrivalThreshold) {
        //MovementComp->StopMovementImmediately();
        return;
    }
    FVector Direction = (Base - GetActorLocation()).GetSafeNormal();// * MovementSpeed;

    MovementComp->AddInputVector(Direction);
}

void ACrow::CheckTargetCondition(){
    TArray<AActor*> OverlappedActors;
    UKismetSystemLibrary::SphereOverlapActors(
        GetWorld(), Base, SearchRange,
        TArray<TEnumAsByte<EObjectTypeQuery>>(),
        ADrone::StaticClass(),
        TArray<AActor*>{ this },
        OverlappedActors
    );
    DrawDebugSphere(GetWorld(), Base, SearchRange, 32, FColor::Cyan, false, DetectionInterval);
    for (AActor* Actor : OverlappedActors) {
        ADrone* Drone = Cast<ADrone>(Actor);
        if (!IsValid(Drone) || !Drone->GetIsHolding()) continue;

        FHitResult Hit;
        FCollisionQueryParams TraceParams;
        TraceParams.AddIgnoredActor(this);
        TraceParams.AddIgnoredActor(Drone);

        bool bHit = GetWorld()->LineTraceSingleByChannel(
            Hit, GetActorLocation(),
            Drone->GetHoldableActor()->GetActorLocation(),
            ECC_Visibility, TraceParams
        );

        if (bHit && Hit.GetActor() == Drone->GetHoldableActor()) {
            TargetDrone = Drone;
            bIsChasing = true;
            return;
        }
    }

    //Failed to Find
    TargetDrone = nullptr;
    bIsChasing = false;
}

void ACrow::SetBase() {
	Base = GetActorLocation();
}

void ACrow::StopAttack(UPrimitiveComponent* HitComp, AActor* OtherActor, UPrimitiveComponent* OtherComp, FVector NormalImpulse, const FHitResult& Hit)
{ 
    AHoldable* HoldableActor = Cast<AHoldable>(OtherActor);
    if (IsValid(HoldableActor) && CanAttack) {
        
        float ImpulseStrength = FVector::DotProduct(GetVelocity(), -Hit.ImpactNormal) * HoldableActor->FindComponentByClass<UStaticMeshComponent>()->GetMass();

        
        CanAttack = false;
        UGameplayStatics::ApplyDamage(HoldableActor, 1,nullptr,this, UDamageType::StaticClass());
        GetWorld()->GetTimerManager().SetTimer(AttackTimer, this, &ACrow::ResumeAttack, AttackInterval, false);
    }
}

void ACrow::ResumeAttack()
{
    CanAttack = true;
}

// Called every frame
void ACrow::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);
    if (bIsChasing)
    {
        if (!CanAttack) return;
        AActor* Target = Cast<ADrone>(TargetDrone)->GetHoldableActor();

        if (!IsValid(Target)) return;
        
        FVector NewMovement = (Target->GetActorLocation()
            - GetActorLocation()).GetSafeNormal() * MovementSpeed;
        UE_LOG(LogTemp, Warning, TEXT("Chasing! %f %f %f"), NewMovement.X, NewMovement.Y, NewMovement.Z);
        Charge(NewMovement);
    }
    else
    {
        ReturnBase();
    }
}

// Called to bind functionality to input
void ACrow::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);

}
```

## 물방울 구현
## 물총꼬마 구현
## 이번의 경험