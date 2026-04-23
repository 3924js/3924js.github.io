---
title: "TrickyDroneDelivery: 랜덤 레벨 생성 - GameMode, GameState"
author: Jaeseong Kim
date: 2026-04-22 8:00:00 +0800
categories: [Unreal, Project]
tags: [Unreal, Project, GameMode, GameState]
---
이전엔 레벨마다 스폰될 적들을 만들었다. 이번엔 게임의 규칙과 레벨의 생성을 구현해보자.

## 랜덤 건물 배치
건물은 당장은 플로우 검증 정도만 하고 싶어서 다양한 건물의 배치보단 간단한 건물배치를 이용한 맵을 구현해보도록 하겠다. 간단하게 건물들을 전에 구현한 이벤트존과 하나로 합쳐 2개의 블루프린트 액터에 미리 배치해뒀다. 
<div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
  <figure>
    <img src="/assets/img/260421-water.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>경사 지붕 주택</figcaption>
  </figure>
  <figure>
    <img src="/assets/img/260421-water.gif" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>평지붕 주택</figcaption>
  </figure>
</div>

게임 모드에서 이를 배치할 수 있도록 등록된 구역에 대해 랜덤으로 배치할 수 있는 기능을 구현해보자.
```c++
//TDDGameMode.h
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/GameMode.h"
#include "TDDGameMode.generated.h"

UCLASS()
class TRICKYDRONEDELIVERY_API ATDDGameMode : public AGameMode
{
	GENERATED_BODY()
public:
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = TTDGameState)
	TArray<TSubclassOf<AActor>> BuildingPrefabs;

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = TTDGameState)
	float MaxWidth;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = TTDGameState)
	float MaxHeight;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = TTDGameState)
	float GridSize;

	
	int32 RemainingBoxes;
protected:
	virtual void StartPlay() override;

	void CreateTown();
};
```
```c++
//TDDGameMode.cpp
#include "TDDGameMode.h"

void ATDDGameMode::StartPlay()
{
	Super::StartPlay();
	CreateTown();
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
```
모든 주택의 사이즈는 같고 정사각형 칸 안에 배치된다는 전제 하에, 구역의 가로 세로 영역을 배치될 구역 1칸의 크기로 나눈 값에 따라 한칸에 한칸씩 주택을 배치한다. 월드의 0,0,0 원점 좌표를 기준으로 주택들을 배치하며, x 혹은 y 좌표가 0이면 주택을 배치하지 않아 길같은 느낌을 준다.
![image](/assets/img/260421-crow.gif)

한편 모든 배달 지점이 활성화 되어 있다면 가장 가까운 지점만 계속해서 배달해도 게임을 클리어 할 수 있을것이다. 랜덤한 배달지점 하나만 매번 활성화 하도록 하는 기능을 구현해보자.
## 랜덤 적 배치

## 게임 클리어/오버 판정


![image](/assets/img/260421-crow.gif)

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
1