---
title: "Unreal: 가짜 안개(Fake Volumetric Fog)"
author: Jaeseong Kim
date: 2026-05-26 02:00:00 +0800
categories: [Unreal, TA]
tags: [Unreal, TA, Volumetric Fog]
---
탑뷰 시점의 게임을 만들었을 때, 손전등을 이용해 비춘 모습을 위에서 보면 손전등이 비친 부분만 보이게 된다. 사실적인 비주얼인건 맞지만, 경로가 보이지 않기에 수직인 벽에 빛만 비춘다면 수직방향 기준으로 반사될 빛이 없기에 꽤나 어둡게 보이고 아무것도 인지할 수가 없다. 그래서 방향이라도 인식하게 하고 자연스럽게 만들고자 먼지나 안개낀듯한 산란효과를 주고자 했다.
* ## Volumetric Fog를 이용한 시도
언리얼 엔진에선 기본적인 안개 액터를 제공한다. Exponential Height Fog가 그것인데, 높이에 따라 아래로 갈수록 점점 짙은 안개를 적용해 뿌옇게 만들 수 있다. 물론 높이에 따른 안개의 밀도, 안개의 시작 높이나 색상등 다양한 옵션을 적용할 수 있다. 다만 이 자체로는 충분한 산란 효과를 만들어낼 수 없다. 단순한 후처리에 가까운 효과이기에 그냥 뿌옇게 보이는 효과를 가벼운 성능으로 구현할 수 있다.
다만 공중에서 빛이 산란하는 효과를 위해선 안개를 공간에 대해 계산하는 Volumetric Fog를 체크해 사용해야 한다. Volumetric부터는 가상의 단위공간에 픽셀에 대해 빛의 산란을 계산할 수 있게된다. Volumetric Fog에도 빛의 산란과 관련된 요소들이 꽤나 있지만, 어두운 환경에서 손전등만 사용하는 제한적인 상황에선 Volumetric Fog를 활성화만 하고, 
* ## 다른 시도: 가짜 안개만들기
위같은 
* ## 원뿔 길이 조절하기

* ## 원뿔 머티리얼 만들기

* ## 결과물


	![image](/assets/img/260511-graph1.png)

<div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
  <figure>
    <img src="/assets/img/260511-instance1.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>보스급 적에 쓰기 위해 만든 MI 세팅</figcaption>
  </figure>
  <figure>
    <img src="/assets/img/260511-instance2.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>특수한 적에게 쓰기 위해 만든 MI 세팅</figcaption>
  </figure>
</div>
![image](/assets/img/260511-preview.png)
![image](/assets/img/260511-preview.gif)