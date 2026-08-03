---
title: "Unreal: 묘력만점(Meowtractive) 작업 내역 정리2"
author: Jaeseong Kim
date: 2026-07-23 08:00:00 +0800
categories: [Unreal, Basics]
tags: [Unreal, Basics, GAS, Niagara, Level Design]
---
이전에 작성한 기획, 행인 구현 외에도 레벨 디자인과 기능 수정, VFX 제작과 연동 또한 진행했다.
## 레벨 디자인

<div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
    <figure>
        <img src="/assets/img/260723-map1.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption style="text-align: center;">초기 맵 컨셉아트</figcaption>
    </figure>
    <figure>
        <img src="/assets/img/260723-map2.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption style="text-align: center;">공원형 맵 샘플</figcaption>
    </figure>
</div>

* 본 게임에 사용할 레벨을 제작하는 과정도 거쳤다. 초기 기획안에 맞추어 컨셉을 만들었다. 완벽하게 따라할 수는 없겠으나 일단 마믈 상점가, 공원정도를 컨셉으로 잡았다.

<div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
    <figure>
        <img src="/assets/img/260723-level1.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption style="text-align: center;">맵 전경</figcaption>
    </figure>
    <figure>
        <img src="/assets/img/260723-level2.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption style="text-align: center;">상점가같은 가벼운 분위기를 연출하고자 했다.</figcaption>
    </figure>
</div>

* 비교적 원경에서 제한적인 각도로 보일 물체들의 경우 2D 텍스처로 제작해 임포스터로 배치했다.
* 이전에 기획에 첨부한 것 같은 이미지를 기반으로 격자 형태의 맵을 구상했다. 이를 바탕으로 팀원이 구한 에셋 일부와 팹에서 구한 무료 에셋들을 바탕으로 맵을 구성했다.
    * 이때 초기 계획에선 20-30초정도 걸어서 반대편에 도달할 정도의 맵사이즈를 생각했으나, 게임플레이 시간에 비해 사이즈가 너무 크다고 판단, 맵 사이즈는 조정과 최대한 자연스럽게 에셋을 배치하고 조정하는 과정을 거쳤다.

<div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
    <figure>
        <img src="/assets/img/260723-level3.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption>동일한 에셋도 회전이나 위치를 달리해 조합하여 다채롭게 구성했다.</figcaption>
    </figure>
    <figure>
        <img src="/assets/img/260723-level4.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption>원경에서 보일 것들은 임포스터 이미지를 제작해 배치했다.</figcaption>
    </figure>
</div>

## 매료빔 판정 수정
<div style="display: grid; grid-template-columns: repeat(1, 1fr); gap: 10px;">
    <figure>
        <img src="/assets/img/260723-unchanged.gif" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption style="text-align: center;">판정 변경 전의 빔 공격 사용 및 이펙트 출력 모습</figcaption>
    </figure>
</div>
* 처음에 먼저 만들어진 매료빔은 카메라를 기준으로 발사하게 되어있었다.
    * 이는 TPS 시점상 캐릭터에서 출발하는게 아니라 캐릭터 뒷편부터 출발하기에, 조준이 캐릭터와 카메라 사이에 물체가 있으면 해당 지점을 공격지점으로 판정해, 카메라가 가려질 때 원래 공격하던 타겟을 놓히는 문제가 있었다.
    * 또한 해당 과정에서 트레이스 결과를 나이아가라 이펙트에서 빔 시작점/끝점 조정을 위해 받아가는데, 조준대상인 행인 중앙 위치로 끝점이 스냅되듯이 딸려가 끊기는 듯 보이는 현상이 있었다.
