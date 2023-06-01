---
title: Dental CT - 1
date: 2023-01-02 22:47:09
tags: [CT]
categories: projects
mathjax: true 
---

학부 연구생 생활 중 Dental CT 제조기업과 함께 프로젝트를 진행한 적이 있다. </br>
연구실 선배들과 같이 열심히 연구하고 실제 제품에도 기여한 좋은 경험을 정리해본다.

### Computed Tomography
- [Radon transform](#radon-transform)
- [Back-projection](#back-projection)

---

## Computed Tomography.

CT는 X-ray 영상을 다각도에서 촬영하는 기법이다. </br>
그러면 대체 투과 영상인 X-ray를 어떻게 해야 CT 같은 단층 촬영 영상이 나올까?

먼저, X-ray 영상을 떠올려 보면, (대개는 흉부 X-ray 영상을 상상한다) 지방, 근육등의 조직은 흐리거나 투명하게, 뼈와 같은 밀도가 높은 조직은 하얗게 나타난다. 이는 X-ray 영상이 조직이 `감쇄된 energy`를 나타내기 때문이며, 감쇄된 정도는 `조사한 energy` - `검출된 energy`로 계산한다.
$$
E_{attenuated} = E_{source} - E_{detector}
$$

CT 촬영기법은 이 **감쇄된 energy**를 모든 각도에서 촬영하고 이를 다시 회전각-감쇄도 축에 그려 `sinogram` 이라는 그래프를 만든다. 다시말해 이 `sinogram`은 "각도에 따라 느껴지는 감쇄정도"로 해석할 수 있는데 이것이 라돈 변환과 매우 비슷하다. </br>

> 초등학생 시절 블럭으로 이상한 모양의 탑을 쌓아두고 다각도에서 바라보았을 때 몇개의 블럭이 보이는지 묻는 문제와 같다.

![Sinogram of Phantom](resource/ct_1/sinogram.png)

### Radon Transform

아래 그림은 라돈변환의 예시인데, 그림과 함께 간단하게 설명하자면 "어떤 점과 점 사이를 선적분한 값" 을 표현한 것이다. 이 선적분된 값들을 회전각 축으로 모아둔다면 위의 `sinogram`과 비슷한 그래프가 그려질 것이다.

![Radon Transform](resource/ct_1/radon.png)

이렇게 CT에서 획득한 `sinogram`은 라돈 변환, 즉 선적분의 결과를 물리적으로 획득한 것이다. 그래서 어쩌라는 거냐고? 이제 이 `sinogram`을 다시 image로 복원하기만 하면 된다. 재밌는 점은 이 복원 과정에 또 다시 적분이 사용된다는 점이다. 푸리에 변환과 역변환도 모두 적분을 사용하는 것을 생각하면 꽤 자연스럽다. </br>
엥? 이거 완전 삼각함수 아니냐!

### Back-projection

가장 간단한 복원 방법은, `sinogram`에서 각 회전각마다 그려진 `감쇄된 energy`를 그대로 back-projection하는 것이다. 이는 마치 키보드에서 입력 키를 인식하는 원리와 같다. 아래는 back-projection의 예시이다.

> 옛날 키보드는 키 인식을 위한 축을 2개만 사용해서 동시 입력 시 인식에 실패하는 키 조합이 있었다. 이를 응용하면 피카츄 배구를 유리하게 할 수 있다.

![Back-projection of Phantom](resource/ct_1/backprojection.png)

그림에서 알 수 있듯이 back-projection이 생각보다 blur하다. 촬영 시 회전각의 변화량을 작게 하고 많은 projection data를 얻을 수 있다면 좋겠지만, X-ray은 방사선이라 많은 양을 조사할 수는 없다.</br>
사실 긴 시간이 지나 왜 naive한 back-projection이 blur한 image를 복원하는지는 기억이 나지 않는다. 다행스럽게도 기억이 나는 것은 `sinogram`의 어떤 회전각의 projection을 1차원 푸리에 변환하면, 같은 회전각을 갖는 푸리에 공간의 주파수 특성을 얻을 수 있다는 점이다. (말이 좀 어렵겠지만... Fourier-Slicing thm을 찾아보면 도움이 될지도?)</br>
일단 주파수 영억으로 넘어가게 되면, 가장 먼저 생각할 수 있는 시도는 filtering이다. 이론적으로는 푸리에 변환 후 투사방식에 맞는 filter를 적용하고 다시 푸리에 역변환한 뒤 back-projection을 해야하지만, 그럴 필요 없이 convolution으로도 frequency filtering을 할 수 있었고, 그래서 convolution back-projection이라고도 부르더라!

> 초창기 CT는 하나의 source와 하나의 detector가 쌍을 이루는 `pen-beam` 방식을 사용하며, 주로 사용되는 filter는 **hann** 혹은 **hammin** 윈도우를 사용했다. </br>
> 하지만, 이 후 하나의 source와 다수의 detector를 사용하는 `fan-beam` 방식을 사용하면서 더 복잡한 형태의 filter를 사용하게 되었고, 현대에 들어서는 `corn-beam` 방식을 사용한다. 이게 정말 머리 터진다.



## Task: Metal Artifact Reduction

주 과제는 딥러닝으로 구강 내 금속물로 인한 artifact를 제거하는 것이였다.

문제는 `ground truth image`, 즉 artifact free image를 구할수 없다는 점이였다.

---

그래서 metal이 없는 image로부터 metal artifact를 생성하기로 했다.

정교한 시뮬레이션을 위해서는 metal artifact 생성과정을 알아야한다.

X-선은 object를 투과하며 energy가 감쇠된다.

즉, $E_{source} - E_{detector}$를 imaging하는 것이다.

모든 object는 고유한 감쇠계수를 갖는데, 문제는 이 계수가 E에 의존한다는 점이다.

$$
C_{attenuation} = f(object, E)
$$

실제로는 아래와 같이 streak artifact가 심하게 발생한다.

![실제 distorted image](resource/ct_1/Untitled.png)

불행 중 다행은 **fan beam CT**라 근방의 image에서 **간섭하는 artifact**는 무시해도 된다는 점이다.

---

### Data generation

1. metal이 없는 image 준비하기
    
    metal artifact를 시뮬레이션하기로 했으니 이제 재료가 필요하다.
    
    기업에 간곡히 요청하면 (구걸 x) metal이 없는 image를 받을 수 있다.

    ![metal이 없는 image](resource/ct_1/Untitled_1.png)
    
2. metal 추가하기
    
    원하는 부위(치아)에 원하는 metal을 추가할 수 있어야 한다.
    
    - 치아 선택 기능이 있는 gui 제작한다.
    - 선택된 치아의 edge를 찾고 metal을 생성한다.

    ![GUI](resource/ct_1/Untitled_3.png)
    ![예시](resource/ct_1/Untitled_2.png)

3. CT imaging 시뮬레이션 하기
    
    MATLAB의 도움을 아주 많이 받았다! 최고!

    ![**Left** : 시뮬레이션으로 생성한 distorted image **Right** : difference image = `raw image + metal - distorted image`](resource/ct_1/Untitled_4.png)

    **Left** : 시뮬레이션으로 생성한 distorted image
    **Right** : difference image = `raw image + metal - distorted image`

---

### Method & Result

신경망은 **U-net**을 사용했고, 2가지 학습 방법을 시도했다.

- `Direct` : distorted image → metal free image
- `Residual` : distorted image → difference image
    - metal free image = distorted image - difference image

![Residual 학습 방법](resource/ct_1/Untitled_5.png)

Residual 학습 방법

여러 hyper parameter, data 구성, dynamic range등을 조절하며 두 학습 방법을 비교한 결과

일관적으로 `Residual` 학습 방법이 더 좋은 성능을 보였다 → **criteria = NMSE**

결국, 아래의 reconstructed image를 얻을 수 있었다. 짜잔~

![distorted image](resource/ct_1/Untitled_6.png)


엄청 잘되는 것 같아 보이지만, metal이 너무 가까이 붙어있거나 많으면 artifact가 남는다.

- 치아 사이 입천장 즈음에 dark band는 ROI가 아니라 괜찮지만…
- metal과 metal 사이에 위치한 치아는 형태를 알아보기 어려워지기도 한다.

![](resource/ct_1/Untitled_7.png)

여러 방법을 시도하고 NMSE 기준 TOP-4를 선정.
양치를 너무 안하면 그래도 artifact가 많이 남는다…