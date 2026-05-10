---
title: "Unreal: 마스터 머티리얼 만들기(Master Material)"
author: Jaeseong Kim
date: 2026-05-11 02:00:00 +0800
categories: [Unreal, TA]
tags: [Unreal, TA, Master Material, Material Instance]
---
* 마스터 머티리얼(Master Material)
	* 머티리얼을 만들다 보면 동일한 패턴으로 색상, 속도, 텍스처등 일부만 바꾼채 노드/그래프의 구성을 거의 동일하게 가져갈 때가 많이 있다. 이럴 때 해당 구성을 템플릿/함수처럼 미리 만들어놓고 재사용 할 수 있도록 제작해놓는 표준이되는 머티리얼을 마스터 머티리얼(Master Material)이라고 한다.
	* 언리얼의 머티리얼 시스템에서 강력하게 드러나는 요소중 하나로, 유니티나 다른 머티리얼 시스템에선 비교적 적용이 어려울 수 있는 개념이다.
* 마스터 머티리얼 제작
	* 제작과정 자체는 머티리얼의 제작과 동일하다. 차이점이 있다면 중요한 값이나 노드를 후에도 조절할 수 있게 매개변수로 승격시켜놓는 것이다.
	* 안개와 유사하게 만들어 머티리얼 구체에 사용하기 위해 구성한 전체 그래프는 다음과 같다.
	![image](/assets/img/260511-graph1.png)

		* 먼저 앞을 살펴보노라면 AbsoluteWolrdPosition을 이용해 절대좌표를 불러와 UVCoordinate 대신 사용한다. Mask를 이용해 Z축을 포함한 평면을 Panner에 넘겨줘 흔들리는 효과를 부여한다. Sine의 주기는 미리 각각 3,5로 정해놓았다. 분리된 XZ좌표값의 이동은 뒤에서 Y값과 다시 Append노드를 통해 3차원 벡터로 합쳐져 Noise에 연결되어 움직이는 노이즈 패턴을 생성한다.
		![image](/assets/img/260511-graph2.png)

			* 뒤에 이어질 노이즈 노드에서 TextureCoordinate는 만들경우 경계가 맞닿는 부분에서, 특히 Sphere같은 형태에서 경계면/심(Seam)이 뚜렷하게 보이는 현상을 보여 안개같이 전방위로 보이는덴 적합하지 않은 형태가 나온다.
			![image](/assets/img/260511-seam.png)

		* 노이즈패턴은 변화하는 속도에 따라 위아래를 나누어 위쪽은 색상을 입힌 후 BaseColor와 EmissiveColor에, 하단부는 Opacity에 넣어줬다. 위쪽과 아래쪽에서 Sine 주기가 다르기에 서로 어긋나게 움직이며 더 일렁이는 안개/불규칙성을 더한다.
		![image](/assets/img/260511-graph3.png)

		* 마스터 머티리얼로 활용하기 위해 적용될 2가지 색상과 수직/수평 이동속도까지 총 4개의 매개변수를 노출시켜놓았다.
		![image](/assets/img/260511-graph4.png)
* 머티리얼 인스턴스(Material Instance)
	* 먼저 만든 마스터 머티리얼을 상속받는 머티리얼을 만들면 먼저 노출시켜놓은 매개변수들을 수정해 간단하게 비슷한 느낌의 여러 다른 머티리얼을 만들 수 있다. 흰색과 푸른색을 베이스로 하는 빠르게 변하는 안개, 노란색과 주황색을 베이스로 하는 비교적 정적인 안개도 단순히 매개변수를 달리하는 인스턴스를 여러개 만들어서 생성해낼 수 있다.
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