---
title: "Unreal: 인터페이스(Interface) & 델리게이트(Delegates)"
author: Jaeseong Kim
date: 2026-07-16 08:00:00 +0800
categories: [Unreal, Basics]
tags: [Unreal, Basics, Interface, Delegate]
---
## 인터페이스(Interface)
* 인터페이스는 **특정 클래스의 부모가 무엇인지와 관계없이, 특정한 기능을 수행할 수 있는지를 확인하고 호출하기 위한 약속**이다. 이는 클래스가 특정한 기능을 가지고 있고 구현했음을 강제하여 클래스 대신 인터페이스만 알더라도 기능을 호출할 수 있도록 한다. 
* 언어마다 형태는 다르지만, C++는 인터페이스 기능을 별도로 지원하지 않아 추상 클래스를 만들어 다중 상속하는 형식으로 대신한다. 언리얼은 C++을 사용은 하지만 인터페이스 기능을 추가적으로 제공한다. 다만 언리얼 리플렉션에 반영되기 위해 그 구조는 다른 프로그래밍 언어의 인터페이스 사용법과는 약간 다르다. 언리얼의 UObject는 다중상속을 지원하지 않기에 추상 클래스 다중 상속 같은 방법을 사용할 수 없다. 대신 인터페이스 클래스를 별도로 제공하고, 이를 하나의 클래스에서 여러개 상속하여 사용할 수 있다.

* ### 선언
    * 언리얼의 인터페이스는 `UInterface`를 상속하는 U 접두사 클래스와, 실제 함수를 선언하는 I 접두사 클래스 두 개로 만든다. U 접두사 클래스는 리플렉션과 Blueprint 노출을 위한 껍데기이고, 실제 C++ 상속과 구현은 I 접두사 클래스를 사용한다.

    ```c++
    // Interactable.h
    #pragma once

    #include "CoreMinimal.h"
    #include "UObject/Interface.h"
    #include "Interactable.generated.h"

    //변경하지 않는 인터페이스 자료형. 언리얼 리플렉션에 등록하기 위해 형식적으로 존재하는 클래스이다.
    //블루프린트에 노출하기위해선 BlueprintType을, 구현까지 하기 위해선 Blueprintable를 사용해야 한다.
    //처음 기본으로 생성한다면 MinimalAPI 키워드만 들어가있을 것이다.
    UINTERFACE(BlueprintType, Blueprintable)
    class UInteractable : public UInterface
    {
        GENERATED_BODY()
    };

    //실제로 구현하는 내용은 여기에 들어간다.
    class IInteractable
    {
        GENERATED_BODY()

    public:
        UFUNCTION(BlueprintNativeEvent, BlueprintCallable, Category = "Interaction")
        void Interact(AActor* Interactor);  //구현할 함수 선언
    };
    ```

    * `BlueprintNativeEvent`는 C++ 기본 구현과 Blueprint 재정의를 모두 허용한다. C++ 구현 함수 이름은 `Interact()`가 아닌 `Interact_Implementation()`이 된다. Blueprint에서만 구현하게 하고 싶다면 `BlueprintImplementableEvent`를 사용한다.

    ```c++
    // Door.h
    UCLASS()
    class ADoor : public AActor, public IInteractable
    {
        GENERATED_BODY()

    public:
        //인터페이스 상속 함수 구현
        virtual void Interact_Implementation(AActor* Interactor) override;
    };
    ```
    ```c++
    // Door.cpp
    void ADoor::Interact_Implementation(AActor* Interactor)
    {
        // 문 열기, 애니메이션 재생 등의 처리
    }
    ```

* ### 호출 방법
    * 대상이 인터페이스를 구현하는지 먼저 확인한다. `BlueprintNativeEvent`, `BlueprintImplementableEvent`는 Blueprint 구현도 존재할 수 있으므로 `IInteractable::Execute_Interact()`로 호출하는 편이 안전하다.

    ```c++
    //인터페이스 함수 호출 예시
    void AMyCharacter::TryInteract(AActor* Target)
    {
        if (!IsValid(Target)) return;
        
        //인터페이스 구현했는지 확인
        if (Target->Implements<UInteractable>())
        {
            //인터페이스에 달린 정적 함수에 해당 객체와 호출자인 이 객체를 함께 넘겨서 대리 실행
            IInteractable::Execute_Interact(Target, this);
        }
    }
    ```

    * `Cast<IInteractable>()`는 C++에서 인터페이스를 직접 구현한 객체를 얻을 때 사용할 수 있다. 다만 Blueprint만으로 인터페이스를 구현한 경우까지 처리하려면 `Implements<UInteractable>()`와 `Execute_` 호출을 사용해야 한다.

