---
title: "TrickyDroneDelivery: 랜덤 레벨 생성-GameMode, GameState"
author: Jaeseong Kim
date: 2026-04-22 8:00:00 +0800
categories: [Unreal, Project]
tags: [Unreal, Project, GameMode, GameState]
---
이전엔 레벨마다 스폰될 적들을 만들었다. 이번엔 게임의 규칙과 레벨의 생성을 구현해보자.
## 랜덤 건물 배치
건물은 당장은 플로우 검증 정도만 하고 싶어서 다양한 건물의 배치보단 간단한 건물배치를 이용한 맵을 구현해보도록 하겠다. 방식의 유효성, 플레이 가능성의 판단이 우선이니 일단은 정사각형 그리드로 건물이 배치될 수 있는지를 구현해볼 것이다. 간단하게 기존에 테스트용을 대체할 주택 건물 2개를 큐브를 배치해 만들고, 배달 도착지점까지 포함시켜 미리 블루프린트상에 배치해놓았다.
<div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
  <figure>
    <img src="/assets/img/260422-house1.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>경사지붕 집</figcaption>
  </figure>
  <figure>
    <img src="/assets/img/260422-house2.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>평지붕 집</figcaption>
  </figure>
</div>
이제 이 건물들을 배치할 수 있는 로직을 만들어보자. GameMode에서는 등록되어있는 배치용 건물 블루프린트를 정해진 구역 안에서 GridSize마다 배치하여 마을을 랜덤으로 형성한다. 도착지점이 항상 활성화되어 있으면 가까운 곳에 모두 배달하여 재미 없으니, 스폰과 동시에 모든 배달지점은 비활성화시키고, 택배상자의 픽업과 배달/파괴시 호출될 수 있는 EnableDelivery()/DisableDelivery()를 만들었다. 
```c++
//TDDGameMode.h

#pragma once

#include "CoreMinimal.h"
#include "GameFramework/GameMode.h"
#include "TDDGameMode.generated.h"

class ADeliveryZone;

UCLASS()
class TRICKYDRONEDELIVERY_API ATDDGameMode : public AGameMode
{
	GENERATED_BODY()
public:
	

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = TTDGameState)
	TArray<TSubclassOf<AActor>> Enemies;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = TTDGameState)
	TArray<TSubclassOf<AActor>> DeliveryPackages;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = TTDGameState)
	TArray<TSubclassOf<AActor>> BuildingPrefabs;

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = TTDGameState)
	float MaxWidth;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = TTDGameState)
	float MaxHeight;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = TTDGameState)
	float GridSize;

	
	void EnableDelivery();
	void DisableDelivery(AActor* DestroyedActor);
protected:
	virtual void StartPlay() override;

	void CreateTown();

	int32 RemainingBoxes;
	int32 Difficulty;
	TArray<AActor*> DeliveryZones;
	int32 EnabledZone;
};
```
```c++
//TDDGameMode.cpp


#include "TDDGameMode.h"
#include "Kismet/GameplayStatics.h"
#include "DeliveryZone.h"

void ATDDGameMode::StartPlay()
{
	Super::StartPlay();
	CreateTown();
	UGameplayStatics::GetAllActorsOfClass(GetWorld(), ADeliveryZone::StaticClass(), DeliveryZones);
	UE_LOG(LogTemp, Warning, TEXT("Found EventZones: %d"), DeliveryZones.Num());
	for (AActor* Zone : DeliveryZones) {
		Zone->SetActorHiddenInGame(true);
	}
	EnabledZone = -1;
}

void ATDDGameMode::CreateTown()
{
	UE_LOG(LogTemp, Warning, TEXT("Creating Town"));
	int WidthIndex = MaxWidth / GridSize;
	int HeightIndex = MaxHeight / GridSize;
	for (int i = -WidthIndex; i <= WidthIndex; i++) {
		for (int j = -HeightIndex; j <= HeightIndex; j++) {
			if (i == 0 || j == 0) {
				continue;
			}
			
			if (BuildingPrefabs.IsEmpty()) {
				UE_LOG(LogTemp, Warning, TEXT("No Building Prefabs! Spawning sphere instead!"));
				DrawDebugSphere(GetWorld(), FVector(i * GridSize, j * GridSize, 0), GridSize / 2, 32, FColor::Green, false, 30);
			}
			else {
				UE_LOG(LogTemp, Warning, TEXT("Spawning Building Prefabs!"));
				FActorSpawnParameters SpawnParams;
				SpawnParams.Owner = this;
				AActor* EventZone = GetWorld()->SpawnActor<AActor>(BuildingPrefabs[FMath::RandRange(0, BuildingPrefabs.Num() - 1)], FVector(i * GridSize, j * GridSize, 0), FRotator::ZeroRotator, SpawnParams);
			}
			
		}
	}
}

void ATDDGameMode::EnableDelivery() {
	UE_LOG(LogTemp, Warning, TEXT("Delivery zone enabled!"));
	EnabledZone = FMath::RandRange(0, DeliveryZones.Num() - 1);
	DeliveryZones[EnabledZone]->SetActorHiddenInGame(false);
}

void ATDDGameMode::DisableDelivery(AActor* DestroyedActor) {
	UE_LOG(LogTemp, Warning, TEXT("Delivery zone disabled!"));
	if (EnabledZone >= 0) {
		DeliveryZones[EnabledZone]->SetActorHiddenInGame(true);
	}
	EnabledZone = -1;
}
```
생성 후에 레벨의 게임모드로 설정하고, 게임모드에 집들을 등록한 후에 실행해보면 잘 작동함을 확인할 수 있다.
<div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
  <figure>
    <img src="/assets/img/260422-grid1.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>시작시 보이지 않는 배달지점</figcaption>
  </figure>
  <figure>
    <img src="/assets/img/260422-grid2.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>픽업과 함께 활성화된 배달지점</figcaption>
  </figure>
</div>

## 랜덤 적 배치, 난이도 설정
이제 전에 만들어두었던 적들도 배치를 할 수 있도록 해야겠다. 3개의 스테이지로 구성하고, 1레벨엔 적 없이 가장 작은 상자만, 2레벨엔 중간 상자까지, 3레벨엔 가장 큰 상자까지 배치할 것이다. 적들도 1레벨엔 아무것도 안뜨다가 2레벨에선 4명의 물총꼬마를, 3레벨에선 4마리의 까마귀까지 추가로 생성할 것이다. GameMode클래스에서 구현해보자.
```c++
```
```c++
```

## 게임 클리어/오버 판정


![image](/assets/img/260421-crow.gif)

