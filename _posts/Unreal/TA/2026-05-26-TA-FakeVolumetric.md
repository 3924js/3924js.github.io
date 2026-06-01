---
title: "Unreal: 가짜 안개(Fake Volumetric Fog)"
author: Jaeseong Kim
date: 2026-05-26 02:00:00 +0800
categories: [Unreal, TA]
tags: [Unreal, TA, Volumetric Fog]
---

![image](/assets/img/260526-basic.png)

탑뷰 시점의 게임을 만들었을 때, 손전등을 이용해 비춘 모습을 위에서 보면 손전등이 비친 부분만 보이게 된다. 사실적인 비주얼인건 맞지만, 경로가 보이지 않기에 수직인 벽에 빛만 비춘다면 수직방향 기준으로 반사될 빛이 없기에 꽤나 어둡게 보이고 아무것도 인지할 수가 없다. 그래서 방향이라도 인식하게 하고 자연스럽게 만들고자 먼지나 안개낀듯한 산란효과를 주고자 했다.
* ## Volumetric Fog를 이용한 시도
언리얼 엔진에선 기본적인 안개 액터를 제공한다. Exponential Height Fog가 그것인데, 높이에 따라 아래로 갈수록 점점 짙은 안개를 적용해 뿌옇게 만들 수 있다. 물론 높이에 따른 안개의 밀도, 안개의 시작 높이나 색상등 다양한 옵션을 적용할 수 있다. 다만 이 자체로는 충분한 산란 효과를 만들어낼 수 없다. 단순한 후처리에 가까운 효과이기에 그냥 뿌옇게 보이는 효과를 가벼운 성능으로 구현할 수 있다.
다만 공중에서 빛이 산란하는 효과를 위해선 안개를 공간에 대해 계산하는 Volumetric Fog를 체크해 사용해야 한다. Volumetric부터는 가상의 단위공간에 픽셀에 대해 빛의 산란을 계산할 수 있게된다. Volumetric Fog에도 빛의 산란과 관련된 요소들이 꽤나 있지만, 어두운 환경에서 손전등만 사용하는 제한적인 상황에선 Volumetric Fog를 활성화만 하고, 광원에서 Volumetric Scattering Intensity를 조정하면 안개에 대해서 산란을 얼마나 강하게 적용할 것인지 설정할 수 있다. 

<div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
  <figure>
    <img src="/assets/img/260511-instance1.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>Volumetric Fog 활성화</figcaption>
  </figure>
  <figure>
    <img src="/assets/img/260526-scattering.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>광원에 있는 Volumetric Scatterting Intesity<br>기본값은 1로 잡혀있다.</figcaption>
  </figure>
  <figure>
    <img src="/assets/img/260526-scattering5.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>Volumetric Scatterting Intesity = 5<br>어느정도의 산란이 보이기 시작한다.</figcaption>
  </figure>
  <figure>
    <img src="/assets/img/260526-scattering20.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>Volumetric Scatterting Intesity = 20<br>산란 효과와 함께 계단현상 또한 뚜렷해진다.</figcaption>
  </figure>
</div>

저 특유의 계단현상만 무시할 수 있다면 좋은 방법일 것이다. 다만 이는 픽셀 단위로 공간을 나눠 안개를 계산하는 Volumentric Fog의 특성상 한계의 영역이다. 콘솔 커맨드나 엔진 설정 수정으로 VolumetricFog.GridPixelSize 픽셀의 단위를 더 촘촘하게 조정할 수도 있는데, 기본값인 8에서 2정도까지 조정하면 안개를 계산하는 픽셀 사이즈가 촘촘해지면서 계단현상은 사라진다, 하지만 연산량이 늘어남에 따라 잔상이 남는 고스팅 현상이 일어나고, 이를 해결하기 위해서 VolumetricFog.GridSizeZ(수직 방향에 대한 픽셀 사이즈), VolumetricFog.Jitter(안개의 픽셀 위치를 얼마나  흔들지), VolumetricFog.TemporalReprojection(지터링을 사용할지 설정) VolumetricFog.HistoryWeight(과거의 계산 결과를 얼마나 섞어 넣을것인지, 많이 섞일수록 잔상은 많이 남지만 그만큼 부드럽다.) 같은 여러 설정들이 있는데, 바꾸어도 하드웨어 스펙이 받춰주지 않는 이상 계단현상, 고스팅, 일렁거리는듯한 노이즈중 어떠한 문제는 계속 남게된다.

* ## 다른 시도: 가짜 안개만들기
위같은 상태로는 단점이 뚜렷한 나머지 쓸수가 없다. 대신 다른 방법을 써보자. 어차피 spotlight가 원뿔 형태로 빛을 발사하니, 그 영역에 메쉬를 두고 비슷한 발광효과를 주면 되지 않을까? 현실적이자 꽤나 효율적인 접근이다. 빛의 산란을 계산하는 것보단 훨씬 비용도 적기에 게임에서 충분히 쓰일 법한 대체제중 하나일 것이다. 
해당 영역을 표현해줄 원뿔은 간단하게 블렌더에 있는 기본 메쉬를 가져왔다. ShadeSmooth라는 자동으로 Normal을 정리해 면이 살아있는 메쉬가 아닌 부드러운 곡면의 메쉬처럼 보여주는 기능이 있는데, 언리얼로 메쉬를 불러들이면 적용이 잘 안되는지 깨져서, 이번엔 일단 ShadeSmooth 없이 면 갯수만 128로 늘려서 충분히 부드럽게 느끼도록 만들고 가져왔다. 빛을 쏘는 원뿔의 각도를 언리얼 안에서 조정하는 기능도 있기에, 스케일로 편하게 계산하려면 45도, 그러니까 높이와 반지름이 1대1이 되도록 조정했다.