* ### Blueprint 구현을 포함한 안전 호출 상세
    * UINTERFACE() 매크로에 Blueprintable 키워드를 추가하면 블루프린트에서 인터페이스를 구현할 수 있다.
    * C++에서는 `Target->Implements<UInteractable>()`를 사용할 수 있고, Blueprint Function Library 방식으로는 `UKismetSystemLibrary::DoesImplementInterface()`를 사용할 수 있다. 두 방식 모두 Blueprint 구현을 확인할 수 있다.

    ```c++
    #include "Kismet/KismetSystemLibrary.h"

    void AMyTorchlight::NotifyFireDetected()
    {
        for (const TWeakObjectPtr<AActor>& Item : Items)
        {
            AActor* Target = Item.Get();
            if (!IsValid(Target)) continue;

            //UKismetSystemLibrary를 이용한 Interface 구현 여부 검사
            if (UKismetSystemLibrary::DoesImplementInterface(Target, UTestMyInterface::StaticClass()))
            {
                //인터페이스 함수 실행
                ITestMyInterface::Execute_OnFireDetected(Target, 100.0f, FVector::ZeroVector);
            }
        }
    }
    ```

    * `Execute_OnFireDetected()` 같은 함수는 인터페이스 함수가 `BlueprintNativeEvent` 또는 `BlueprintImplementableEvent`로 선언되어야 UHT가 생성한다. 이때 `BlueprintNativeEvent`는 호출 시 Blueprint 재정의가 있는지 먼저 확인하고, 없다면 C++의 `_Implementation()`을 호출한다.

    ```c++
    class ITestMyInterface
    {
        GENERATED_BODY()

    public:
        //BlueprintNativeEvent로 선언된 인터페이스 함수
        UFUNCTION(BlueprintCallable, BlueprintNativeEvent, Category = "Interaction")
        void OnFireDetected(float Temperature, FVector HitLocation);
    };

    // 구현 클래스의 헤더에선 이렇게 선언 후 실행
    virtual void OnFireDetected_Implementation(float Temperature, FVector HitLocation) override;
    ```

    * `BlueprintNativeEvent` 함수 선언 자체에는 `virtual`을 붙일 수 없다. 대신 UHT가 생성하는 호출 경로를 사용하고, C++ 기본 동작은 `_Implementation()`에 `virtual`로 구현한다. `OnFireDetected()` 예시에서 본문을 직접 정의하면 UHT 생성 코드와 충돌한다.
    * `Execute_` 호출은 객체가 유효하고 인터페이스를 구현한다는 전제가 필요하다. 내부 검사만 믿고 잘못된 객체를 넘기면 assert 또는 crash로 이어질 수 있다.
        * UObject를 받기에 객체의 유효성을 보장 못함.
        * Private 선언과 관계 없이 외부에서도 호출 가능함.
        * 이미 해제된 메모리거나 nullptr이면 치명적인 버그/크래시 가능.
        * `Execute_OnFireDetected()`에도 체크용 로직은 존재하지만 크래시를 낼 수 있기에 `UKismetSystemLibrary::DoesImplementInterface()` 혹은 `Implements<>()`로 더블체크 해주도록 하자.

* ### 사용 상황
    * 상호작용 가능한 문, 아이템, NPC, 레버
    * 데미지를 받을 수 있는 액터, 팀 판별이 가능한 액터
    * 서로 다른 Actor Component가 같은 기능을 제공해야 하는 경우
    * 의존성을 줄이고 싶은 경우. Character가 `ADoor`, `AChest` 같은 구체적인 타입을 직접 알 필요가 없어진다.

## 델리게이트(Delegate)
* 델리게이트는 한 객체에서 발생한 일을 다른 객체에 알리기 위한 콜백 구조다. 인터페이스가 "이 기능을 실행할 수 있는가?"를 묻는 구조라면, 델리게이트는 "이 일이 발생했다"를 구독자에게 알리는 구조다. 체력 변경을 HUD에 반영하거나, 게임 상태 변경에 여러 UI가 반응하게 만들 때 유용하다.

