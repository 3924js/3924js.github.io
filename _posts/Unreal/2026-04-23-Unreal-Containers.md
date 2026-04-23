---
title: "Unreal: 자료 컨테이너 - TArray, TMap, TSet"
author: Jaeseong Kim
date: 2026-04-23 8:00:00 +0800
categories: [Unreal, Project]
tags: [Unreal, Project, Container, TArray, TMap, TSet]
---
## TObjectPtr<>
* UE4까지 존재하던 원시 포인터 (UObject*)를 대체하기 위해 추가된 탬플릿 기반의 포인터이다.
    * 지연 로딩(Lazy Loading): TObjectPtr를 사용해 선언된 데이터는 실제로 필요하게 된 떄 메모리에 로딩된다. 다른 언어에서 있는 지연 평가(Lazy Evaluation)과 유사한 개념으로, 한번에 과한 메모리 부하를 줄여주는 한편, 간단한 호출이나 선언의 경우에는 불필요한 복잡함일 수도 있다.
    * 엑세스 트래킹(Access Tracking): 어디에서 데이터가 호출되는지 GC와 연계되어 더 정밀한 측정을 제공한다. 간단한 지역변수의 경우 지연 로딩과 마찬가지로 불필요한 기능일 수도 있다.
    * 지역 변수/짧은 수명이 확실한 경우에는 원시 포인터를 사용해도 무방하며, UPROPERTY()헤더가 붙는다면 TObjectPtr을 사용하는게 좋을 것이다.
* 실제 출시품으로 패키징할 떄는 내부적으로 원시포인터로 변환되기에 의미 없다. 위 제공되는 기능들도 단순 프로그램을 실행하는데 있어서는 필요없기에 다 사라지며, 프로그램의 성능을 목적으로 하는게 아닌 개발과정에서의 관리가 주 목적이라 봐야한다.
* TObjectPtr<>을 반복문에서 호출할 떄는 auto* 대신 auto&가 주소값의 복사 로직이 빠지기에 성능에 더 좋다.

## TSubclassOf<>
* 클래스의 유형 정보를 담기 위해 사용하는 탬플릿 클래스다.
* UClass* 와 비슷하게 클래스에 대한 하드 레퍼런스를 제공하는데, UCLass*는 어떤 클래스나 들어갈 수 있지만, TSubclassOf<>는 특정한 클래스만 가능하도록 제한시켜준다.
* 탄환 생성등 특정한 클래스를 미리 할당해놔야 하는 데이터에 많이 사용된다.

## TArray<>
* C++에서 제공하는 std::vector의 언리얼 버전이다. 구조도 기본적으로 동일하지만, 언리얼 리플렉션을 위해 재정의된 자료형이라 보면 된다.
* 장단점도 vector와 공유된다.
    * 랜덤접근이 $O(1)$으로 빠르다.
    * 자동으로 정렬되지 않는다.
    * 배열 끝에 삽입은 $O(1)$으로 빠르지만, 배열 중간에 삽입 삭제는 $O(n)$으로 비교적 느리다.
* 함수
    * Add()
    * Emplace() -Add()와 유사하지만 더 좋은 성능 제공, 암시적인 변환을 포함하기에 추천하진 않음.
    * Remove()
    * AddUnique()
    * Find()
    * Num()
    * Empty()
    * IsEmpty()
    * Sort()

## TMap<>
* std::map의 언리얼 버전이다. 전반적인 사용법은 비슷하지만 차이점도 있다.
    * 트리 기반이 아닌 해시 테이블 기반이다. 그런 점에선 map보단 unordered_map과 유사하다고도 할 수 있다.
    * 없는 값에 접근하면 새로 추가하는게 아니라 에러를 발생시킨다. 때문에 있는지 확인하기 위해서는 Contains()으로 확인해야 한다.
* 함수
    * Add()
    * Emplace() -Add()와 유사하지만 더 좋은 성능 제공, 암시적인 변환을 포함하기에 추천하진 않음.
    * FindOrAdd() -AddUnique()와 유사한 기능으로, 찾아보고 있으면 있는 value를 없으면 새로운 key로 추가해서 반환해준다.
    * Remove()
    * Find()
    * Num()
    * Empty()
    * IsEmpty()