![image](/assets/img/260526-cone.png)

원뿔에 입힐 머티리얼을 만들어보자. 반투명의 발광체로 만들어야 하니 Translucent에 Unlit으로 만들어줬다. 스스로 빛을 내기만 하면 되니 굳이 Default Lit일 필요는 없겠다.
<div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
  <figure>
    <img src="/assets/img/260526-mat1.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>Emissive Color엔 광원의 색을 그대로 넣어줬다.</figcaption>
  </figure>
  <figure>
    <img src="/assets/img/260526-mat2.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>DepthFade노드로 기본적인 Opacity를 조절하며<br>다른 메쉬와 부딪힐 부분을 부드럽게 조정한다.</figcaption>
  </figure>
  <figure>
    <img src="/assets/img/260526-mat3.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>Fresnel대신 사용할 벡터를 이용한 외곽 페이드<br>벡터 연산으로 픽셀이 빛의 축과 얼마나 겹치는지 계산한다.</figcaption>
  </figure>
  <figure>
    <img src="/assets/img/260526-mat4.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>수직 축에 따른 페이드 효과와<br>빛의 거리에 따른 페이드 효과</figcaption>
  </figure>
</div>

![image](/assets/img/260526-mat5.png)

적용하면 대략적으로 이런 모습이다. 

처음엔 Fresnel노드에 OneMinus를 이용해 외부에 대한 페이드를 구현해봤는데, 결과물 자체는 내가 원하는 방향성이였지만, 약간 사선에서 보는 카메라 특성상 원뿔이 가리키는 방향에 따라 표면이 카메라와 이루는 각도가 달라지면서, 방향에 따른 방향의 변화가 너무 뚜렷하게 나타났다. 백터를 이용해 카메라와 손전등의 방향이 이루는 평면과의 각도를 직접 계산하는 지금 방식에선 해결되는 문제이다.
<div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
  <figure>
    <img src="/assets/img/260526-fresnel1.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>Fresnel노드 사용시 위로 비출때<br>영역이 뚜렷하게 보인다.</figcaption>
  </figure>
  <figure>
    <img src="/assets/img/260526-fresnel2.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>픽셀 방향이 카메라 기준 수직에 가까워 뚜렷히 보인다.</figcaption>
  </figure>
  <figure>
    <img src="/assets/img/260526-fresnel3.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>Fresnel노드 사용시 아래로 비출때<br>영역이 희미하게 보인다.</figcaption>
  </figure>
  <figure>
    <img src="/assets/img/260526-fresnel4.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>픽셀 방향이 카메라의 방향과 벌어짐에 따라 연해진다.</figcaption>
  </figure>
</div>

![image](/assets/img/260526-penetration.png)

한가지 문제가 있다면 벽을 메쉬가 뚫고 들어가면 빛난다는 것이다. 모든 방향에 대해 동적으로 계산해 빛의 산란을 보이는게 아닌, 그냥 하나의 통짜 메쉬로 퉁치는 방식 특유의 문제라 할 수도 있겠다.

이는 거리 전방 혹은 여러 방향으로의 거리를 계산해 원뿔의 크기를 동적으로 조절하면 어느 정도는 더 낫게 보일 수 있다. 벽이 있으면 벽을 뚫지 않을 만큼 메쉬의 크기 자체를 줄이는 것이다. 위에 머티리얼에서 빛기둥이 사이드로 갈수록 페이드되도록 구현한 점 또한 벽이 뚫릴 때의 문제를 어느정도는 보완해준다. 원뿔 각도의 변환을 하고 싶다면, 원하는 각도를 바탕으로 탄젠트로 비율을 계산해낼 수 있다. 원뿔의 높이와 반지름을 별도로 조작하는 기능은 언리얼에 따로 없어서, Scale을 이용해 구현했다. 
<div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
  <figure>
    <img src="/assets/img/260526-bp1.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>라인트레이스로 전방까지의 거리를 구한다.<br>이는 스케일, 원뿔의 길이로 보간/적용된다.</figcaption>
  </figure>
  <figure>
    <img src="/assets/img/260526-bp2.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>탄젠트를 이용해 원뿔의 각도를 구해 적용한다.<br>이는 집중에 따른 머티리얼 밝기 조절에 연결된다.</figcaption>
  </figure>
  <figure>
    <img src="/assets/img/260526-bp3.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>광원의 방향,위치,카메라와 이루는 평면까지<br>BP에서 연산하여 넘겨준다. <br> 머티리얼에서 하면 GPU 부하가 심하다.</figcaption>
  </figure>
  <figure>
    <img src="/assets/img/260526-bp4.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>앞서 구한 값들은 머티리얼에 변수로 넘겨준다.<br>거리나 각에 따른 밝기/패턴 변화를 적용한다.</figcaption>
  </figure>
</div>

![image](/assets/img/260526-final.gif)

다 끝내면 대락 이런 모습이다. 보는 방향, 빛의 각도에 따라 메쉬의 크기와 머티리얼의 밝기를 동적으로 변화시키는 모습을 볼 수 있다. 거리 체크에 있어선 경계영역인 대각선까지 총 3개의 라인트레이스로 거리를 조절하는 것도 시도해봤는데, 원치 않는 상황에 지나치게 원뿔 길이 변화가 많아서 광원의 방향, 중앙 하나만 라인트레이스 하도록 남겨두었다.
