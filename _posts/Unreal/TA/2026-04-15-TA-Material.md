---
title: "TA: 머티리얼(Material)"
author: Jaeseong Kim
date: 2026-04-01 12:00:00 +0800
categories: [Unreal, TA]
tags: [Unreal, TA, Material]
---

## 머티리얼(Material)
* 물질을 나타내는 개념으로, 사물의 느낌, 질감등을 표현하는 정보를 저장한다. 
* 객체에 적용되어 표면에 색이나 모양, 특수한 시각적 효과를 나타내는데 사용될 수 있다.
* 단순히 많은 폴리곤으로 3D모델의 디테일을 나타낸다면 이는 프로그램이 많은 성능 자원을 소모하도록 만들것이다. 폴리곤의 절대적인 수를 줄이고 거기서 줄어든 디테일을 텍스처, 머티리얼 효과등으로 보충해 성능과 비주얼을 모두 챙기는 것이 게임에서의 머티리얼의 목적성이라 할 수 있겠다.
	* 또는 단순한 모델의 형상으로 표현하기 힘든 반사광, 투명도같은 시각적 효과의 사실적인 시뮬레이션, 물리 기반 렌더링(Physical Based Rendering, PBR)에선 머티리얼이 중요할 것이다. 이는 성능보단 시각적 표현이 중요한 애니메이션과 그래픽 제작에 있어서 더 부각되는 지점이다. 
* ### Blend Mode
	* 머티리얼 연산의 방향성/성향을 나타낸다. 보통 Opaque와 Translucent가 많이 쓰이고, 각각 투명한 재질과 불투명한 재질을 만들때 사용된다.
* ### 기초적인 값
	* 기본 색/발광 색(Base Color/)
		* 물체가 가지는 표면 혹은 RGBA에 해당하는 값을 숫자를 이용해 표현할 수 있으며, 각각 빨강, 초록, 파랑, 알파(투명도)에 해당한다. 알파값 없이도 RGB로 사용할 수도 있으며, HSV, HSL같은 다른 색 표현 방법도 있지만, 언리얼에서는 대게 RGB가 직관적이고 많이 사용된다.
	* Opque 설정
		* 거칠기(Roughness)
			* **표면의 거친 정도**를 나타낸다. 값이 낮으면 매끄러워 전반사 중심으로 일어나고, 높으면 거칠어져 난반사되고 반사광의 선명함이 줄어들고 표면에 퍼다.
			* 유니티 엔진에서는 반대개념인 부드러움(Smoothness)로 대신한다. 방향만 반대일 뿐 개념 자체는 동일하다.
		* 금속성(Metailc)
			* 얼마나 금속성을 띄는지, 즉 **얼마나 반사하는지**를 나타내는 값이다. 값이 높으면 금속처럼 빛을 반사하고, 값이 낮으면 비금속처럼 반사하지 않는 모습을 보여준다.
			* 반사하는 색상은 표면의 색, 알베도(Albedo)와 메탈릭 값의 곱을 반사하게 된다. 즉 표면 색상/텍스처에 영향을 받는다.
			* 난반사되어 퍼지는 색은 알베도(Albedo)와  메탈릭이 아닌 만큼(1-metalic)의 곱으로 결정된다. 즉 난반사도 표면 색상의 영향을 받는다.
		* 반사성(Specular)
			* 금속성과 마찬가지로 얼마나 반사를하는지를 나타내지만, 계산방식이 다르다.
			* 금속성과 유사하지만, 과거 랜더링 방식들처럼 PBR이 아닌 곳에서 더 중요한 개념으로, 반사에 관여하면서도 메탈릭과는 계산 방식이 다르다. 
			* 반사되고 디퓨즈되는 색상등 모두 표면의 색이 아닌 별도로 지정된 반사광 색상, 디퓨즈 색상에 따라 결정된다.
			* 언리얼 5에서는 PBR을 기본으로 삼기에 메탈릭을 사용할 일이 더 많다.
	* Translucent 설정
		* 투명도(Opacity)
			* 얼마나 투명한지, 즉 얼마나 뒤의 빛이 투과되는 결정한다. 
			* 0에 가까울수록 투명하고, 1에 가까울수록 선명하다.

* ### 머티리얼 인스턴스(Materital Instance, MI)
	* 머티리얼의 복잡한 연산과 그래프를 숨기고 사용할 수 있도록 인스턴스화한 에셋. 필요하다면 머티리얼쪽에서 노출시킬 변수들을 선택해 MI에서도 수정할 수 있다.
