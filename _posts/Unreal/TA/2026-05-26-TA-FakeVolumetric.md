---
title: "Unreal: 가짜 안개(Fake Volumetric Fog)"
author: Jaeseong Kim
date: 2026-05-26 02:00:00 +0800
categories: [Unreal, TA]
tags: [Unreal, TA, Volumetric Fog]
---
탑뷰 시점의 게임을 만들었을 때, 손전등을 이용해 비춘 모습을 위에서 보면 손전등이 비친 부분만 보이게 된다. 사실적인 비주얼인건 맞지만, 경로가 보이지 않기에 수직인 벽에 빛만 비춘다면 수직방향 기준으로 반사될 빛이 없기에 꽤나 어둡게 보이고 아무것도 인지할 수가 없다. 그래서 방향이라도 인식하게 하고 자연스럽게 만들고자 먼지나 안개낀듯한 산란효과를 주고자 했다.
* ## Volumetric Fog를 이용한 시도
* ## 다른 시도: 가짜 안개만들기
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