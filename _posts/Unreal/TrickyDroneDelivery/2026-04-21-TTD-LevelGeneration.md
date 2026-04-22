---
title: "TrickyDroneDelivery: 랜덤 레벨 생성-GameMode, GameState"
author: Jaeseong Kim
date: 2026-04-22 8:00:00 +0800
categories: [Unreal, Project]
tags: [Unreal, Project, GameMode, GameState]
---
이전엔 레벨마다 스폰될 적들을 만들었다. 이번엔 게임의 규칙과 레벨의 생성을 구현해보자.
## 배달 지점 구현
구현에 앞서 먼저 배달 지점을 만들어보자. 택배상자가 충돌하면 

## 랜덤 건물 배치
건물은 당장은 플로우 검증 정도만 하고 싶어서 다양한 건물의 배치보단 간단한 건물배치를 이용한 맵을 구현해보도록 하겠다. 간단하게 건물들을

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