* 이 현상을 해결하고 일반적인 TPS스러운 자연스러운 판정을 위해 단계를 2개로 나누었다.
    * 카메라에서 발사하는 트레이스는 실제 공격을 판단하지 않도록 바꾼다. 대신, 건물 벽이나 행인등 실제로 부딫히는 지점만 확인하도록 바꾼다.
        * 이 때 내적을 이용해 충돌 위치를 파악하고, 카메라와 캐릭터 사이에 있는 물체에 충돌했으면 무시한다.
    
    ```c++
    //bool UGA_AttractiveBeam::TraceBeam()
    //...
    FVector CamLocation;
    FRotator CamRotation;
    PC->GetPlayerViewPoint(CamLocation, CamRotation);
    const FVector AimDirection = CamRotation.Vector();
    const FVector CameraTraceEnd = CamLocation + (AimDirection * TraceDistance);
    OutTraceStart = GetAttractiveBeamFXStartLocation();

    // 1) 카메라 정면 LineTrace로 시각적 조준점을 계산한다.
    // 월드 Trace에서는 Pawn을 제외하고 벽/지형만 찾는다.
    FCollisionQueryParams CameraWorldParams;
    CameraWorldParams.AddIgnoredActor(Avatar);
    for (TActorIterator<APawn> It(World); It; ++It)
    {
        CameraWorldParams.AddIgnoredActor(*It);
    }

    FVector CameraTarget = CameraTraceEnd;
    FHitResult WallHit;
    if (World->LineTraceSingleByChannel(WallHit, CamLocation, CameraTraceEnd, ECC_Visibility, CameraWorldParams))
    {
        CameraTarget = WallHit.ImpactPoint;
    }

    // Pawn도 반경 없는 LineTrace로 찾는다. 카메라와 플레이어 사이 Hit는 제외한다.
    FCollisionQueryParams CameraPawnParams;
    CameraPawnParams.AddIgnoredActor(Avatar);
    FCollisionObjectQueryParams PawnObjectParams;
    PawnObjectParams.AddObjectTypesToQuery(ECC_Pawn);
    TArray<FHitResult> CameraPawnHits;
    World->LineTraceMultiByObjectType(
        CameraPawnHits, CamLocation, CameraTraceEnd, PawnObjectParams, CameraPawnParams);

    //대상 판별용 내적 계산, 플레이어에서 발사할 트레이스의 목표지점 설정.
    const float PlayerDepth = FVector::DotProduct(Avatar->GetActorLocation() - CamLocation, AimDirection);
    for (const FHitResult& Hit : CameraPawnHits)
    {
        AActor* HitActor = Hit.GetActor();
        if (!IsValid(HitActor))
        {
            continue;
        }

        //플레이어 위치와 충돌 지점을 내적값을 바탕으로 거리 비교, 플레이어보다 카메라에 가까울시 무시
        const float HitDepth = FVector::DotProduct(Hit.ImpactPoint - CamLocation, AimDirection);
        if (HitDepth <= CameraPlayerDepthTolerance
            || HitDepth < PlayerDepth - CameraPlayerDepthTolerance)
        {
            continue;
        }

        if (Hit.Distance < FVector::Distance(CamLocation, CameraTarget))
        {
            CameraTarget = Hit.ImpactPoint;
        }
        break;
    }

    // 카메라 뒤 장애물은 플레이어 시작 빔을 막지 않는다.
    OutBeamEnd = FVector::DotProduct(CameraTarget - OutTraceStart, AimDirection) > 0.f
        ? CameraTarget
        : OutTraceStart + (AimDirection * TraceDistance);
    //...
    ```

    * 해당 지점까지 고양이에서 트레이스를 다시 발사한다. 이 판정으로 실제 공격을 할 수 있을지 없을지 결정한다.

    ```c++
    //bool UGA_AttractiveBeam::TraceBeam()
    //...
    FHitResult WorldHit;
    bOutHitWorld = World->LineTraceSingleByChannel(
        WorldHit,
        OutTraceStart,
        OutBeamEnd,
        ECC_Visibility,
        CameraWorldParams);
    if (bOutHitWorld)
    {
        OutBeamEnd = WorldHit.ImpactPoint;
    }

    // 3) 데미지 판정만 소켓→시각 끝점 Sphere Sweep으로 수행한다.
    // 판정 Hit로 OutBeamEnd를 덮어쓰지 않는다.
    FCollisionQueryParams DamageParams;
    DamageParams.AddIgnoredActor(Avatar);
    TArray<FHitResult> DamageHits;
    World->SweepMultiByObjectType(
        DamageHits,
        OutTraceStart,
        OutBeamEnd,
        FQuat::Identity,
        PawnObjectParams,
        FCollisionShape::MakeSphere(BeamRadius),
        DamageParams);

    //충돌들에 대해 행인 대상일지 확인.
    for (const FHitResult& Hit : DamageHits)
    {
        AActor* HitActor = Hit.GetActor();
        if (!IsValid(HitActor))
        {
            continue;
        }
        bOutHitActor = true;
        if (AMTPedestrianBase* Pedestrian = Cast<AMTPedestrianBase>(HitActor))
        {
            OutPedestrian = Pedestrian;
            break;
        }
    }
    ```

