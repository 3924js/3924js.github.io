---
title: "Unreal: 트레이스(Trace)"
author: Jaeseong Kim
date: 2026-04-24 8:00:00 +0800
categories: [Unreal, Basics]
tags: [Unreal, Basics, Line Trace, Async Trace]
---
3D 게임의 많은 로직은 시야를 중심으로 한다. 무엇을 가리키는지, 방향에 무엇과 충돌하는지를 알아야 어떤 사물을 보고있고, 쏜 탄환이 어디 맞았는지, 어떤각도로 맞았는지등 게임에 필요한 정보로 변환하고 사용할 수 있다. 그 기본적인 개념인 트레이스(Trace)를 연습해보자.

## 콜리전(Collision)
트레이스를 접하기 전에 알아야 할 개념이 있다면 콜리전이다. 트레이스를 진행하는데 있어 충돌 검사를 할 수 있는 여러 채널들이 존재한다. 이런 채널들을 임의로도 추가해줄 수 있다. Edit-Project Settings-Engine-Collision으로 들어가면, Preset에 여러 기본제공 콜리전 프레셋이 존재한다. 먼저 테스트를 위해 새로운 채널을 추가해주자. Object Channels의 New Object Channel을 누르고 이름을 NewChannel이라 지어주겠다. Default Response는 Block 그대로 사용해보자.
![image](/assets/img/260424-channel.png)

방금 추가한 것 같은 채널들에 대해 언리얼에서 기본적으로 각 채널에 대해 어떻게 처리할건지 정의해둔 프리셋(Preset)이 있다. 보편적인 필요에 따라서 모두 막을건지, 모두 통과시킬건지, 무시할건지등 다양한 세팅이 있다. 새로운 채널인 NewChannel을 추가해줬을 때, Default Response에 의해 기본 프리셋들도 NewChannel에 대해서는 Block으로 처리할것이다. 채널을 추가했을 때처럼 필요에 따라서는 임의의 프리셋들을 별도로 추가/정의해줄 수 있다. New를 누르고 Visibility와 Camera를 Ignore로 설정하고 이름은 NewChannelPreset으로 해줬다.
![image](/assets/img/260424-preset.png)
## LineTraceSingle()
이제 본격적인 트레이스를 알아보자. 쉽게 결과부터 보자면 시작지점에서 끝지점까지 선을 긋는데, 가려지는등 끊기는 지점이 있으면 그 지점에서의 충돌 정보를 기록한다. 시험삼아 선을 쏴서 충돌하는 처음만 체크하는 LineTraceSingle을 기본으로 알아보자.
UWorld에서 호출하는 UWorld::LineTraceSingleByChannel() 가 있고, 또 KismetSystemLibrary를 통해 호출하는 UKismetSystemLibrary::LineTraceSingle() 가 있다. 방식은 유사하고 성능도 유사하지만 KismetSystemLibrary가 디버깅 용도로는 더 적합하다. 간단한 설정 변경으로 트레이스가 어떻게 진행되는지 확인할 수 있고, 여러 정보가 확인 가능한 방면, UWorld에서 호출하는 트레이스는 그렇지 못하다. 개발이 끝나고 패키징 전에 UWorld 방식으로 교체하면 개발과정과 결과물의 성능을 모두 챙길 수는 있지만 하나하나 바꾸는건 꽤나 오래 걸리는 일일 수 있다.
트레이스는 몇가지 인자를 받는다. 선언을 하고 차근차근 살펴보자.

## AsyncLineTraceByChannel()
방금 본 모든 동작은 하나의 스레드에서 작동한다. 이는 요즘 보편적인 멀티코어/멀티스레드 환경을 제대로 활용하지 못할 뿐만 아니라, 수많은 트레이스를 날려야 하는 게임 환경에는 아쉬운 동작일 수 있겠다. 다행히도 언리얼은 그런 트레이스 검사를 멀티스레딩으로 할 수 있도록 함수를 제공한다.
