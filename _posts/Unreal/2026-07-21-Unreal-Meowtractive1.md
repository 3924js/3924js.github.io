---
title: "Unreal: 묘력만점(Meowtractive) 작업 내역 정리1"
author: Jaeseong Kim
date: 2026-07-21 08:00:00 +0800
categories: [Unreal, Basics]
tags: [Unreal, Basics, GAS, State Tree, Skeletal Mesh Merge]
---
## 묘력만점
![묘력만점 타이틀](/assets/img/260721-title.png)
[묘력만점 깃허브 레포지토리](https://github.com/NBcampUnrealTrack/8th-Team9-CH4-Project)

내일배움캠프 챕터 4 멀티플레이어 게임에선 고양이의 매력을 발산하여 매료한 사람들의 수로 경쟁하는 게임을 만들기로 했다. 처음에 내가 아이디어를 제시하고 전체적인 기획과 행인 클래스 제작, 맵 디자인과 나이아가라 이펙트 제작을 담당했다. 전체적인 작업과정과 선택, 난항등을 남겨놓고자 한다.

## 아이디어와 빌드업 과정
처음에 간단히 재밌을 것 같은 게임의 요소들을 합치고 구체화하는 과정을 거쳤다.
### 아이디어 떠올리기
![성 로맨스 학원](/assets/img/260721-motif.webp)

성 로맨스 학원(聖ロマンス学園)이라는 플래시게임이 있다. 흔히 눈빛보내기라는 이름으로 한국에서 과거 알려졌었다. 사람들을 매료시켜서 점수를 쌓는다는 개념은 꽤 재밌게 풀어나갈 수 있을 것 같았다. 기존엔 매력있는 여학생이 남학생들을 매료한다는 컨셉은 캐릭터를 바꿔야할 것 같았다. 그래서 비슷한 계열로 예쁨/귀여움을 대신해줄 고양이를 메인 컨셉으로 잡았다.

### 컨셉 구체화
![기획안 이미지](/assets/img/260721-concept.png)

멀티플레이어로 하기에 단순히 눈빛만 보내기엔 컨텐츠가 부족하다. 또한 싱글 게임으로 일방적으로 다른 경쟁자를 이기면서 플레이해도 됬던 기존의 눈빛보내기와는 다르게, 멀티플레이라면 일반적으로 한명이 다 이기고 다녀서는 안된다. 이에 플레이어마다 독립적인 매료 게이지 수치를 가지고 이를 가지고 행인을 둔 채로 경쟁할 수 있는 구조를 구상했다.


![맵 예시 이미지](/assets/img/260721-map.png)


<div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
  <figure>
    <img src="/assets/img/260721-cats.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>초기엔 캐릭터별로 컨셉에 따른 상성을 구상했다.</figcaption>
  </figure>
  <figure>
    <img src="/assets/img/260721-spotted.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>암살자형 캐릭터 점박이 나비</figcaption>
  </figure>
  <figure>
    <img src="/assets/img/260721-markel.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>원거리형 캐릭터 고등어 범이</figcaption>
  </figure>
  <figure>
    <img src="/assets/img/260721-cheese.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>탱커형 캐릭터 뚱냥이 치즈</figcaption>
  </figure>
</div>

### 플레이 루프 구상
눈빛 보내기로 매료한 행인 수를 통한 경쟁, 즉 땅따먹기의 변형게임이라는 큰 틀 안에서 다음과 같은 게임플로우를 구상했다.

1. 플레이어 스폰
  - 4개의 맵 스폰 구역에 **랜덤**하게 각 플레이어 스폰한다.
2. 게임 스타트
  - 시작하면 **최대한 많은 행인을 매료**하여 점수 획득
    - 행인은 랜덤하게 맵을 돌아다닌다.
      - 기본 행인은 특정 지역에 종속되지 않고 자유롭게 맵을 돌아다님
      - 특수 행인은 세팅에 따라 제한된 구역 반경 안에서만 돌아다니거나, 특정 경로를 이용해 움직이도록 구성한다.
    - 행인을 매료하면 1명당 10점을 얻으며, 특수행인은 점수 혹은 특수한 버프를 획득한다.
  - 플레이어는 **다양한 방법을 통해 행인을 매료**할 수 있다.
    - 기본 애교와 스킬로 사용하는 특수한 기능을 이용해 행인을 매료한다.
    - 행인은 플레이어마다 100이라는 최대 목표 매료도를 가지고, 각 플레이어는 0에서 시작해 애교나 스킬을 이용해 매료도를 100까지 올리면 매료된 것으로 판정한다.
    - 한번 매료된 행인도 다른 행인이 빼앗을 수 있다.
    - 동시에 여러 고양이가 한 행인을 매료하고자 할 때 매료도가 많이 쌓여있는 쪽이 우위를 가진다.
      - 2명 이상의 플레이어로부터 오는 매료도가 각각 동시에 증가할 수 있지만 가장 높은 매료도 값을 기준으로 다른 플레이어들은 매료도 추가에 감쇠가 적용된다.
  - 각 고양이는 매료와 공격등을 위한 기술들을 가진다.
    - 기본기술
      - 애교눈빛 - 행인 전용 원거리 공격
      - 냥냥펀치 - 근거리 공격
      - 대쉬 - 이동 겸 공격 기술
    - 특수기술
      - 장판 데미지, 타게팅 누킹 데미지 등 고양이마다 서로 다른 능력을 가진다.
  - 플레이어는 **적 고양이를공격해 KO상태**로 만들 수 있다.
    - 모든 고양이는 체력을 100으로 고정한다. 단 패시브 스킬등으로 수치가 바뀔 수 있다.
    - 체력이 0이 되면 고양이는 정신을 잃고 쓰러져, 10초 뒤에 선택한 스폰 지점에서 다시 스폰한다.
3. 게임오버
  - 게임 종료 시점에 개인/팀 단위로 **가장 높은 점수 획득한 플레이어가 승리**한다.

설명을 보면 알겠지만 2000-2010년대 온라인게임에서 많이 보이던 세션 형태의 배틀 게임, 가령 그랜드체이스, 겟엠프드같은 액션 게임과 같은 캐주얼하고 빠른 플레이를 원한 것을 알 수 있다. 분위기와 스타일에 있어서는 가장 최신까지 유지되는 게임중엔 스플레툰3가 있을 것 같다. 전투 자체는 목적보단 수단으로 두고, 상대를 처치하는 것 외에 계속해서 뒤집어질 수 있는 승리 조건을 두어 긴장감과 전략적 다양성을 유지시키는 것을 목표로 잡았다.

## 행인 구현
게임의 점수 역할이자 맵과 분위기의 큰 역할을 할 행인의 구현을 이후로 먼저 진행했다.

### 스테이트 트리를 이용한 방랑 패턴 구현
![ST](/assets/img/260721-st.png)

AI 행동 패턴을 만들기 위해 `StateTree`를 활용했다. 단순히 맵을 돌아다니는 간단한 행동 패턴을 기본으로 하기에, 코드에서 `FSM` 형태로 구현하거나 `Behavior Tree`를 이용할 수도 있었지만, 돌아다니다가 플레이어에가 반응하는 모드의 전환이 중요하고, 매 프레임마다 복잡하게 다양한 행동을 계산할 목적까진 필요 없기에 상태를 중심으로 판단하는게 적합하다 생각했고, 이에 따라 `StateTree`를 선택했다.

![STT](/assets/img/260721-stt.png)

기본 행인의 패턴은 앞서 언급한것처럼 크게 복잡하지 않다. 평소에는 주변 일정 반경의 `NavMeshBoundVolume`상에서 접근 가능한 지점을 랜덤하게 정하고, 해당 위치로 이동한다. 도달하면 랜덤하게 짧은 시간동안 멈췄다가 다시 새로운 지점을 찾아 움직이기를 반복한다. 이 때 랜덤한 위치를 획득해 움직이는 과정은 `StateTreeTask`로 분리해 `StateTree`에서 불러와 사용할 수 있도록 구성했다.

기획서에는 추가점수를 주거나 특수 효과를 부여하는등 맵에 오브젝트 성격을 띄는 특수 행인까지 기획에 있었는데, 특수행인을 위한 독립적인 이동동선을 설계하거나 패턴을 추가할 때의 유연성을 고려해 `StateTree`를 선택한 것도 있으나, 이는 이번 발표 시점에서 구현 대상까지는 아니였다.

### 매료 수치 세팅 - AS 에서 외부 수치로 변경
기획에 따르면 행인은 플레이어마다 개별적인 독립수치를 하나씩 가져야한다. 이를 `AttributeSet`을 이용해 저장하려고 시도했지만 실패했다. GAS 특성상 AS내에 동적으로 갯수가 임의도 달라지는 구조를 지원하지 않는다. TArray로 어트리뷰트를 선언은 할 수 있지만 GAS의 어트리뷰트로는 인식할 수 없다. 언리얼 리플렉션 구조가 어트리뷰트 데이터를 가리키는 것이 아닌 배열 그 자체만 가리키고, 원소는 리플렉션 식별 정보에 포함되지 않기 때문에 각 원소가 어트리뷰트로 인식되지 않으며, 어트리뷰트 매크로와 기존 GAS 함수들은 배열을 지원하지 않기에 호환도 불가능하다. 이에 따라 GE에서 일반적인 어트리뷰트처럼 접근해 수정이 불가능하다. AS 외부에 캐릭터 맴버 변수등으로 만들면 동적으로 갯수를 달리 할 수 있겠지만, 마찬가지로 GE에서 어트리뷰트처럼은 접근/수정 불가능하다.

동적으로 플레이어 갯수만큼 할당하는 기능은 빠질 수 없다고 판단, 이를 먼저 행인 캐릭터에 부착할 `AttractiveComponent`로 분리하여 외부에 저장하기로 했다. 이 때 배열 전체가 항상 복제되는 현상을 막고 네트워크 부하를 줄이기 위해 `TArray` 대신 `FastArray`를 활용했다. 이는 델타 복제(Delta Replication), 즉 차이점을 복제하는 방식으로 맵에 많이 배치되어 추적되어야 할 행인의 정보 업데이트가 네트워크에 부하를 주는 것을 경감한다. 또한 이에 대한 제한적인 접근과 수정 함수만 노출시키고 입력 검사를 추가해 매료도 데이터에 대한 과도한 접근을 차단하고 외부로부터 보호한다. 추후에 작업하여 부착한 행인 관련 나이아가라 VFX와 SFX 또한 이 컴포넌트에서 값에 변화에 따른 시점을 파악하여 호출할 수 있도록 구성했다.

```c++
USTRUCT(BlueprintType)
struct FMTAttractiveAmountEntry : public FFastArraySerializerItem
{
	GENERATED_BODY()

	UPROPERTY(BlueprintReadOnly, Category = "Attractive")
	TObjectPtr<APlayerState> PlayerState = nullptr;

	UPROPERTY(BlueprintReadOnly, Category = "Attractive")
	float AttractiveAmount = 0.f;

	// 서버 전용: 이 시각 전까지 해당 플레이어의 매료도 감소를 정지한다.
	double RegenResumeTimeSeconds = 0.0;
};

USTRUCT()
struct FMTAttractiveAmountContainer : public FFastArraySerializer
{
	GENERATED_BODY()

	UPROPERTY()
	TArray<FMTAttractiveAmountEntry> Entries;

	// FastArray의 변경된 항목만 네트워크로 직렬화한다.
	bool NetDeltaSerialize(FNetDeltaSerializeInfo& DeltaParams)
	{
		return FastArrayDeltaSerialize<FMTAttractiveAmountEntry, FMTAttractiveAmountContainer>(
			Entries,
			DeltaParams,
			*this);
	}

	void SetOwner(UMTAttractiveComponent* InOwner);
	void PostReplicatedReceive(const FFastArraySerializer::FPostReplicatedReceiveParameters& Parameters);

private:
	TWeakObjectPtr<UMTAttractiveComponent> Owner;
};

template<>
struct TStructOpsTypeTraits<FMTAttractiveAmountContainer>
	: public TStructOpsTypeTraitsBase2<FMTAttractiveAmountContainer>
{
	enum
	{
		WithNetDeltaSerializer = true
	};
};
```

한편 `AttributeSet`외부에 선언한 변수에 대해 `Gameplay Effect`에서 일반적인 방법으로 접근하고 수정하는건 불가능하기에 `ExecCalc`를 사용했다. 코드로 직접 연산을 정의할 수 있기 때문에 외부 변수에 대해 접근할 수 있고, 이번엔 직접 수정하기 보단 최종적으로 반영항 보정된 변경값을 계산하고 `AttractiveComponent`로 전달해 컴포넌트 측에서 변경하도록 하는 어댑터처럼 구성했다. 이렇게 해서 보정을 위한 특수 계산 수행, GAS 태그 부여, 매료치 변경이라는 동작을 충돌 없이 수행하도록 구성했다.

```c++
void UExecCalc_PedestrianDamage::Execute_Implementation(const FGameplayEffectCustomExecutionParameters& ExecutionParams, FGameplayEffectCustomExecutionOutput& OutExecutionOutput) const
{
	const FGameplayEffectSpec& Spec = ExecutionParams.GetOwningSpec();
	UAbilitySystemComponent* TargetASC = ExecutionParams.GetTargetAbilitySystemComponent();

	if (!TargetASC)
	{
		return;
	}

	const FGameplayTag DamageTag = FGameplayTag::RequestGameplayTag(FName("Data.Damage"));
	const float BaseDamage = Spec.GetSetByCallerMagnitude(DamageTag, false, 10.f);

	if (BaseDamage <= 0.0f)
	{
		return;
	}

	APlayerState* SourcePlayerState = nullptr;
	if (AActor* Instigator = Spec.GetContext().GetOriginalInstigator())
	{
		if (const APawn* Pawn = Cast<APawn>(Instigator))
		{
			SourcePlayerState = Pawn->GetPlayerState();
		}
		else if (const AController* Controller = Cast<AController>(Instigator))
		{
			SourcePlayerState = Controller->PlayerState;
		}
	}

	float HighestOtherAmount = 0.f;
	float MaxAttractiveAmount = 100.f;
	if (const AMTPedestrianBase* Pedestrian = Cast<AMTPedestrianBase>(TargetASC->GetAvatarActor()))
	{
		if (const UMTAttractiveComponent* AttractiveComponent = Pedestrian->GetAttractiveComponent())
		{
			HighestOtherAmount =
				AttractiveComponent->GetHighestAttractiveAmountExcluding(SourcePlayerState);
			MaxAttractiveAmount = AttractiveComponent->GetMaxAttractiveAmount();
		}
	}

	// 다른 플레이어 최고 매료도 0 -> 100%, 30 -> 85%, 100 -> 50%.
	const float NormalizedOtherAmount = MaxAttractiveAmount > 0.f
		? FMath::Clamp(HighestOtherAmount / MaxAttractiveAmount, 0.f, 1.f)
		: 0.f;
	const float DamageScale = FMath::Lerp(1.f, 0.5f, NormalizedOtherAmount);
	const float FinalDamage = BaseDamage * DamageScale;

	OutExecutionOutput.AddOutputModifier(
		FGameplayModifierEvaluatedData(
			UMTPedestrianAttributeSet::GetAttractiveAmountAttribute(),
			EGameplayModOp::Additive,
			FinalDamage
		)
	);
}
```

### Skeletal Mesh Merge 를 이용한 모듈형 캐릭터 생성 기능
캐릭터가 모두 같은 모습이라면 이상하게 느껴질 것이다. 더 자연스러운 거리와 환경을 연출하기 위해서 캐릭터의 옷이나 신발등 파츠별로 갈아끼울 수 있는 기능을 넣어서 랜덤한 모습의 행인으로 거리를 채우는 기능을 만들었다. 이전에 쓴 글에도 한번 적었지만 여러 메시를 하나의 메시처럼 움직이게 하는데는 몇가지 방법이 있다. 그중에서 이번엔 `Skeletal Mesh Merge`를 사용했는데, 이는 `Leader Pose Component`를 설정하는 방법이나 `CopyMeshPose`를 사용하는 방법들에 비해 1개의 메시처럼 취급되는 새로운 메시를 만드는 방식 특성상 드로우콜과 애니메이션등 이후 사용시의 비용을 감소시킨다. 처음 생성할 때 비용이 다른 방법들에 비해 큰편이긴 하나, 게임 시작시 행인을 스폰하고 게임이 끝날때까지 사라지거나 달라지지 않은 채 계속 걸어다닐 것을 생각하면, 스폰 이후의 비용이 더 중요한 지금 상황에 적합한 선택이라 할 수 있다.

![GameModeSettings](/assets/img/260721-gamemode.png)

모듈로 구성할 메시와 정보 목록은 `AMTMatchGameMode`에 설정해놓는다. 스켈레톤 크기/구조가 다른 특성상 남성과 여성을 먼저 분리시키고 각 파트에 해당하는 메시, 그 메시에 넣을 머티리얼 슬롯 이름과 머티리얼을 각각 미리 넣어놓을 수 있도록 구조체를 만들었다. 이에 따라 다른 맵을 추가하더라도 파생되는 게임모드를 따로 만들어 별도의 행인 외형 목록을 맵별로 구성할 수 있을 것이다. 게임이 시작되면 정해진 확률에 따라 남성과 여성을 먼저 선택하고, 이후 메시와 머티리얼을 랜덤으로 선택해 행인의 메시를 구성, 정해진 캐릭터 클래스를 스폰 한 후 그 캐릭터 오브젝트의 스켈레탈 메시로 할당한다. 이때 같은 조합이 추후에도 나올 수 있고, 이를 다시 병합할 때의 비용을 줄이기 위해, 게임인스턴스에 서브시스템으로 이를 캐싱해둘 저장소를 마련했고, 같은 조합이 나왔을 때 캐싱된 조합에 있다면 새로 병합하는 대신 캐싱된 메시를 불러와 비용을 절감하도록 했다.

![Ped](/assets/img/260721-ped.png)

다만 프로젝트 발표 시점에는 사용되지 못했다. 기능은 모두 동작했지만 기능 구현 후에 모듈형 캐릭터 외형에 대해 의견이 갈려서 다른 단순 캐릭터로 변경됬기 때문이다. 팀 차원에서 진행 전에 의견과 방향성을 검토하고 조율하는 것의 중요함이 드러나는 부분이라 할 수 있겠다.