![change](/assets/img/260723-attracted.gif)
* 이 바뀐 판정으로 빔에 적용한 이펙트가 사물에 가려질때 해당 사물로 타겟이 바뀌거나, 공격 지점이 끊기듯이 이동하는 현상을 해결했다.

## 게임플레이 큐 사용 & 나이아가라 VFX 제작
* 게임에서 공격이나 스킬등 파티클은 모두 제작하여 사용했다.
* 단순 원형 파티클 외에 사용할 특수한 형상은 블렌더나 포토샵으로 간단하게 제작하거나 머티리얼에서 프로시주얼로 제작하여 사용했다.

<div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
    <figure>
        <img src="/assets/img/260723-mesh1.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption>블렌더에서 제작한 매료빔 하트형태 메시</figcaption>
    </figure>
    <figure>
        <img src="/assets/img/260723-mesh2.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption>블렌더에서 제작한 할퀴기 반달형태 메시</figcaption>
    </figure>
    <figure>
        <img src="/assets/img/260723-texture1.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption>머티리얼로 제작한 삼각형 돌기 텍스처<</figcaption>
    </figure>
    <figure>
        <img src="/assets/img/260723-texture2.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption>복잡한 형상의 경우 외부 작업후 이미지로 임포트했다.</figcaption>
    </figure>
</div>

* 이를 나이아가라 시스템을 활용, 핵심적인 이미터와 보조적인 오라나 파티클을 합쳐 완성했다.
<div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
    <figure>
        <img src="/assets/img/260723-color.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption style="text-align: center;">피격모션이나 걸음등 기본 파티클은 흰색,<br>스킬이나 특징적인 부분은 플레이어 색으로 구별했다.</figcaption>
    </figure>
    <figure>
        <img src="/assets/img/260723-param.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption style="text-align: center;">필요에 따라 위치나 방향등 정보를 변수로 받아온다.<br>시스템에서 방향이나 위치를 추가적으로 계산해 이펙트에 활용한다.</figcaption>
    </figure>
</div>

* 제작된 이펙트들은 필요에따라`GameplayCue`를 통해 호출하거나, 시작과 종료에 대한 제어가 필요한 경우 `GameplayAbility`에서 조절하도록 연결했다.

```c++
//GA를 통해 매료빔 이펙트를 시작,종료하는 예시
// UGA_AttractiveBeam::ActivateAbility()
StartAttractiveBeamFX();
OnBeamStart(); // VFX 시작 훅

// UGA_AttractiveBeam::StartAttractiveBeamFX()
ActiveAttractiveBeamFX = UNiagaraFunctionLibrary::SpawnSystemAttached(
    AttractiveBeamFXAsset,
    AttachComponent,
    AttractiveBeamSocketName,
    FVector::ZeroVector,
    FRotator::ZeroRotator,
    EAttachLocation::SnapToTarget,
    false);

ActiveAttractiveBeamFX->SetVariableBool(FName(TEXT("User.BeamStarting")), true);
ActiveAttractiveBeamFX->SetVariableBool(FName(TEXT("User.BeamEnding")), false);
ActiveAttractiveBeamFX->SetVariableFloat(FName(TEXT("User.BeamFadeInTime")), BeamFXFadeInTime);
ActiveAttractiveBeamFX->SetVariableLinearColor(FName(TEXT("User.PlayerColor")), GetAvatarPlayerColor());

// UGA_AttractiveBeam::EndAbility()
StopAttractiveBeamFX();
OnBeamEnd(); // VFX 종료 훅

// UGA_AttractiveBeam::StopAttractiveBeamFX()
ActiveAttractiveBeamFX->SetVariableBool(FName(TEXT("User.BeamStarting")), false);
ActiveAttractiveBeamFX->SetVariableBool(FName(TEXT("User.BeamEnding")), true);
ActiveAttractiveBeamFX->SetVariableFloat(FName(TEXT("User.BeamFadeOutTime")), BeamFXFadeOutTime);

World->GetTimerManager().SetTimer(
    BeamFXCleanupTimerHandle,
    this,
    &UGA_AttractiveBeam::FinishAttractiveBeamFX,
    BeamFXFadeOutTime,
    false);

// UGA_AttractiveBeam::FinishAttractiveBeamFX()
ActiveAttractiveBeamFX->Deactivate();
ActiveAttractiveBeamFX->DestroyComponent();
ActiveAttractiveBeamFX = nullptr;
```

