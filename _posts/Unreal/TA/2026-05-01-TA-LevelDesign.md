---
title: "TA: 게임에 쓸 간단한 레벨 만들기 & 프롬 생성하기"
author: Jaeseong Kim
date: 2026-05-11 08:00:00 +0800
categories: [Unreal, TA]
tags: [Unreal, TA, Level Design, AI]
---
* 레벨디자인
	* 레벨을 디자인하는 모든 일련의 행위. 지형/동선의 설계, 사물의 배치등 레벨을 만드는데 관련된 과정이 모두 레벨디자인으로 부를 수 있다.
* Environment Light Mixer
	* 언리얼에서 제공하는 기초적인 레벨 광원 세팅용 기능. Directional light, SkyAtmosphere등 전체적인 환경광을 손쉽게 추가,설정할 수 있다.
	![image](/assets/img/260510-lightmixer.png)

* Landscape Mode
	* 지형을 조작할 수 있는 기능으로 산, 계속, 언덕같은 지형을 만들 수 있다. 지형을 추가하고나면 스컬프팅을 하듯 지형을 조작해 만들어낼 수 있다. ZBrush같은 다른 그래픽 툴에서 스컬프팅을 다뤄봤거나, 롤러코스터 타이쿤/플레닛코스터같은 시뮬레이션 게임에서 지형변경을 사용해봤다면 쉽게 익숙해질 수 있는 형태를 가지고있다.
	![image](/assets/img/260510-landscape.png)

* Emmisive Color를 이용하는게 PointLight보다 부하가 낮다.
	* 특정한 위치에 조명, 가로등처럼 광원을 배치하고자 한다면, PointLight를 이용하는게 기초이겠으나, 머티리얼의 Emmisive Color를 이용해서 이 효과를 대체할 수도 있다.
	* 언리얼5의 루멘이 들어오면서 페이크 라이트라는 개념으로 빛처럼 밝히는 것으로 보여도 GPU연산에는 들어가지 않아서 성능상 더 가볍다.
* Forward lighting
	* 빛이 그려질 때 바로바로 계산한다. 기본적이고 전통적인 방식이다.
	* Deferred lighting보다 메모리 사용량이 적고 투명오브젝트를 처리하는데 유리하지만, 다수의 광원을 처리하는데는 비용이 크다.
	* 바로바로 곱연산으로 계산해줘야 하기에 사물이 많아지면 그만큼 느려진다.
* Deferred lighting
	* 정보만 저장해놓고 빛에 대한 연산을 나중에 한번에 처리한다.
	* 다수의 광원을 처리하는데는 효율적이지만, 메모리를 ForwardLighting보다 더 사용하고, MSAA를 사용하는데는 연산 비용이 증가해 궁합이 좋지 않다.
	* 언리얼 모바일은 Deffered Lighting을 이용하길 추천하는데, 다만 과거에는 채널 문제로 지원하지 않았다.
* 게임에 쓸 간단한 레벨 제작
	* 멀리 보일 원경을 만들거나, 자연스러운 지형을 만들고자 한다면 Landscape를 이용한 지형 조작이 적합하겠지만, 건물의 내부같이 일정하고 평면적인 지형을 만드는데는 단순한 에셋의 배치로도 충분할 수 있겠다.
	* 대략적으로 만들어진 레벨의 전경은 현재 다음과 같다. 괴물과 귀신으로 가득찬 병원이라는 컨셉이다. VolumetricFog로 흐릿하고 음산한 분위기를 연출했다. 사진은 임시로 Directional light를 추가하고 캡처했다.
	![image](/assets/img/260510-level1.png)

		* 각 방은 테마에 맞는 에셋들을 배치해 분위기를 조성하는 한편, 적들을 적절히 배치해 처치해야만 해당 방의 자원을 얻을 수 있도록 구성했다.
		<div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
		<figure>
			<img src="/assets/img/260510-level2.png" alt="" style="width:100%; aspect-ratio:1/1; object-fit:cover;">
			<figcaption>플레이어가 시작하는 회복실</figcaption>
		</figure>
		<figure>
			<img src="/assets/img/260510-level3.png" alt="" style="width:100%; aspect-ratio:1/1; object-fit:cover;">
			<figcaption>Scrubbing Room</figcaption>
		</figure>
		<figure>
			<img src="/assets/img/260510-level4.png" alt="" style="width:100%; aspect-ratio:1/1; object-fit:cover;">
			<figcaption>마취실</figcaption>
		</figure>
		<figure>
			<img src="/assets/img/260510-level5.png" alt="" style="width:100%; aspect-ratio:1/1; object-fit:cover;">
			<figcaption>멸균실</figcaption>
		</figure>
		<figure>
			<img src="/assets/img/260510-level6.png" alt="" style="width:100%; aspect-ratio:1/1; object-fit:cover;">
			<figcaption>수술실</figcaption>
		</figure>
		</div>

		* 보스방은 거대한 강당같은 구조로, 사물들을 이용해 따라오는 보스 몬스터를 피해 도망치며 배회할 수 있다.
		![image](/assets/img/260510-level7.png)

* 게임에 쓸 프롭 생성법
	* 팹이나 에셋스토어에서 필요한 모든 것들을 조달할 수 있으면 좋겠지만, 때로는 원하는 느낌의 에셋이 없을 수도 있다. AI를 통한 이미지 생성으로 간단하지만 구하기 힘든 프롭들을 충분히 만들어낼 수 있다. 폴리곤이 최적화되어있지 않은점은 아쉽지만, 리토폴로지와 베이킹을 할줄 안다면 개선해서 쓸수도 있고, 5만 폴리곤까진 나나이트가 언리얼엔진에선 잘 돌아가게 만들어준다.
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
