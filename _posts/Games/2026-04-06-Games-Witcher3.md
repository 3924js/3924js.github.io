---
title: "게임 분석: 위쳐3: 와일드 헌트(Witcher3: Wild Hunt)"
author: Jaeseong Kim
date: 2026-04-06 12:00:00 +0800
categories: [Game, Inspection]
tags: [Game, Inspection, Witcher]
---
## 위쳐 3: 와일드헌트(Witcher3: Wild Hunt)
폴란드의 게임사 CD프로젝트 레드(CD Projekt RED)에서 자체개발 엔진으로 제작해 **2015년 출시한 RPG게임**으로, 온라인 게임 위주로 플레이해보다가 대학생때 처음으로 경험해본 첫 AAA게임이다. 처음 살 때도  이미 출시된지 꽤 되어서 높은 할인률로 판매해주었기에 퀄리티있는 게임에 대한 가성비있는 경험이였다고 할 수 있겠다. 사이버 펑크 2077을 출시할 때 보여준 상황과 비슷하게, 출시 초기에는 버그나 부실함으로 악평을 많이 받았다가 패치와 DLC로 하여금 거의 새 게임으로 탄생했다고 했으니, 처음에 해봤으면 꽤나 실망하고 이런 RPG류는 접근 안했을 수도 있겠다.
둠, 호그와트 레거시등 벤치마크 용도로 사용되는 꽤나 많은 게임들이 있는데, 위쳐3도 그 목록에 끼어 꽤나 오랫동안 사용되어 왔다. 게임플레이와 기술력, 그래픽등 전반에 있어서 꽤나 완성도 있는 작품이였다고 할 수 있겠으나, 처음 플레이할 당시 1050TI라는 게임의 모든 가능성을 느끼기엔 부족한 그래픽카드였기에 완벽히 경험하기에는 무리가 있었다. 지금은 성능이 풀옵을 하더라도 돌아갈 정도는 되기에, 다시 한번 게임에서 확인할 수 없었던 점들을 확인할 수 있겠다. 또한 **2022년 업데이트를 통해 레이트레이싱과 DLSS등 최신 기술도 추가**적으로 업데이트 되면서, 기존 렌더링 방식과 레이트레이싱의 광원 품질 차이등도 비교해볼 수 있을 것이다. 새로운 플레이에선 RT와 DLSS를 모두 키고 보도록 하겠다.
![image](/assets/img/260406-title.png)
## 게임 플레이/최적화
기본적으로 광활한 맵을 지역 단위로 나눠놓은 **세미 오픈 월드** 구조로 스토리와 플레이 진행에 따라 지역을 옮겨가며 플레이하도록 되어있다. 추측컨데 이는 지역별 개발 과정을 분리하고 로딩 시간을 줄여주었을 것으로 보인다. 오픈월드에서 쓰는 방식처럼 맵을 일정 단위/청크로 분할하여 로딩하는 방식으로 줄일 수도 있겠고, 실제로 가장 넓고 주로 플레이하게되는 노비그라드 지도에선 적용되는 모습이 보인다. 다만 이로 제한하기에도 확인해야할 에셋과 데이터들의 영역이 넓다면 탐색하는데도 영향을 줄 것이다. 하지만 플레이어가 다녀야할 스켈리게, 닐스가드등 다른 진영의 맵들을 로딩할 목록 자체에 존재하지 않게함으로서, 어떤 청크를 로딩해야 할지 확인하는 시간 자체를 줄여줄 수 있을 것이다.
**거리에 따른 렌더링 여부 설정**하는 방식도 눈에 띈다. 이는 평소 게임 특성상 건물/지형에 가려서 멀리 지평선이 볼 일은 많지 않고, 바다를 항해하는 스켈리게와 높은 산에 올라갈 일이 있는 노비그라드 남부와 케어모헨, 그리고 DLC에서 플레이하는 투생 지역에서 주로 확인할 수 있다. 멀리 있는 사물은 결국 렌더링 결과물에서도 작게 표시되고, 그에 따라 전반적인 그래픽 경험에서의 중요도는 떨어진다. 때문에 존재하더라도 랜더링 목록에 굳이 포함하면 성능만 잡아먹고 플레이 경험에는 영향이 미미하기에, 굳이 가까이서 볼때만큼 세밀히 렌더링할 이유가 없다. 에이펙스나 배틀로얄의 낙하때 멀리있는 지형에 대해 덜 세밀하게 보이는 것과 같은 방식으로, 이는 다른 렌더링 최적화와도 비슷한 맥락으로 위에서 말한 청크 기반 로딩과 유사한 효과를 낼 것이다.

