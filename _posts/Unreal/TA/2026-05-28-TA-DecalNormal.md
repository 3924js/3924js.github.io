---
title: "TA: 데칼 입체적으로 만들기 - 알파에서 노멀맵으로"
author: Jaeseong Kim
date: 2026-05-28 02:00:00 +0800
categories: [Unreal, TA]
tags: [Unreal, TA, Volumetric Fog]
---
피가 튄 데칼 자체는 꽤나 많이 찾아볼 수 있는데, 데칼로 쓸 수 있는 것들 중에는 입체감까지 있는게 많이는 없는 것 같다. 노멀맵이 머티리얼로 같이 쓰이도록 주어지는 경우도 있긴 하지만 그렇게까지 만족스럽지는 않다. 일반 액터에 쓰이는 머티리얼이라면 World Position Offset으로 입체감을 살리는 방법도 있겠으나, 데칼에서는 적용되지 않는 방법이다.
<div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
  <figure>
    <img src="/assets/img/260528-basic1.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>기본으로 적용하려고 만든 데칼 머티리얼</figcaption>
  </figure>
  <figure>
    <img src="/assets/img/260528-basic2.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>핏자국이 패턴과 무관하게 밋밋하고 꽤 균일하게 반사한다.</figcaption>
  </figure>
</div>

## 알파로 노멀맵 만들기
핏자국, 얼룩같이 물체의 덧씌워지는 데칼이라면 그라데이션처럼 천천히 변하고, 또 부분마다 투명도가 많이 다른 모습을 볼 수 있을 것이다. 이것 자체를 노멀맵으로 사용해서 입체감을 살려보자. 평평한 바닥에 피나 점성있는 액체가 쏟아진다면 단면을 보았을 때 가운데 쪽은 평평할 것이고, 경계면쪽은 가파를 것이다.
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
    <figcaption>전보다 반사광과 패턴 자체가 차분해진 느낌을 준다.</figcaption>
  </figure>
</div>