## TSet<>
* std::set의 언리얼 버전이다. TMap과 유사하지만 key값으로만 이루어져 있다.
    * TMap처럼 해시테이블 기반으로 되어있다.
* 함수
    * Add()
    * Emplace()
    * FindOrAdd()
    * Remove()
    * Find()
    * Num()
    * Empty()
    * IsEmpty()


* C++의 컨테이너들처럼 반복자를 지원하는데, 언리얼 버전인 TIterator를 제공한다. for문 처럼 맥락이 명백한 경우라면 auto로 할당받아 편하게 사용할 수 있겠다.
* 위 3개 배열 모두 저장을 위해 자료가 없어도 미리 메모리를 할당해놓는 슬랙(Slack)을 가지게 된다. Compact로 슬랙을 컨테이너 끝으로 몰아놓을 수 있고, 끝단의 슬랙들을 제거하는 Shrink()를 함께 사용해 메모리 효율을 높일 수 있다.
## 간단한 인벤토리 예시
데이터 테이블과 조합하면 효율적인 인벤토리를 구현할 수 있다. 아이템 객체들을 직접 가지고 있는 경우보다 아이템의 코드만 컨테이너에 가지고 있다가 인벤토리나 아이템을 불러올 때 테이블을 참조하는 식이다. TMap을 선택하면 자동정렬된, 아이템 코드와 갯수로 이루어진 효율적인 인벤토리를 제공할것이고, TArray를 사용하면 아이템이 각각의 인벤토리 칸을 차지하는 자연스러운 구조를 만들 수도 있을 것이다. 지금은 TArray로 아이템의 저장을, TMap으로 아이템의 출력을, TSet으로 아이템 사용 판정에 쓰일 칭호들을 저장을 맡도록 사용했다.

