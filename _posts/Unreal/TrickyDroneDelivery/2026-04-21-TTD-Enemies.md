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
	bool IsChasing;
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
#include "Drone.h"
#include "Holdable.h"
#include "Kismet/KismetSystemLibrary.h"
#include "Kismet/GameplayStatics.h"

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
    AttackInterval = 2.0f;

}

// Called when the game starts or when spawned
void ACrow::BeginPlay()
{
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
            IsChasing = true;
            return;
        }
    }

    //Failed to Find
    TargetDrone = nullptr;
    IsChasing = false;
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
    if (IsChasing)
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
크게 몇가지 기능이 있다. 시작되면 Base에 스폰됬던 위치를 저장해놓고, 일정 시간마다 드론이 일정 거리 내로 진입했는지 그리고 LineTrace가 성공하는지 확인한다. 프레임마다 수준으로 많이 확인해야 할정도는 아니라 판단, 해당 시간은 0.2로초 설정해두었다. 이 성공 여부에 따라 IsChasing 값을 결정한다. 이 값에 따라 Tick()에서는 실제 행동이 이루어진다. true이면 대상인 드론의 상자를 향해 움직이고, 충돌시 데미지를 주고 일정시간동안 멈춘다. 플레이어가 Base로부터 일정 거리 안에 없으면 Base로 돌아간다.

![image](/assets/img/260421-crow.gif)

적절한 까마귀 메쉬를 구하지 못했기에 일단 임시로 메쉬를 넣어줬다. 확인을 위해 넣어놓은 파란색 영역에 상자를 가지고 들어가면 추적하고, 아닐경우 제자리로 돌아가는 모습이다.

## 물방울 구현
다음으로 꼬마를 구현해야 하는데, 물방울을 먼저 구현해주자. 충돌하면 사라지는데, 충돌 대상이 상자면 데미지를 주어야 한다. 충돌하지 않더라도 시간이 지나면 부하 방지를 위해 스스로 사라지도록 만들어보자.
```c++
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "DamageCollider.generated.h"

class UStaticMeshComponent;
class UProjectileMovementComponent;

UCLASS()
class TRICKYDRONEDELIVERY_API ADamageCollider : public AActor
{
	GENERATED_BODY()
	
public:	
	// Sets default values for this actor's properties
	ADamageCollider();

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;

	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = ADamageCollider)
	UStaticMeshComponent* MeshComp;

	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = ADamageCollider)
	UProjectileMovementComponent* ProjectileComp;

	UPROPERTY(EditAnywhere,BlueprintReadWrite, Category = DamageCollider)
	float Lifespan;
	FTimerHandle LifeTimer;

	void EndLifeTimer();
	
	UFUNCTION()
	void OnCollision(UPrimitiveComponent* HitComp, AActor* OtherActor, UPrimitiveComponent* OtherComp, FVector NormalImpulse, const FHitResult& Hit);
};
```
```c++
// Fill out your copyright notice in the Description page of Project Settings.


#include "DamageCollider.h"
#include "Holdable.h"
#include "Kismet/GameplayStatics.h"
#include "Components/StaticMeshComponent.h"
#include "GameFramework/ProjectileMovementComponent.h"

// Sets default values
ADamageCollider::ADamageCollider()
{
 	// Set this actor to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
    MeshComp = CreateDefaultSubobject<UStaticMeshComponent>("MeshComp");
    RootComponent = MeshComp;

    ProjectileComp = CreateDefaultSubobject<UProjectileMovementComponent>(TEXT("ProjectileComp"));
    ProjectileComp->InitialSpeed = 1000.0f;
    ProjectileComp->MaxSpeed = 1000.0f;
    ProjectileComp->bRotationFollowsVelocity = true;
}

// Called when the game starts or when spawned
void ADamageCollider::BeginPlay()
{
	Super::BeginPlay();
    GetWorld()->GetTimerManager().SetTimer(LifeTimer, this, &ADamageCollider::EndLifeTimer, Lifespan, false);
    MeshComp->OnComponentHit.AddDynamic(this, &ADamageCollider::OnCollision);
}

void ADamageCollider::EndLifeTimer()
{
    GetWorld()->GetTimerManager().ClearTimer(LifeTimer);
    Destroy();
}

void ADamageCollider::OnCollision(UPrimitiveComponent* HitComp, AActor* OtherActor, UPrimitiveComponent* OtherComp, FVector NormalImpulse, const FHitResult& Hit)
{
    UE_LOG(LogTemp, Warning, TEXT("Waterball Collided!"));
    AHoldable* HoldableActor = Cast<AHoldable>(OtherActor);
    if (IsValid(HoldableActor)) {
        UGameplayStatics::ApplyDamage(HoldableActor, 1, nullptr, this, UDamageType::StaticClass());   
    }
    Destroy();
}
```
충돌시에는 OnCollision()이 호출되어 상자면 ApplyDamage()를 호출하고, 어디에 충돌하든 파괴된다. LifeTimer가 끝나도 파괴되도록 해놨는데, EndLifeTimer가 호출되며 파괴된다. 간단하게 쓸 머티리얼도 만들어서 씌워줬다. 당장은 물보다는 바다나 강의 표면같은 느낌이 들기도 한다.
<div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
  <figure>
    <img src="/assets/img/260421-water.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>WaterBall의 머티리얼 그래프</figcaption>
  </figure>
  <figure>
    <img src="/assets/img/260421-water.gif" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>레벨에 배치된 WaterBall</figcaption>
  </figure>
