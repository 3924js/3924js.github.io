---
title: "TA: 나이아가라 시스템(Niagara System)"
author: Jaeseong Kim
date: 2026-05-28 02:00:00 +0800
categories: [Unreal, TA]
tags: [Unreal, TA, Niagara]
---
언리얼에서 기존에 제공되던 캐스케이드(Cascade) 시스템을 잇는 다음세대 VFX시스템이 나이아가라(Niagara)다. 폭포라는 뜻에 이어 상징성 큰 폭포인 나이아가라를 따와서 지은 차세대 시스템 답게 더 많은 기능, 복잡한 이펙트를 제공한다.
* 파라미터를 나이아가라 내부에서 선언하고 블루프린트를 통해 손쉽게 전달할 수 있다.
* 스켈레탈 메쉬같은 게임 데이터에 직접적인 접근이 가능하다.
* 파티클의 충돌같은 이벤트에 따른 로직을 생성할 수 있다.
* GPU연산으로 다량의 파티클을 다루는데 더 원활하다.
레거시 프로젝트를 다루는게 아니라면, 캐스캐이드는 이제 거의 쓸일이 없고 나이아가라를 주로 쓰게된다. 사용법 자체가 어렵다기 보단 주어진 기능을 어떻게 다양하게 활용할 수 있는지에 대한 경험과 감각이 더 많은 영향을 미치는 듯 하다.

## 이미터(Emitter)
파티클을 발생시키는 단위를 이미터라고 부른다. 이미터 안에서는 어떤 파티클을 만들고 얼마나 많이, 어떤색으로, 어떤방향으로, 얼마동안 발생시킬지등 다양한 조건과 상태를 설정할 수 있다. 핵심적으로 많이 쓰는 부분들의 구조를 살펴보노라면 다음과 같다.
* Emitter Spawn - 이미터가 스폰되는 시점에 CPU에서 정해질 요소들이 배치된다.
* Emitter Update - 지속적으로 이미터에서 변화시켜야 하는 요소들이 배치된다.
* Particle Spawn - 이미터가 발생시킬 파티클이 소환될 때 정해지는 요소들이 배치된다. 즉 파티클이 스폰될 때 1번씩 호출된다.
* Particle Update - 파티클이 소환된 이후 변화되는 요소들이 배치된다. 소환 이후에 지속적으로 적용되는 요소들인 중력, 바람, 색상, 노이즈등이 배치될 수 있다.
* Render - 파티킬이 어떻게 보일지를 결정한다. 스프라이트, 리본등 어떤 파티클을 소환할지와 어떤 머티리얼을 사용할지등을 정한다. 


![image](/assets/img/260528-reflection.png)
즉 경사의 변화율을 알면 적절한 노멀맵을 만들어 입체감을 줄 수 있을 것인데, 그에 딱 알맞는 노드가 있다. DDX 와 DDY라는 노드는 인접한 픽셀과의 값 차이를 계산해 반환해준다. 즉 지금 상황에서 투명도를 넣는다면, 점점 짙어지는 경계면에서 빠르게 증가한다면 높은 증가율, 높은 값을 반환할 것이고, 거의 변화가 없는 일정한 투명도라면 평평한 면처럼 작은 값을 반환해 줄 것이다. 이를 3차원 벡터로 만들어 노멀에 연결해주면 전보단 입체적인 모습을 볼 수 있을 것이다.
<div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
  <figure>
    <img src="/assets/img/260528-normal1.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>알파값에서 ddx, ddy를 이용해 노멀을 만들어 연결했다.</figcaption>
  </figure>
  <figure>
    <img src="/assets/img/260528-normal2.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>전보단 울퉁불퉁하게 패턴과 일치하는 반사를 보여준다.</figcaption>
  </figure>
</div>

다만 지금은 피를 흘린 패턴이기에 이대로 쓰기엔 너무 묽게 튀긴듯한,점성없어 보이는 상태일 수 있다. Roughness를 올려 광을 조금 죽이고, 노멀의 값에도 배율을 달아 조금 줄여주면 스프레이처럼 산발한 방울들보단 좀더 엉겨붙은 듯한 질감을 보일 것이다.
<div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
  <figure>
    <img src="/assets/img/260528-normal3.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>Roughness와 DDX/DDY 결과물에 Multiply노드를 연결했다.</figcaption>
  </figure>
  <figure>
    <img src="/assets/img/260528-normal4.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>전보다 반사광이 덜하고 패턴 자체가 차분해진 느낌을 준다.</figcaption>
  </figure>
</div>