<div style="display: grid; grid-template-columns: repeat(1, 1fr); gap: 10px;">
    <figure>
        <img src="/assets/img/260723-gc.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption style="text-align: center;">시작과 종료를 코드에서 신호하지 않아도 되는 경우 GameplayCue에서 연결했다.</figcaption>
    </figure>
</div>

<div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
    <figure>
        <img src="/assets/img/260723-attracted.gif" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption style="text-align: center;">고양이 매료빔 및 행인 매료 이펙트</figcaption>
    </figure>
    <figure>
        <img src="/assets/img/260723-move.gif" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption style="text-align: center;">고양이 기본이동 이펙트</figcaption>
    </figure>
    <figure>
        <img src="/assets/img/260723-punch.gif" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption style="text-align: center;">고양이 할퀴기 이펙트</figcaption>
    </figure>
    <figure>
        <img src="/assets/img/260723-dash.gif" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption style="text-align: center;">고양이 대쉬 이펙트</figcaption>
    </figure>
    <figure>
        <img src="/assets/img/260723-cling.gif" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption style="text-align: center;">점박이 고양이 매달리기, 피격 이펙트</figcaption>
    </figure>
    <figure>
        <img src="/assets/img/260723-dominate.gif" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption style="text-align: center;">점박이 고양이 서열정리 이펙트</figcaption>
    </figure>
    <figure>
        <img src="/assets/img/260723-glare.gif" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption style="text-align: center;">고등어 고양이 째려보기 이펙트</figcaption>
    </figure>
    <figure>
        <img src="/assets/img/260723-heart.gif" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption style="text-align: center;">고등어 고양이 하트빔 이펙트</figcaption>
    </figure>
    <figure>
        <img src="/assets/img/260723-purr.gif" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption style="text-align: center;">뚱냥이 골골대기 이펙트</figcaption>
    </figure>
    <figure>
        <img src="/assets/img/260723-lay.gif" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption style="text-align: center;">뚱냥이 드러눕기 이펙트</figcaption>
    </figure>
</div>

## 텍스처 및 이미지 제작
* 게임에 사용될 아트 이미지나 일부 에셋들의 경우 AI를 활용해 생성했다.
<div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
    <figure>
        <img src="/assets/img/260723-main1.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption style="text-align: center;">메인화면 이미지의 경우 ai로 생성 후<br>포토샵으로 톤을 보정해 사용했다.</figcaption>
    </figure>
    <figure>
        <img src="/assets/img/260723-main2.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption style="text-align: center;">타이틀은 샘플을 생성 후 요소를 분리해 생성했다.<br> 배치의 자유도와 애니메이션의 용이성을 높여준다.</figcaption>
    </figure>
</div>

* 플레이어로 쓸 고양이 텍스처의 경우 실사 형태라 툰으로 변환하는 과정을 거쳤고, 삼색 고양이의 경우엔 직접 제작해야 했다. 블렌더에서 툰 스타일로 다시 그려 언리얼로 가져오는 과정을 거쳤다. AI를 이용해 배치는 그대로 두고 느낌만 바꾸어 새 텍스처 이미지를 생성하고 했으나, seam 부분에 연결 문제나 uv상 각 부위의 배치가 어긋나는등 완벽히 새로 생성하는데는 문제가 있어 직접 작업해야 했다.
<div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
    <figure>
        <img src="/assets/img/260723-cat1.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption style="text-align: center;">블렌더에서의 텍스처 페인팅으로<br>단순화된 색상과 패턴으로 그렸다.</figcaption>
    </figure>
    <figure>
        <img src="/assets/img/260723-cat2.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption style="text-align: center;">에셋의 기본 실사 텍스처와의 비교</figcaption>
    </figure>
</div>
