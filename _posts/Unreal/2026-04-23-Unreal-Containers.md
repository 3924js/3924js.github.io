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
* 클래스의 유형 정보를 담기 위해 사용하는 탬플릿 클래스로, UClass*와 동일한 기능을 제공한다.
* 하드 레퍼런스를 제공한다.

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

## TArray<>

## TMap<>

## TSet<>

## 간단한 인벤토리 예시