```c++
//InventoryTestActor.h
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "ItemRow.h"
#include "InventoryTestActort.generated.h"

class AItemBase;
UCLASS()
class NBC_INVENTORY_API AInventoryTestActort : public AActor
{
	GENERATED_BODY()
	
public:	
	// Sets default values for this actor's properties
	AInventoryTestActort();

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;

	UFUNCTION(BlueprintCallable, Category = "Inventory")
	void AddItem(FName NewItem);

	UFUNCTION(BlueprintCallable, Category = "Inventory")
	void UseItem(FName UsedItem);

	UFUNCTION(BlueprintCallable, Category = "Inventory")
	void ShowInventory();

	UFUNCTION(BlueprintCallable, Category = "Inventory")
	void AddTitle(FName NewTitle);

	UFUNCTION(BlueprintCallable, Category = "Inventory")
	void RemoveTitle(FName RemovedTitle);

	UFUNCTION(BlueprintCallable, Category = "Inventory")
	void ShowTitles();

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Inventory")
	UDataTable* ItemTable;

	FItemRow* FindItem(FName ItemCode);
	TArray<FName> Inventory;
	TSet<FName> Titles;
public:	

};
```
```c++
//InventoryTestActor.cpp
#include "InventoryTestActort.h"

// Sets default values
AInventoryTestActort::AInventoryTestActort()
{
 	// Set this actor to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	
}

// Called when the game starts or when spawned
void AInventoryTestActort::BeginPlay()
{
	Super::BeginPlay();
	//타이틀 추가
	AddTitle(FName("CEO"));
	AddTitle(FName("Gardner"));
	ShowTitles();
	RemoveTitle(FName("aaa"));	//없는 타이틀 삭제 안됨
	ShowTitles();
	RemoveTitle(FName("CEO"));	//Gardner만 남음
	ShowTitles();

	//인벤토리 조작
	AddItem(FName("CEOChair"));
	AddItem(FName("GardnerBush"));
	AddItem(FName("GardnerBush"));
	AddItem(FName("GardnerShovel"));	//테이블에 없는 아이템 추가 안됨
	ShowInventory();
	UseItem(FName("GardnerBush"));	//Gardner 호칭으로 사용 가능
	ShowInventory();
	UseItem(FName("GardnerShovel"));	//없는 아이템 사용 안됨
	UseItem(FName("CEOChair"));
}

void AInventoryTestActort::AddItem(FName NewItemCode)
{
	UE_LOG(LogTemp, Warning, TEXT("Try Adding an item: %s"), *(NewItemCode.ToString()));
	//Add new item to TArray
	FItemRow* Item = FindItem(NewItemCode);
	if (Item == nullptr) {
		UE_LOG(LogTemp, Warning, TEXT("Cannot add an unexisting item! - %s"), *(NewItemCode.ToString()));
		return;
	}
	Inventory.Add(NewItemCode);
	UE_LOG(LogTemp, Warning, TEXT("Item Added: %s"), *(Item->ItemName.ToString()));
}

void AInventoryTestActort::UseItem(FName UsedItemCode)
{
	UE_LOG(LogTemp, Warning, TEXT("Try using an item: %s"), *(UsedItemCode.ToString()));
	//Add new item to TArray
	FItemRow* Item = FindItem(UsedItemCode);
	if (Item == nullptr) {
		UE_LOG(LogTemp, Warning, TEXT("Cannot add an unexisting item! - %s"), *(UsedItemCode.ToString()));
		return;
	}
	//Remove a item from the TArray
	if (Titles.Find(Item->RequiredTitle) == nullptr) {
		UE_LOG(LogTemp, Warning, TEXT("Does not have required title! - %s"), *(Item->RequiredTitle.ToString()));
		return;
	}
	Inventory.Remove(UsedItemCode);
	UE_LOG(LogTemp, Warning, TEXT("Item Used: %s"), *(Item->ItemName.ToString()));
}

void AInventoryTestActort::ShowInventory()
{
	//Count Items using TMap
	TMap<FName, int32> InventoryDisplayer;
	for (FName Item : Inventory) {
		InventoryDisplayer.FindOrAdd(Item)++;
	}
	//Display Contents in TMap
	UE_LOG(LogTemp, Warning, TEXT("Displaying Items:"));
	for (auto Item : InventoryDisplayer) {
		UE_LOG(LogTemp, Warning,TEXT("%s: %d"), *(Item.Key.ToString()), Item.Value);
	}
}

void AInventoryTestActort::AddTitle(FName NewTitle)
{
	Titles.Add(NewTitle);
	UE_LOG(LogTemp, Warning, TEXT("Title Added: %s"), *(NewTitle.ToString()));
}

void AInventoryTestActort::RemoveTitle(FName RemovedTitle)
{
	Titles.Remove(RemovedTitle);
	UE_LOG(LogTemp, Warning, TEXT("Title Removed: %s"), *(RemovedTitle.ToString()));
}

void AInventoryTestActort::ShowTitles() {
	//Display Titles
	UE_LOG(LogTemp, Warning, TEXT("Owning Titles:"));
	for (FName Title : Titles) {
		UE_LOG(LogTemp, Warning, TEXT("%s"), *(Title.ToString()));
	}
}

FItemRow* AInventoryTestActort::FindItem(FName ItemCode)
{
	if (!IsValid(ItemTable)) {
		UE_LOG(LogTemp, Warning, TEXT("No Item Table!"));
		return nullptr;
	}
	return ItemTable->FindRow<FItemRow>(ItemCode, TEXT(""));
}
```
```C++
//ItemRow.h
#pragma once

#include "CoreMinimal.h"
#include "ItemRow.generated.h"

USTRUCT(BlueprintType)
struct FItemRow : public FTableRowBase
{
	GENERATED_BODY()
public:
	UPROPERTY(EditAnywhere, BlueprintReadWrite)
	FName ItemName;
	UPROPERTY(EditAnywhere, BlueprintReadWrite)
	FName RequiredTitle;
	UPROPERTY(EditAnywhere, BlueprintReadWrite)
	UStaticMesh* ItemMesh;
};
```

![image](/assets/img/260423-table.png)
![image](/assets/img/260423-result.png)
UI까지 구현해보고 싶었는데 3D모델을 UI창에 띄울 방법을 아직까진 잘 모르겠다. 필요하다면 구현만 해서 연결해도 나중에 쓸만한 구조일 것이다.