---
title: "Unreal: FastArray"
author: Jaeseong Kim
date: 2026-07-02 08:00:00 +0800
categories: [Unreal, Basics]
tags: [Unreal, Basics, Fast Array]
---
## Fast Array
언리얼에서 변수의 서버-클라이언트 간의 복제는 Replicated 를 UPROPERTY로 선언할 때 같이 선언함으로서 설정된다. 정수, 실수같은 원시자료형으로 선언된 자료는 복제에 별 부담이 없겠지만, 배열을 복제해야한다면 어떨까? 일반적인 Replicated를 사용하면 배열의 원소중 하나만 바뀔 때 개별 원소만 업데이트 하고 콜백하는 기능을 제공하지 않고, 이는 원소단위로 변할 일이 많을 때 복제 비용 증가와 네트워크 상의 부하로 이어질 수 있다.

이 때 고려해볼 수 있는 방법은 Fast Array를 사용하는 것이다. 언리얼에서 제공하는 네트워크 복제에 최적화된 구조로, 차이점만 갱신하는 방식인 델타 복제(Delta Serialization)을 활용하는 구조를 가진다. Replication을 대체하는 구조는 아니고, TArray에 추가적인 기능을 더해 네트워크 상 복제를 유연하게 하는 확장 기능을 가진 구조로 볼 수 있다. 

## 선언
FastArray는 상속을 통해 만들 수 있고, 원소와 배열 2개의 구조체를 상속받아 만든다.
```c++
//원소(아이템) 구조체
USTRUCT()
struct FMyItem : public FFastArraySerializerItem
{
    GENERATED_BODY()

    UPROPERTY()
    int32 Value = 0;

    void PreReplicatedRemove(const struct FMyArray& InArraySerializer);
    void PostReplicatedAdd(const struct FMyArray& InArraySerializer);
    void PostReplicatedChange(const struct FMyArray& InArraySerializer);
};
```
```c++
//배열 구조체
USTRUCT()
struct FMyArray : public FFastArraySerializer
{
    GENERATED_BODY()

    UPROPERTY()
    TArray<FMyItem> Items;

    bool NetDeltaSerialize(FNetDeltaSerializeInfo& DeltaParms)
    {
        return FFastArraySerializer::FastArrayDeltaSerialize<FMyItem, FMyArray>(
            Items,
            DeltaParms,
            *this
        );
    }
};
```

이전 2개 구조체를 선언하는 것과 더불어, 배열 구조체에는 NetDeltaSerialize()를 구현하고, TStructOpsTypeTraits를 통해 이 구조체가 NetDeltaSerialize를 사용한다는 사실을 엔진의 리플렉션/복제 시스템에 알려주어야 한다. 직렬화 설정같은 기본적인 값들을 설정해놓은 구조체 베이스로, 상속해 선언한 후 필요한 값만 덮어씌우면 편하게 설정할 수 있다. 템플릿 특수화를 이용하기에 TStructOpsTypeTraits<>라는 이름이 달라져서는 안된다. 오래된 버전에선 타입 인자를 안받던 TStructOpsTypeTraitsBase가 있었는데, UE4 후기 버전 즈음부터 타입인자를 받는 TStructOpsTypeTraitsBase2<>로 주로 사용하게 됬다.
```c++
//FMyArray 타입에 대한 StructOps 인자 설정
template<>
struct TStructOpsTypeTraits<FMyArray> : public TStructOpsTypeTraitsBase2<FMyArray>
{
    enum
    {
        WithNetDeltaSerializer = true
    };
};
```

## 원소의 변경
일반적인 Replicated 설정된 데이터와는 다르게, 원소 단위의 변경/수정은 자동으로 추적되지 않는다. 대신, 이를 dirty표시를 통해 델타 복제 시스템이 변경이 있음을 알려주어야 한다.
```c++
// 아이템 값 변경
Items[Index].Value = 10;
MarkItemDirty(Items[Index]);

// 아이템 추가
Items.Add(NewItem);
MarkItemDirty(Items.Last());

// 아이템 삭제
Items.RemoveAt(Index);
MarkArrayDirty();
```
삭제같은 배열구조의 변경은 MarkItemDirty() 대신 MarkArrayDirty()를 호출해 배열 구조가 변경되었음을 델타 복제 시스템에 알려야 한다. 이것이 꼭 배열 전체를 매번 통째로 다시 보낸다는 의미는 아니다.


원소 구조체에는 변경이 생겼을 때 복제의 결과를 얻을 수 있는 콜백 이벤트를 만들어둘 수 있다. 이를 통해 값이 변경되었을 때 클라이언트에 동작이 따라올 수 있도록 구현할 수 있다. 보면 멀티캐스트RPC와 역할이 겹치지 않을까 싶은데, 상태, 값 동기화같은 내용이면 콜백을 쓰는게 상태의 관리 측면에서 더 좋을것이고, 이펙트같은 일회성 동작으로 충분한 경우엔 멀티캐스트가 더 간편할것이다.
```c++
void FMyItem::PostReplicatedAdd(const FMyArray& InArraySerializer)
{
    // 클라이언트에 새 아이템이 추가됨
}

void FMyItem::PostReplicatedChange(const FMyArray& InArraySerializer)
{
    // 기존 아이템 값이 변경됨
}

void FMyItem::PreReplicatedRemove(const FMyArray& InArraySerializer)
{
    // 아이템이 제거되기 직전
}
```

일반 배열을 Replicate 할때와 마찬가지로, 인덱스에 기반하는 동작은 위험할 수 있다. 델타 복제 과정에서 인덱스와는 다른 별도의 식별자인 ReplicationID가 사용되고, 이에 따라 서버와 클라이언트에서 동일한 인덱스는 고유한 식별자가 아니며 순서도 섞일 수도 있다. 확실하게 하고자 한다면 추가적으로 확인 가능한 값이나 ID를 아이템 구조체에 추가해 확인해야 하겠다.

액터에서 추가해 사용하는 것은 기존과 동일하게 Replicated 혹은 OnRep_ 을 통해 등록해주어야 한다.
```c++
//MyActor.h
UCLASS()
class AMyActor : public AActor
{
    GENERATED_BODY()

public:
    UPROPERTY(Replicated)
    FMyArray MyArray;

    virtual void GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const override;
};
```
```c++
//MyActor.cpp
//...
void AMyActor::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const
{
    Super::GetLifetimeReplicatedProps(OutLifetimeProps);

    DOREPLIFETIME(AMyActor, MyArray);
}
```