* ### 기초 노드
	* 머티리얼 그래프 안에서 효과를 만드는 구성요소로서 많은 기능들이 제공된다. 기초적이고 많이 쓰이는 노드들은 다음이 있다.
	* 값 노드
		* Constant, Vector2D, Vector3D, Vector4D까지가 값을 머티리얼 그래프에서 설정하는데 많이 쓰인다. 각각 qwer 키 위에 1,2,3,4,를 누른채로 빈공간을 클릭하면 추가할 수 있으며, 그래프에 쓰이는 값이나 색상들을 나타내는데 사용될 수 있다.
	* Lerp
		* 2개의 값을 받아 사이의 값으로 보정해준다. 알파 값이 0에 가까우면 A에, 1에 가까우면 B에 가깝게 보정된 수치를 출력한다.
		* 여러 값을 자연스럽게 더하거나 합칠때 많이 사용된다.
	* TextureSample
		* 이미지 텍스처를 불러와 사용할 수 있게 해준다.
		* 표면의 텍스처 뿐 아니라 Roughness, Opacity등 다른 영역에도 값의 분포도로서 사용할 수 있다.
	* TextureCoordinate
		* 텍스처 좌표에 따른 값을 계산한 분포를 제공한다.
		* 좌표에 따라 변화해야 하는 함수같은 구조를 설계할 때 유용하다.

<div  style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
	<figure>
		<img  src="/assets/img/260415-values.png"  alt=""  style="width:100%; aspect-ratio:4/3; object-fit:cover;">
		<figcaption style="text-align: center;">4종류의 값 노드들</figcaption>
	</figure>
	<figure>
		<img  src="/assets/img/260415-lerp.png"  alt=""  style="width:100%; aspect-ratio:4/3; object-fit:cover;">
		<figcaption style="text-align: center;">Lerp 노드</figcaption>
	</figure>
	<figure>
		<img  src="/assets/img/260415-texture.png"  alt=""  style="width:100%; aspect-ratio:4/3; object-fit:cover;">
		<figcaption style="text-align: center;">TextureSample노드</figcaption>
	</figure>
	<figure>
		<img  src="/assets/img/260415-coordinate.png"  alt=""  style="width:100%; aspect-ratio:4/3; object-fit:cover;">
		<figcaption style="text-align: center;">TextureCoordinate노드</figcaption>
	</figure>
</div>

* ## 간단한 예시
	* 간단하게 텍스처를 Panner를 이용해 회전시키고 Emissive Color를 이용해 붉은 빛으로 빛나는 느낌을 주는 불덩어리를 만들었다.

	<div  style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
		<figure>
			<img  src="/assets/img/260415-panner.png"  alt=""  style="width:100%; aspect-ratio:4/3; object-fit:cover;">
			<figcaption style="text-align: center;">Panner까지 적용한 그래프</figcaption>
		</figure>
		<figure>
			<img  src="/assets/img/260415-panner.gif"  alt=""  style="width:100%; aspect-ratio:4/3; object-fit:cover;">
			<figcaption style="text-align: center;">회전하는 화염 택스처</figcaption>
		</figure>
	</div>

	* BlendMode를 Translucent로 바꾸고, 프레넬 렌즈의 효과를 일으키는 Fresnel 노드를 Oneminus 노드로 뒤집어 Opaque에 넣어주면 보이는 윤곽선쪽/보이는 각도가 수직보단 사선/평행에 가까울수록 투명해지는, 중심만 뚜렷히 보이도록 할 수 있다. power노드를 사용해 제곱하여 소수점 값의 옅은 부분을 더 옅게 만들어줬다.
	<div  style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
		<figure>
			<img  src="/assets/img/260415-fresnel.png"  alt=""  style="width:100%; aspect-ratio:4/3; object-fit:cover;">
			<figcaption style="text-align: center;">Fresnel 까지 적용한 그래프</figcaption>
		</figure>
		<figure>
			<img  src="/assets/img/260415-fresnel.gif"  alt=""  style="width:100%; aspect-ratio:4/3; object-fit:cover;">
			<figcaption style="text-align: center;">흐려진 외곽 윤곽선</figcaption>
		</figure>
	</div>

	* 간단한 나이아가라 이펙트를 만들고 액터에 합쳐서 회전하는 불공을 만들었다. 보다 복잡하고 잘 설계된 나이아가라와 함께 넣으면 화려하고 사실적인 불공이 될것이다.
	<div  style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
		<figure>
			<img  src="/assets/img/260415-gbnl.png"  alt=""  style="width:100%; aspect-ratio:4/3; object-fit:cover;">
			<figcaption style="text-align: center;">레벨에 배치한 불공</figcaption>
		</figure>
		<figure>
			<img  src="/assets/img/260415-gbnl.gif"  alt=""  style="width:100%; aspect-ratio:4/3; object-fit:cover;">
			<figcaption style="text-align: center;">회전하는 불공</figcaption>
		</figure>
	</div>