</div>

## 물총꼬마 구현
투사체인 물방울이 구현됬으니 돌아다닐 꼬마의 차례이다. 스폰된 위치에서 일정 범위를 무작위적으로 돌아다니고, 일정 거리 안에 들어오면 플레이어에게 물방울을 발사한다. 까마귀와 유사하지만 지상에서만 돌아다니고 원거리로 공격하는 차이정도이니 구현 내용은 크게 다르지 않다.
```c++
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Character.h"
#include "Kid.generated.h"

class ADamageCollider;
class ADrone;
UCLASS()
class TRICKYDRONEDELIVERY_API AKid : public ACharacter
{
	GENERATED_BODY()

public:
	// Sets default values for this character's properties
	AKid();

	// Called every frame
	virtual void Tick(float DeltaTime) override;

	// Called to bind functionality to input
	virtual void SetupPlayerInputComponent(class UInputComponent* PlayerInputComponent) override;
protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;
	
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Kid)
	float DetectionInterval;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Kid)
	float SearchRange;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Kid)
	float WanderingRange;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Kid)
	float ArrivalDistance;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Kid)
	float FireRate;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Kid)
	float ProjectileSpeed;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Kid)
	float FireLead;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Kid)
	FVector SpawnOffset;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Kid)
	TSubclassOf<class ADamageCollider> ProjectileActor;

	FVector Base;
	FVector Destination;
	ADrone* TargetDrone;
	FTimerHandle DetectionTimer;
	FTimerHandle AttackTimer;
	
	bool IsAttacking;
	void FireProjectile();
	void CheckTargetCondition();
	FVector GetRandomLocation();
};
```
```c++
// Fill out your copyright notice in the Description page of Project Settings.


#include "Kid.h"
#include "DamageCollider.h"
#include "GameFramework/CharacterMovementComponent.h"
#include "GameFramework/ProjectileMovementComponent.h"
#include "Kismet/KismetSystemLibrary.h"
#include "Kismet/GameplayStatics.h"
#include "Drone.h"

// Sets default values
AKid::AKid()
{
 	// Set this character to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;

}

// Called when the game starts or when spawned
void AKid::BeginPlay()
{
	Super::BeginPlay();
	Base = GetActorLocation();
	Destination = GetRandomLocation();
	IsAttacking = false;
	DrawDebugSphere(GetWorld(), Base, WanderingRange, 32, FColor::Magenta, false, 60);
    GetWorld()->GetTimerManager().SetTimer(DetectionTimer, this, &AKid::CheckTargetCondition, DetectionInterval,true);
}
// Called every frame
void AKid::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);
    if (IsAttacking) {
        DrawDebugLine(GetWorld(), GetActorLocation(), TargetDrone->GetActorLocation(), FColor::Red, false, 0.1);
    }
    else {
        if ((GetActorLocation() - Destination).Length() <= ArrivalDistance) {
            Destination = GetRandomLocation();
        }
        else {
            GetMovementComponent()->AddInputVector((Destination - GetActorLocation()).GetSafeNormal());
        }
    }
}

// Called to bind functionality to input
void AKid::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);

}

FVector AKid::GetRandomLocation()
{
	FVector Direction = FMath::VRand();
	Direction.Z = 0;
	float Distance = FMath::FRandRange(0, WanderingRange);
	FVector NewLocation = Base + Direction * Distance;
	return NewLocation;
}

void AKid::FireProjectile()
{
    if (!IsValid(TargetDrone) || !ProjectileActor) return;

    FVector StartLocation = GetActorLocation() + SpawnOffset;
    FVector TargetLocation = TargetDrone->GetActorLocation() + TargetDrone->GetVelocity() * FireLead;

    FVector OutVelocity;
    bool bSuccess = UGameplayStatics::SuggestProjectileVelocity(this, OutVelocity, StartLocation, TargetLocation, ProjectileSpeed, false, 0.0f, 0.0f, ESuggestProjVelocityTraceOption::DoNotTrace);

    if (!bSuccess) return;

    FActorSpawnParameters SpawnParams;
    SpawnParams.Owner = this;

    ADamageCollider* Projectile = GetWorld()->SpawnActor<ADamageCollider>(ProjectileActor, StartLocation,OutVelocity.Rotation(), SpawnParams);

    if (IsValid(Projectile)) {
        Projectile->GetComponentByClass<UProjectileMovementComponent>()->Velocity = OutVelocity;
    }
}

void AKid::CheckTargetCondition() {
    TArray<AActor*> OverlappedActors;
    UKismetSystemLibrary::SphereOverlapActors(
        GetWorld(), Base, SearchRange,
        TArray<TEnumAsByte<EObjectTypeQuery>>(),
        ADrone::StaticClass(),
        TArray<AActor*>{ this },
        OverlappedActors
    );
    DrawDebugSphere(GetWorld(), GetActorLocation(), SearchRange, 32, FColor::Cyan, false, DetectionInterval);
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
            if (!IsAttacking) {
                IsAttacking = true;
                GetWorld()->GetTimerManager().SetTimer(AttackTimer, this, &AKid::FireProjectile, FireRate, true);
            }
            //IsAttacking = true;
            return;
        }
    }

    //Failed to Find
    TargetDrone = nullptr;
    IsAttacking = false;
    GetWorld()->GetTimerManager().ClearTimer(AttackTimer);
}
```
Crow에서는 IsChasing을 이용해 Tick에서 계속 이동을 넣어줬지만,Kid는 사격을 일정 시간간격마다 반복하는 구조로 만들어 Tick에서 처리하지 않고 CheckTargetCondition()에서 타이머의 설정과 해제에 따라서 공격하게 된다. 사격에 있어서는 그냥 쏘면 투사체가 잘 맞지 않기 때문에 드론의 속도에 따라 리드값을 조금 주었는데, 영역 안에 접근 자체를 지양하는 플레이를 위해서는 빗나가기 보단 잘 맞추도록 만드는게 더 적합할 것으로 생각된다.
메쉬는 전에 생성해놓은 꼬마 모델을 넣어줬다. 확인을 위해 넣어둔 보라색 영역 안에서 랜덤한 좌표로 계속 돌아다니며, 들어간 드론이 상자를 가지고 있으면 쏘는 것을 볼 수 있다.
![image](/assets/img/260421-kid.gif)
## 이번의 경험: 적 난이도 조절하기
지금 적들의 동작에 필요한 속도, 탐색 범위, 사거리등 여러 요소들을 에디터에서 수정할 수 있도록 만들어두었는데, 플레이에 적합하도록 적절한 움직임과 행동으로 난이도를 만들어내는 것은 많은 시간을 필요로 하는 일인 것 같다. 당장은 동작하는 기초적인 틀은 갖추어졌으니 추가적인 조정은 폴리싱의 영역이긴 하겠지만, 의도대로, 그리고 디테일하게 조정될수록 결과물의 퀄리티에 좋은 영향을 미칠것이다.