* ### 종류
    * Single Delegate - 하나의 함수만 연결한다. 반환값을 받을 수 있다.
    * Multicast Delegate - 여러 함수를 연결하고 한 번에 호출한다. 반환값은 사용할 수 없다.
    * Dynamic Delegate - 리플렉션을 사용한다. Blueprint 연결, `UPROPERTY`, 직렬화가 필요한 경우 사용한다. 일반 델리게이트보다 호출 비용이 크다.
    * UI와 Blueprint에 이벤트를 노출해야 하는 경우에는 `Dynamic Multicast Delegate`가 자주 쓰인다.

* ### 선언과 호출

    * 예시) 체력 컴포넌트에서 체력이 변할 때마다 이벤트 발행
        * `Broadcast()`가 호출되면 연결된 모든 함수가 실행된다. 구독자가 없어도 Multicast Delegate의 `Broadcast()`는 안전하게 호출할 수 있다.

    ```c++
    // HealthComponent.h
    //델리게이트 타입의 선언, float타입의 NewHealth를 매개변수로 호출될 때 같이 전달한다.
    DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FOnHealthChanged, float, NewHealth);

    UCLASS(ClassGroup = (Custom), meta = (BlueprintSpawnableComponent))
    class UHealthComponent : public UActorComponent
    {
        GENERATED_BODY()

    public:
        //미리 선언한 델리게이트 타입을 이용한 델리게이트 변수 선언
        UPROPERTY(BlueprintAssignable, Category = "Health")
        FOnHealthChanged OnHealthChanged;

        void SetHealth(float NewHealth);

    private:
        float Health = 100.0f;
    };
    ```

    ```c++
    // HealthComponent.cpp
    // 델리게이트 호출 예시, Health 값을 전달하며 델리게이트를 호출, 바인드 되어있는 함수들이 실행된다.
    void UHealthComponent::SetHealth(float NewHealth)
    {
        Health = FMath::Clamp(NewHealth, 0.0f, 100.0f);
        OnHealthChanged.Broadcast(Health);
    }
    ```

    

* ### 바인딩과 해제
    * Dynamic Delegate에 연결하는 함수는 `UFUNCTION()`이어야 하며, 파라미터 타입과 개수가 델리게이트 선언과 같아야 한다. Widget처럼 수명이 짧은 객체는 제거될 때 바인딩도 해제해야 중복 호출과 유효하지 않은 객체 참조를 막을 수 있다.

    ```c++
    // MyHealthWidget.h
    UCLASS()
    class UMyHealthWidget : public UUserWidget
    {
        GENERATED_BODY()

    public:
        //바인드할 예시 함수
        UFUNCTION()
        void UpdateHealth(float NewHealth);
        
        //바인딩 구현 함수
        void BindHealthComponent(UHealthComponent* InHealthComponent);

    protected:
        virtual void NativeDestruct() override;

    private:
        UPROPERTY()
        TObjectPtr<UHealthComponent> HealthComponent;
    };
    ```

    ```c++
    // MyHealthWidget.cpp
    // 함수를 델리게이트에 바인드한다.
    void UMyHealthWidget::BindHealthComponent(UHealthComponent* InHealthComponent)
    {
        if (HealthComponent)
            HealthComponent->OnHealthChanged.RemoveDynamic(this, &UMyHealthWidget::UpdateHealth);

        HealthComponent = InHealthComponent;

        if (HealthComponent)
            HealthComponent->OnHealthChanged.AddUniqueDynamic(this, &UMyHealthWidget::UpdateHealth);
    }

    //함수를 델리게이트에서 언바인드한다.
    void UMyHealthWidget::NativeDestruct()
    {
        if (HealthComponent)
            HealthComponent->OnHealthChanged.RemoveDynamic(this, &UMyHealthWidget::UpdateHealth);

        Super::NativeDestruct();
    }
    ```

* ### 인터페이스와 델리게이트 선택
    * Character가 대상에게 직접 "상호작용해라"라고 요청한다면 인터페이스가 맞다.
    * 체력이 변했음을 HUD, 사운드, 퀘스트 시스템처럼 여러 곳에 알려야 한다면 Multicast Delegate가 맞다.
    * 호출 대상이 하나이고 반드시 응답해야 한다면 일반 함수 또는 Single Delegate도 고려할 수 있다.
    * 델리게이트는 호출 순서와 연결 상태에 의존한다. 객체가 파괴되거나 UI가 다시 생성되는 시점에는 `RemoveDynamic`, `RemoveAll` 등으로 수명을 관리해야 한다.