<div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
  <figure>
    <img src="/assets/img/260406-crowd1.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>가까이 있으면 군중이 모두 보인다.</figcaption>
  </figure>
  <figure>
    <img src="/assets/img/260406-crowd2.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>거리가 멀어지면 렌더링되지 않는다.</figcaption>
  </figure>
  <figure>
    <img src="/assets/img/260406-landscape1.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>지형사물도 가까이에선 모두 보이지만,</figcaption>
  </figure>
  <figure>
    <img src="/assets/img/260406-landscape2.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>거리가 멀어지면 렌더링되지 않는다.</figcaption>
  </figure>
</div>

또한 다른 게임을 보았던 것들에 비해, **인게임 시네마틱을 짧게, 많이** 집어넣었다. 인게임 시네마틱은 게임의 몰입도를 올리는 장치임과는 별개로, 성능에서 보노라면 씬 전환 과정에서 필요한 로딩과 연산을 사전에 진행할 시간을 벌어주는 장치이기도 하다. 이는 특히 맵과 주변 환경이 역동적으로 변해야 하는 후반부 다른 세계를 탐험하는 과정과 최종보스들과의 전투 씬에서 많이 보이는데, 다른 세계에 왔다는 수준의 변화를 성능으로 인한 버벅임 없이 보이기 위한 기술적 장치라 할 수 있겠다.

한편 요즘 자주 쓰이는 개념인 **절차적 생성은 따로 들어가지 않았다.** 나무나 물, 사물들을 보면 다르기보단 회전 혹은 사이즈정도만 어느정도 조절하여 배치된 모습을 볼 수 있다. 그래도 꽤나 밀도있고 신경써서 배치했기 때문에 월드가 비거나 어색해보이진 않는다.
<div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
  <figure>
    <img src="/assets/img/260406-same1.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>같은 꽃 패턴으로 반복한다.</figcaption>
  </figure>
  <figure>
    <img src="/assets/img/260406-same2.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption>나무들의 모양이 다르지 않다.</figcaption>
  </figure>
</div>
## 색감/질감
사실적인 위쳐 세계의 몰입과 모험을 위해 사실적인 그래픽에 보다 초점을 맞춘 것으로 생각 된다. 이를 위해선지 전반적인 사물의 전반적인 텍스처에서 단독으로 보기엔 밋밋하고 어색하다고 보일 수 있는 **높은 명도와 낮은 채도**를 주로 쓴 것 같다. 제일 빛이 강할 때인 정오에 벽돌과 지붕을 많이 볼수 있는 노비그라드 도시가 이를 잘 보여주는데, 사물들을 보면 전반적으로 많이 먼지앉은듯 색이 죽은듯한 회색/흰색의 느낌이 든다. 강한 햇빛의 반사를 반영했다고 평할수도 있겠지만, 그만큼 텍스처로 전달되는 사물의 느낌 자체가 약하다는 표현이기도 하겠다. 하지만 뒤에서 말하겠지만 이는 다양한 광원과 조합되어 다양한 환경을 시각적으로 반영하는데 도움을 준다.
<div style="display: grid; grid-template-columns: repeat(1, 1fr); gap: 10px;">
  <figure>
    <img src="/assets/img/260406-noon1.png" alt="" style="width:100%; aspect-ratio:4/2; object-fit:cover;">
  </figure>
  <figure>
    <img src="/assets/img/260406-noon2.png" alt="" style="width:100%; aspect-ratio:4/2; object-fit:cover;">
  </figure>
</div>

**환경에 따른 질감의 변화**도 사실적인 경험을 더한다. 물에 빠졌다가 빠져나올 때 반사가 더 강해지고 기존의 텍스처가 덜 투영되는 젖은 느낌으로 변하고, 시간이 지남에 따라 원래대로 돌아온다. 이는 게임에서 많이 있을 수시로 물속을 들락거리고 수영하며 온갖 장소에서 구르는 캐릭터를 더 사실적으로 만들어준다.
<div style="display: grid; grid-template-columns: repeat(4, 1fr); gap: 10px;">
  <figure>
    <img src="/assets/img/260406-leather1.png" alt="" style="width:100%; aspect-ratio:1/3; object-fit:cover;">
    <figcaption style="text-align: center;">가죽 젖기 전</figcaption>
  </figure>
  <figure>
    <img src="/assets/img/260406-leather2.png" alt="" style="width:100%; aspect-ratio:1/3; object-fit:cover;">
    <figcaption style="text-align: center;">가죽 젖은 후</figcaption>
  </figure>
  <figure>
    <img src="/assets/img/260406-plate1.png" alt="" style="width:100%; aspect-ratio:1/3; object-fit:cover;">
    <figcaption style="text-align: center;">판금 젖기 전</figcaption>
  </figure>
  <figure>
    <img src="/assets/img/260406-plate2.png" alt="" style="width:100%; aspect-ratio:1/3; object-fit:cover;">
    <figcaption style="text-align: center;">판금 젖은 후</figcaption>
  </figure>
