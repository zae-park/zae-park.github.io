---
title: CT - imaging 기초
date: 2023-01-02 22:47:09
tags: [CT, Radon transform, Fourier transform, sinogram, parallel beam, fan beam, corn beam]
categories: CT
mathjax: true 
---

### Contents
- [Computed Tomography](#computed-tomography)
  - [Radon Transform](#radon-transform)
  - [Back-projection](#back-projection)

인생의 첫 연구는 학부 연구생 시절 Dental CT 제조기업(G**)과 함께 진행한 Metal Artifact Reduction 프로젝트다. 너무 어려웠지만 선배들과 같이 연구하고 실제 제품에도 기여한 좋은 경험이니 최대한 잘 정리해보려 한다.

우선, 이 글에서는 CT image의 촬영 원리를 최대한 기억해서 써본다.

---

## Computed Tomography

CT는 X-ray 영상을 다각도에서 촬영하는 기법이다. </br>
그러면 대체 투과 영상인 X-ray를 어떻게 해야 CT 같은 단층 촬영 영상이 나올까?

먼저, X-ray 영상을 떠올려 보면, (대개는 흉부 X-ray 영상을 상상한다) 지방, 근육등의 조직은 흐리거나 투명하게, 뼈와 같은 밀도가 높은 조직은 하얗게 나타난다. 이는 X-ray 영상이 조직이 `감쇠된 energy`를 나타내기 때문이며, 감쇠된 정도는 `조사한 energy` - `검출된 energy`로 계산한다.
$$
E_{attenuated} = E_{source} - E_{detector}
$$

CT 촬영기법은 이 **감쇠된 energy**를 모든 각도에서 촬영하고 이를 다시 회전각-감쇠도 축에 그려 `sinogram` 이라는 그래프를 만든다. 다시말해 이 `sinogram`은 "각도에 따라 느껴지는 감쇠정도"로 해석할 수 있는데 이것이 라돈 변환과 매우 비슷하다. </br>

> 초등학생 수학 시험에서 블럭으로 이상한 모양의 탑을 쌓아두고 다각도에서 바라보았을 때 몇개의 블럭이 보이는지 묻는 문제와 같다.

![Sinogram of Phantom](resource/ct_basic/sinogram.png)

### Radon Transform

아래 그림은 라돈변환의 예시인데, 그림과 함께 간단하게 설명하자면 "어떤 점과 점 사이를 선적분한 값" 을 표현한 것이다. 이 선적분된 값들을 회전각 축으로 모아둔다면 위의 `sinogram`과 비슷한 그래프가 그려질 것이다.

![Radon Transform](resource/ct_basic/radon.png)

이렇게 CT에서 획득한 `sinogram`은 라돈 변환, 즉 선적분의 결과를 물리적으로 획득한 것이다. 그래서 어쩌라는 거냐고? 이제 이 `sinogram`을 다시 image로 복원하기만 하면 된다. 재밌는 점은 이 복원 과정에 또 다시 적분이 사용된다는 점이다. 푸리에 변환과 역변환도 모두 적분을 사용하는 것을 생각하면 꽤 자연스럽다. </br>
엥? 이거 완전 삼각함수 아니냐!

### Back-projection

가장 간단한 복원 방법은, `sinogram`에서 각 회전각마다 그려진 `감쇠된 energy`를 그대로 back-projection하는 것이다. 이는 마치 키보드에서 입력 키를 인식하는 원리와 같다. 아래는 back-projection의 예시이다.

> 옛날 키보드는 키 인식을 위한 축을 2개만 사용해서 동시 입력 시 인식에 실패하는 키 조합이 있었다. 이를 응용하면 피카츄 배구를 유리하게 할 수 있다.

![Back-projection of Phantom](resource/ct_basic/backprojection.png)

그림에서 알 수 있듯이 back-projection이 생각보다 blur하다. 촬영 시 회전각의 변화량을 작게 하고 많은 projection data를 얻을 수 있다면 좋겠지만, X-ray은 방사선이라 많은 양을 조사할 수는 없다.</br>
사실 긴 시간이 지나 왜 naive한 back-projection이 blur한 image를 복원하는지는 기억이 나지 않는다. 다행스럽게도 기억이 나는 것은 `sinogram`의 어떤 회전각의 projection을 1차원 푸리에 변환하면, 같은 회전각을 갖는 푸리에 공간의 주파수 특성을 얻을 수 있다는 점이다. (말이 좀 어렵겠지만... Fourier-Slicing thm을 찾아보면 도움이 될지도?)</br>
일단 주파수 영억으로 넘어가게 되면, 가장 먼저 생각할 수 있는 시도는 filtering이다. Back-projection 이전에 `sinogram`을 filtering하는 방식을 filtered back-projection이라고 하는데, 이 filtering을 convolution으로 대체하기도 해서 convolution back-projection이라고도 부르는 것 같다. 아래는 평범한 back-projection과 filtered back-projection의 비교이다. </br>

![FBP of Phantom](resource/ct_basic/projection.png)

> 초창기 CT는 하나의 source와 하나의 detector가 쌍을 이루는 `pen-beam` 방식을 사용하며, 주로 **hann** 혹은 **hamming** 윈도우로 filtering을 했었다. 하지만, 이 후 하나의 source와 다수의 detector를 사용하는 `fan-beam` 방식을 사용하면서 극좌표계를 사용하여 좀 더 복잡한 filter를 설계해야 했고, 현대에 들어서는 `corn-beam` 방식을 사용하면서 구좌표계를 사용한 filter가 필요하게 되었다.

![Comparison of Beams](resource/ct_basic/CT_beam.png)

---

본 포스트에서는 CT imaging 원리에 대한 기초적인 내용을 다루었다. 이후 포스트에서는 기업 프로젝트로 진행했던 metal artifact reduction, compressive(sparse) sensing에 대한 내용을 차례로 다루려고 한다.

