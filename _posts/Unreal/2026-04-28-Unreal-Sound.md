---
title: "Unreal: 사운드 출력"
author: Jaeseong Kim
date: 2026-04-28 8:00:00 +0800
categories: [Unreal, Basics]
tags: [Unreal, Basics, SpawnSoundAtLocation, PlaySoundAtLocation]
---
## SoundBase/SoundCue
기본적으로 음성파일을 불러오면 uSoundBase로 들어오게 된다. SoundBase에선 기본적인 볼륨과 피치등을 음성과 관련된 설정들을 변경해 놓을 수 있으며, 이는 해당 에셋을 사용하는 모든 곳에서 적용되게 된다.
한편 사운드 큐에서는 크로스패이드, 랜덤같은 여러 기능들을 노드로 연결해, 제한적으로만 조정할 수 있었던 SoundBase를 노드의 일부로 활용함으로서 더 다채롭게 변화시키며 사용할 수 있다. 단순한 사운드 출력으로 충분하다면 굳이 SoundCue로 만들지 않고 SoundBase로 사용해도 되겠다.
## PlaySoundAtLocation()/SpawnSoundAtLocation()
주어진 SoundBase/SoundCue를 유저가 들을 수 있도록 재생하는 방법은 여러가지가 있다. 그중 PlaySoundAtLocation()/SpawnSoundAtLocation()는 월드의 특정 위치에 스폰한다. 이는 캐릭터가 움직이면 거리와 방향에 따라 소리가 다르게 들리도록 하여 공간감과 사실성을 더하는데 도움을 준다. 두 함수 모두 특정 위치에서 소리를 재생한다는 점은 같으나,PlaySoundAtLocation()는 반환값 없이, SpawnSoundAtLocation()는 UAudioComponent* 에 해당하는 반환값을 전달한다는 점에서 차이가 있다. 스폰 후에 실시간으로 피치나 재생여부등을 조절해야 한다면, 소리를 스폰한 후에도 접근할 방법을 남겨놔야 하기에 SpawnSoundAtLocation()를 이용해야 한다.

## PlaySound2D()/SpawnSound2D()
위와 비슷하지만 거리나 방향에 따라 달라지지 않는, 시스템 사운드처럼 직접 재생한다는 점에서 차이가 있다. 배경음악, UI 사운드 같은데 적합할 수 있겠다.

## PlaySoundAttached()/SpawnSoundAttached()
PlaySoundAtLocation()/SpawnSoundAtLocation()처럼 월드의 특정 위치에 사운드를 재생한다는 점은 비슷한데, 다만 액터에 부착된듯 소리가 따라다니도록 만든다. 한번 재생을 시작하면 부착된 액터를 따라다니며 소리를 재생한다. 액터 대신 컴포넌트에 대한 부착도 가능하다.

## UAudioComponent
SpawnSound계열의 함수에서 반환값으로 사용되는 소리 재생에 사용되는 컴포넌트이다. 단순 선언 후에 SpawnSound계열 함수의 값을 받는 자료형처럼 사용해도 되지만, 지속적으로 재생되어야 하거나 재생중 재생여부, 볼륨등 조작할 일이 많다면, 다른 여타 컴포넌트처럼 직접 액터에 장착해 사용하는게 더 효율적일 수도 있다. 이 경우엔 에디터 상에서 특정 사운드를 컴포넌트에 장착해놓을 수 있으며, SoundBase/SoundCue를 별도로 선언해 지정해놓지 않고도 장착한 소리를 재생하는 형태로 대신할 수 있다.