</div>

머리카락은 다른 게임들과 비슷하게 선들로 이루어진 텍스처를 몇개의 단으로 연결하는 방식을 사용했는데, 다른 게임들과 큰 차이를 보이진 않는다. Nvidia Hairworks 옵션을 켰을 때 더 자연스럽게 움직이는 것을 볼 수 있는데, 긴 머리의 캐릭터의 움직임에서 특히 그렇다. 그러나 DLSS, RT와의 조합은 좋지 않은지, 같이 쓰면 계속 머리카락이 쉬지않고 움직이는 모습을 보여준다. 괜히 성능만 잡아먹는 것 같고 풍경을 볼일이 더 많아 나도 예전엔 끄고 게임했던걸로 기억한다.
<div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
  <figure>
    <img src="/assets/img/260406-hairworks-disabled.png" alt="" style="width:100%; aspect-ratio:1/1; object-fit:cover;">
    <figcaption>기본 머리 품질,<br>덩어리져있고, 보다 각진 모습이다.</figcaption>
  </figure>
  <figure>
    <img src="/assets/img/260406-hairworks-enabled.png" alt="" style="width:100%; aspect-ratio:1/1; object-fit:cover;">
    <figcaption>Nvidia Hairworks 사용,<br>세밀한 머리카락 묘사와 움직임이 들어간다.</figcaption>
  </figure>
</div>
## 광원/렌더링/이펙트
전반적으로 빛이 있을 때를 중심으로 많이 설계되지 않았나 생각이 든다. 머티리얼의 질감과 색감과 함께, 빛이 있는 낮 시간대의 맵이나 도시지역은 선명하고 자연스러운 색을 많이 보여준다. 머티리얼에서 전반적으로 낮은 채도의 밋밋한 색감을 주로 사용하고, 이를 **주황색, 노란색등 **다양한 광원의 색을 이용**해 보충하는 구조를 보이는 것으로 보인다. 이는 보다 환경에 따라 변화하는 "살아있는" 세계의 느낌을 살려줄 것이다. 
<div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
  <figure>
    <img src="/assets/img/260406-time06.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
  </figure>
  <figure>
    <img src="/assets/img/260406-time12.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
  </figure>
  <figure>
    <img src="/assets/img/260406-time20.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
  </figure>
  <figure>
    <img src="/assets/img/260406-time24.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
  </figure>
</div>

한편 카메라의 세팅이 빛의 양에 따라 조절되는 자동노출은 없는 것 같다. 이는 동굴, 밤 시간대등 빛이 없을 때  사물의 형상이나 위치를 확인하는데 불편함을 만들고 특히 이 게임에서 적지 않은 비중을 차지하는 밤시간대나 동굴지역에서의 탐험을 성가시게 만든다. 횃불과 시야개선용 포션을 제공함으로서 해소한다는 점에서, 아마도 의도적인 사항일 수도 있겠다.
<div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
  <figure>
    <img src="/assets/img/260406-cave0.png" alt="" style="width:100%; aspect-ratio:1/1; object-fit:cover;">
  </figure>
  <figure>
    <img src="/assets/img/260406-cave1.png" alt="" style="width:100%; aspect-ratio:1/1; object-fit:cover;">
  </figure>
</div>

지역이나 이펙트에 따라 사물이 왜곡되는 모습을 구현하기도 했는데, 스킬효과나 습지같은 제한적인 효과는 객체처럼 배치함으로서 구현한다. 환경 전체를 수정하지 않고도 부분적으로 분위기를 구현하는 역할을 한다. 한편, 스켈리게같은 군도지역이나 물속에선 카메라의 가시거리, 렌더링 방식을 차이두는 형식으로 전체적으로 효과를 적용한다. 스켈리게 군도에서의 효과 차이는 위에서 언급한 밋밋한 색감, 흐릿한 광원과 함께 해당 지역의 더 차가운 느낌을 표현하는데 일조한다.
<div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
  <figure>
    <img src="/assets/img/260406-effect1.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption style="text-align: center;">NPC 이펙트</figcaption>
  </figure>
  <figure>
    <img src="/assets/img/260406-effect2.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption style="text-align: center;">스킬 파티클+왜곡</figcaption>
  </figure>
