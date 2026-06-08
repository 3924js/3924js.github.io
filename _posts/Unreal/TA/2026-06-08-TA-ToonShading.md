---
title: "TA: 툰 쉐이딩(Toon Shading)"
author: Jaeseong Kim
date: 2026-06-08 02:00:00 +0800
categories: [Unreal, TA]
tags: [Unreal, TA, Toon Shading]
---
## 비 사실적 랜더링(Non-Photorealistic Rendering, NPR)
비 사실적 랜더링(Non-Photorealistic Rendering, NPR)은 그래픽 느낌이나 성능등 여러 목적 하에 게임이나 영화에서 많이 보이는 그래픽 기법의 큰 범주이다. 물리기반 랜더링(Physical-Based Rendering, PRB), 사실주의 렌더링이 아닌 대부분의 기법을 통칭하는 분류로 쓰이는 편인데, 게임 업계에서 제일 대표적으로 찾아볼 수 있는건 카툰 랜더딩/툰쉐이딩같은 2D 그림 스타일의 그래픽일 것이다. 다만 실제로는 구별되는 스타일들로 여럿 나뉠 수 있다.
1. 툰 쉐이딩/ 셀 쉐이딩/ 카툰 랜더링 (Toon Shading/Cel Shading/Cartoon Rendering)
    <div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
    <figure>
        <img src="/assets/img/260609-ninokuni.jpg" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption style="text-align: center;">니노쿠니2</figcaption>
    </figure>
    <figure>
        <img src="/assets/img/260609-mabinogi.jpg" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption style="text-align: center;">Mabinogi</figcaption>
    </figure>
    </div>

	* 가장 흔하게 생각하는 그 2D 애니메이션/그림체 스타일의 그래픽이다.
	* 명암을 몇개의 단계로 끊어서 그림자가 부드럽지 않고 선명하게 진다.
	* 뒤에 올 라인아트/외곽선과 조합되어 일본풍 애니 스타일에 자주 쓰인다.
	* 몇가지 방법이 있는데, 흔하고 기초적인 방법은 N·L(노멀과 빛의 내적)을 특정 값에 따라 양자화하는 것이다.
2. 그래픽 노블 풍(Graphic Novel Style)
    <div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
    <figure>
        <img src="/assets/img/260609-Wolf.jpg" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption style="text-align: center;">디 울프 어몽 어스</figcaption>
    </figure>
    <figure>
        <img src="/assets/img/260609-borderland.jpg" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption style="text-align: center;">보더랜드3</figcaption>
    </figure>
    </div>
	* 이전의 툰 쉐이딩과 근본은 비슷할 수 있으나, 서구권 만화/코믹북의 특징이 더 많이 드러난다.
		* 거칠고 굵은 선
		* 강렬한 색대비, 포스터같은 명암
		* 해칭, 스크래칭, 먹선같은 보조적인 명암 표현
		* 말풍선이나 의성어같은 코믹북 스타일 UI
3. 라인아트(Line Art)
    <div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
    <figure>
        <img src="/assets/img/260609-sable.jpg" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption style="text-align: center;">세이블</figcaption>
    </figure>
    <figure>
        <img src="/assets/img/260609-Synergy.jpg" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption style="text-align: center;">시너지</figcaption>
    </figure>
    </div>
	* 선으로 그린 그림같은 표현에 집중한다. 툰쉐이딩과 함께 쓰이는 일이 많지만, 선그림 특유의 느낌에 집중하는 형태도 보일 때도 있다.
		* 실루엣 라인/윤곽선을 이용한 표현
		* 단조로운 색상과 명암
		* 
4. 회화풍/ 페인터리 랜더링(Paintary Rendering)
    <div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
    <figure>
        <img src="/assets/img/260609-ori.jpg" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption style="text-align: center;">오리와 눈먼 숲</figcaption>
    </figure>
    <figure>
        <img src="/assets/img/260609-arcane.jpg" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption style="text-align: center;">아케인</figcaption>
    </figure>
    </div>
	* 붓그림, 유화같은 회화의 느낌을 중심으로 하는 스타일이다. 어느정도는 그래픽 노블 풍과 겹치는 느낌도 있다.
		* 붓터치같은 거친 텍스처
		* 회화같은 색 구성
5. 팬앤잉크/해칭/흑백 고대비
    <div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
    <figure>
        <img src="/assets/img/260609-obradinn.jpg" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption style="text-align: center;">오브라 딘 호의 귀환</figcaption>
    </figure>
    <figure>
        <img src="/assets/img/260609-lila.jpg" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption style="text-align: center;">Who's Lila</figcaption>
    </figure>
    </div>
	* 펜으로 그린 것 처럼 선의 밀도로 명암을 표현하는 스타일이다.
	* 선의 밀도와 방향, 흐름등으로 명암이나 색을 구분/표현할 수 있다.
	* 다만 3D 그래픽과 애니메이션에서 볼때는 지글거리는 듯한 느낌을 받을 수도 있다.
