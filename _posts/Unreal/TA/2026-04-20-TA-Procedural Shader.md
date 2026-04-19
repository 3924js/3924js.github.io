---
title: "TA: 프로시주얼 쉐이더(Procedural Shader)"
author: Jaeseong Kim
date: 2026-04-20 12:00:00 +0800
categories: [Unreal, TA]
tags: [Unreal, TA, Material ,Procedural Shader]
---
## 프로시주얼 쉐이더(Procedural Shader)
* 머티리얼이나 텍스처를 이미지에 기반하여 사용하는게 아닌, 수학/알고리즘에 기반한 절차(procedure)에 따라 만들어내는 방법을 말한다.
	* 생성과정을 제어함으로서 원하는 값에 따라 다양한 결과물을 만들어 낼 수 있다.
	* 노이즈, 반복패턴, 값에 따라 바뀌는 효과등 변화를 제어해야 하는 상황에 도움이 될 수 있다.
* 같은 이미지나 패턴을 만들기 위해 이미지를 저장후 불러와 쓸수도 있고, 프로시주얼 쉐이더로 제작하여 사용할 수도 있다. 전자의 경우 메모리에 저장하는 것이기에 드로우콜, 즉 메모리 통신의 비중이 더 늘어나고, 후자의경우 즉석에서 과정으로 만들어지기에 연산 부하가 더 증가한다고 볼 수 있다. 어느쪽이 더 좋다라기보단 장단점의 차이라 할 수 있겠다.
	* 만든 프로시주얼 쉐이더의 결과물을 텍스처 파일로 저장하여 사용할 수도 있다.

## 예시 1: 직물패턴
패션 상품에서 많이 보이는 반복적인 직물 패턴을 만들어보자.
TextureCoordinate노드에 Mask노드에 연결한다. U 혹은 V값만 남긴 후 Modulo를 이용해 나누는 값에 따라 나머지로 남는 값에 의해 따라 타일처럼 분할된다. Step으로 일정 값 이하는 0으로, 더 큰 값은 1로 대체하면 뚜렷한 줄무늬 모양을 만들 수 있다.
![image](/assets/img/260420-fabric1.png)
둘을 Subtract로 빼주면 정사각형의 배치를 얻을 수 있다.
![image](/assets/img/260420-fabric2.png)
한편 Modulo로 얻은 스트라이프 패턴에서 Step전에 Add로 합치면 대각선으로 점점 짙어지는 TextureCoordinate와 유사하지만 그리드로 배열된 패턴을 얻을 수 있다.
![image](/assets/img/260420-fabric3.png)
이를 아까처럼 Step에 적용하면 모서리에만 있는 삼각형을 얻을 수 있다.
![image](/assets/img/260420-fabric4.png)
비슷하게 응용하여 더 큰 삼각형에서 작은 삼각형을 Subtract로 뺴주면 대각선의 직선을 얻는다.
![image](/assets/img/260420-fabric5.png)
이를 모두 합치면 코드나 가방에서 많이 볼법한 직물 패턴인 하운드투스가 만들어진다. Roughness와 EmissiveColor를 약간 조정해 직물의 거친 느낌을 조금 주어보았다. 처음의 TextureCoordinate에서 UTiling/VTiling을 조절하면 패턴의 밀도를 조절할 수 있다.
<div  style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
	<figure>
		<img  src="/assets/img/260420-fabric6.png"  alt=""  style="width:100%; aspect-ratio:4/3; object-fit:cover;">
		<figcaption style="text-align: center;">머티리얼에 최종적으로 연결된 모습 </figcaption>
	</figure>
	<figure>
		<img  src="/assets/img/260420-fabric7.png"  alt=""  style="width:100%; aspect-ratio:4/3; object-fit:cover;">
		<figcaption style="text-align: center;">상자에 적용된 직물패턴</figcaption>
	</figure>
</div>

## 예시 2: 이벤트 구역
이번엔 원형의 이벤트 구역에서 존재를 알리는 위로 솟는 효과를 만들어보자.
이전 방법과 유사하게 Mask와 Modulo로 스트라이프 패턴을 만든다. TextureCoordinate에서 Panner와 연결하고 Speed에 값을 넣으면 UV값을 실시간으로 변화시켜 프로시주얼 쉐이더에서도 움직이는 모습을 만들어낼 수 있다.
![image](/assets/img/260420-Eventzone1.gif)
Step을 이용해 스트라이프 패턴을 뚜렷하게 만든다. 한편 mask로 G값만 받은 후 Multiply로 합치면 위로 갈수록 흐려지는 효과를 연출할 수 있다. Power를 이용해 값을 제곱하면 1이하의 값인 UV의 값들이 더 작아지면서 이 흐려지는 연출을 더 확실히 적용한다.
![image](/assets/img/260420-Eventzone2.gif)
multiply를 사용하면 위쪽에 흐려지기 전까지도 더 뚜렷한 형상을 유지할 수 있다. Blend mode는 Translucent로, 지금 만든 것을 Opacity에 연결하고 Color는 목표지점을 나타내는 초록색을 넣어줬다. 투명한 이펙트가 적용된 면의 안쪽/뒷면에서도 보이게 하고 싶다면 Two Sided를 체크해줘야 한다. 지금 상태로 이대로 원기둥에 적용하면 위아래도 머티리얼이 적용되는 모습을 볼 수 있다.
<div  style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
	<figure>
		<img  src="/assets/img/260420-Eventzone3.gif"  alt=""  style="width:100%; aspect-ratio:4/3; object-fit:cover;">
		<figcaption style="text-align: center;">머티리얼에 최종적으로 연결된 모습</figcaption>
	</figure>
	<figure>
		<img  src="/assets/img/260420-Eventzone4.gif"  alt=""  style="width:100%; aspect-ratio:4/3; object-fit:cover;">
		<figcaption style="text-align: center;">위아래도 출력되는 머티리얼</figcaption>
	</figure>
</div>
해결법은 위아래 면을 삭제하는 것이다. 블렌더에서 실린더를 추가하고 위아래 면을 삭제한 후 fbx로 내보내 언리얼로 불러온다. 이제 머티리얼을 적용후 배치해보면 들어가야할 것 같은 원형의 이벤트 지점이 만들어졌다.
<div  style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
	<figure>
		<img  src="/assets/img/260420-Eventzone5.png"  alt=""  style="width:100%; aspect-ratio:4/3; object-fit:cover;">
		<figcaption style="text-align: center;">블렌더에서 편집된 Cylinder</figcaption>
	</figure>
	<figure>
		<img  src="/assets/img/260420-Eventzone6.gif"  alt=""  style="width:100%; aspect-ratio:4/3; object-fit:cover;">
		<figcaption style="text-align: center;">완성된 Eventzone</figcaption>
	</figure>
</div>