</div>
<div style="display: grid; grid-template-columns: repeat(1, 1fr); gap: 10px;">
  <figure>
    <img src="/assets/img/260406-effect3.png" alt="" style="width:100%; aspect-ratio:4/2; object-fit:cover;">
    <figcaption style="text-align: center;">습지 안개, 몬스터 피격 이펙트</figcaption>
  </figure>
</div>
<div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
  <figure>
    <img src="/assets/img/260406-Islands1.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption style="text-align: center;">스켈리게 군도 절벽</figcaption>
  </figure>
  <figure>
    <img src="/assets/img/260406-Islands2.png" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
    <figcaption style="text-align: center;">스켈리게 군도 원경</figcaption>
  </figure>
</div>

추후에 추가된 **레이트레이싱과 DLSS**는 전반적인 퀄리티를 끌어올린다. 특히 부분적으로 점묘화처럼 흐릿하거나 블러처리된것 같이 보이는 일부 디테일들을 더 정확하고 깔끔하게 구현하는데 크게 기여한다. RT 혹은 DLSS한쪽만 활성화해도 개선을 보이지만, 둘이 합쳐졌을때의 결과물은 최고의 퀄리티를 보여준다. 다만 이는 작은 사물과 반사광, 각종 요소들이 빼곡히 차있는 풍경을 볼 때 도드라지는 편이고, 게임의 대부분을 차지할 **전투와 이동등에선 크게 주목받지 못할 요소**라 생각한다. 기존 그래픽도 지금봐도 좋은 양질의 결과물을 내기 위해 노력이 들어갔다는 반증일 것이다.
<div style="display: grid; grid-template-columns: repeat(1, 1fr); gap: 10px;">
  <figure>
    <img src="/assets/img/260406-comp-FXAA.png" alt="" style="width:100%; aspect-ratio:4/1; object-fit:cover;">
    <figcaption style="text-align: center;">FXAA</figcaption>
  </figure>
  <figure>
    <img src="/assets/img/260406-comp-FXAA+RT.png" alt="" style="width:100%; aspect-ratio:4/1; object-fit:cover;">
    <figcaption style="text-align: center;">FXAA + RT</figcaption>
  </figure>
  <figure>
    <img src="/assets/img/260406-comp-DLSS.png" alt="" style="width:100%; aspect-ratio:4/1; object-fit:cover;">
    <figcaption style="text-align: center;">DLSS</figcaption>
  </figure>
  <figure>
    <img src="/assets/img/260406-comp-RT+DLSS.png" alt="" style="width:100%; aspect-ratio:4/1; object-fit:cover;">
    <figcaption style="text-align: center;">DLSS + RT</figcaption>
  </figure>
</div>


## 총평: 당대 높은 기술력의 산물, 지금은 꽤나 흔해진 결과물?
잘 설계된 게임플레이 동선과 그에 몰입감을 더해주는 사실적인 그래픽을 구현하기 위해 전반적으로 선명하지만 칙칙한 머티리얼을 많이 사용하고, 그 밋밋함을 광원과 렌더링 기술로 매우는 선택을 한 것으로 보인다. 이는 렌더링 결과물이 환경에 따라 다채롭게 변하는 것을 초점을 맞춘 것으로, 때때로 가시성과 프레임레이트에 악영향을 줄 수도 있겠으나, 전반적인 플레이에는 큰 악영향이 없다고 본다. 한편 이 게임의 그래픽 품질과 느낌은 출시된 사이버펑크의 쨍하고 선명한, 펑크스럽고 도시같은 느낌과는 방향성이 다르다고 할 수도 있다.

이제는 RT, DLSS의 보급으로 이정도의 그래픽 품질을 뽑아낼 수 있는 게임은 꽤나 많아진 것 같다. 오히려 RT없는 기본적인 쉐이딩과 렌더링이 성의없어보이는 바이오하자드 레퀴엠같은 신작을 보면, 앞으로는 보기 힘든 노력의 결과물일 꺼란 생각이 들기도 한다. 개발중인 위쳐 4에선 자체개발 엔진 대신 언리얼5를 쓴다고 하는데, 출시되면 비교해보기 좋은 게임이 되지 않을까?