6. 종이공예/클레이/스톱모션풍
    <div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
    <figure>
        <img src="/assets/img/260609-yoshi.jpg" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption style="text-align: center;">요시 크래프트 월드</figcaption>
    </figure>
    <figure>
        <img src="/assets/img/260609-Harold.jpg" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption style="text-align: center;">해롤드 할리벗</figcaption>
    </figure> 
    </div>
	* 수공예 특유의 재질과 모델링 스타일에 보다 집중하는 스타일이다.
        * 거친 질감
        * 끊기는 듯한 프레임
        * 세트장, 모형같은 디자인
        * 자연광보단 조명같은 광원
7. 로우폴리/플랫/미니멀 스타일
    <div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
    <figure>
        <img src="/assets/img/260609-monument.jpg" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption style="text-align: center;">모뉴먼트 벨리</figcaption>
    </figure>
    <figure>
        <img src="/assets/img/260609-hike.jpg" alt="" style="width:100%; aspect-ratio:4/3; object-fit:cover;">
        <figcaption style="text-align: center;">A Short Hike</figcaption>
    </figure>
    </div>
	* 인디게임에서 꽤나 많이 봤을법한 스타일로, 낮은 폴리곤 수의 모델로 구성한다. 디자인의 간편함과 성능에서의 장점등이 있다.
        * 극도로 낮은 폴리곤 수
        * 디테일보단 형태 중심의 표현
        * 색감이나 택스처등 느낌은 게임마다 다양하게 표현 가능

## 머티리얼 vs 포스트 프로세싱
툰쉐이딩이나 NPR을 구현하는데 있어서 머티리얼과 포스트 프로세싱은 둘 다 사용 가능한 기술이지만 서로 경쟁관계라기보단 상호보완적인 관계이다.
* 머티리얼 - 각 물체의 느낌을 구현
	* 명암을 단계화하고 기초적인 색감을 제어한다.
	* 평면, 피부, 털, 천같은 재질의 느낌을 부여한다.
	* 특정한 매쉬나 부위등 강조가 필요한 부분을 특수하게 제어한다.
* 포스트 프로세싱 - 전체적인 느낌/통일감을 구현
	* 전체적인 색감/명암을 조정하고 톤앤매너를 통일한다.
	* 외곽선을 생성할 위치를 확인해 입힌다.
	* 카메라 효과 부여한다.

## 길티기어 스타일의 툰 쉐이딩
<iframe width="560" height="315" src="https://www.youtube.com/embed/tWcaQ3gCbUU?si=NTayNOYcaTOWSIkb" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

블렌더 기준 길티기어식 툰쉐이딩 만들기에 대한 영상이다. 블렌더를 기준으로 하지만 개념 자체는 언리얼이나 다른 환경에서도 충분히 적용 가능하다. 요약하노라면 다음정도로 정리할 수 있겠다. 해당 영상 기준으론 모델링과 머티리얼에서 활용할 수 있는 방법들 중심으로 나와있다.

* 간단요약: 단순히 쉐이딩 구간을 양자화 하는것으론 2D 애니스럽지 않음, 추가적인 테크닉 필요.
	1. 기존 그림자는 너무 사실적임, 애니스럽지 않음.
		* 매쉬/파츠별로 다른 라이트 방향 적용 - 3D 특성상 생기는 2D스럽지 않은 사실적인/무시되야하는 그림자 제거

	2. 1번 방법 적용시 그림자의 방향을 예측하기 힘듬/이상하게 보일 때 있음
        * -> 그냥 그림자 전부 비활성화
        * -> 혹은 메쉬에 비치는 그림자(Self Shadow) 말고 땅에 비칠 그림자(Drop Shadow)만 기존 메쉬에서 가져옴.
        * 추가적인 문제 - 그럼 기존에 그림자가 있어야 할 곳이 그림자지지 않게됨.
            * -> 텍스처에 음영까지 넣어서 간단하게 대체 가능
            * -> Vertex Color - 버텍스에 색 지정해놓기

   	3. 윤곽선의 일정함, 메쉬에 가려져서 끊기는 문제
        * -> Vertex Color로 이것까지 해결 가능 (Blender - VertexGroup으로 부분을 구분, 각 그룹마다 다른 가중치 적용)
        * 하지만 외곽만 적용 가능, 메쉬의 외곽이 아닌 면쪽은?
            * -> 그냥 텍스처 사용, 다만 텍스처 이미지에선 수직/수평으로 윤곽선 배치, 그리고 UV Mapping에서 조절하면 깨지는 텍스처가 아닌 깔끔한 선 표현 가능

   	4. 애니메이션의 자연스러움은 모델링과는 다른영역
        * -> 프레임 단위 애니메이션 - 보간으론 결국 2D스런 과장된 느낌 못줌, 사람이 직접 어느정도는 조정해줘야 함.
        * -> 카메라 테스트 - 결국 보이는건 모든 방향이 아닌 카메라의 뷰 하나, 다른방향에선 망가져보이는 프레임도 활용.
        * -> 망가뜨리기 - 카메라 테스트와 비슷한 내용, 3D는 결국 사람보다 완벽, 2D스럽게 인간의 실수같은 느낌을 의도적으로 넣어줘야함.

   	5. 더 발전시키기 -> 노멀 사용.
        * -> 노멀을 부위별로 그룹되게 조정해놓아 빛의 방향에 따라 그림자 모양이 크게 바뀌도록 제작

* **초간단요약: 수작업 노가다 필요.**
    * ->기술이 발전되고 효과적이여도 그 **느낌과 방향성은 사람이 결정**하고 완성해야 한다.
