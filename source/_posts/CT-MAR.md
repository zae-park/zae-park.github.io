---
title: CT - Metal Artifact Reduction
date: 2023-01-02 22:47:09
tags: [CT, Radon transform, Fourier transform, sinogram]
categories: projects
mathjax: true 
---

### Metal Artifact Reduction
- [Radon transform](#radon-transform)
- [Back-projection](#back-projection)

이전 포스트에서는 CT imaging 기초 원리에 대한 내용을 다루었다. 이번 포스트에서는 치과용 CT에서 치아 내 임플란트 등의 금속물이 있는 경우 imaging 과정이 어떻게 변하는지 설명하고 프로젝트를 어떻게 진행했는지에 대해 정리하려 한다.

---

## Metal Artifact Reduction

CT는 x-ray가 인체 내 조직을 거치며 잃어버린 energy를 image로 나타낸 것이다. 조직마다 각각 다른 attenuation 계수를 가지고 있고, 심지어 이 attnuation은 조직의 밀도와 선형적이지 않다.
또, 만약 촬영하려는 부위에 금속물이 있다면 그 근방에 **streak artifact**라는 흰색-검은색 줄무늬가 발생하게 되어 image를 왜곡시킨다. 문제는, 이 금속물의 개수가 2개 이상이 되면, 각 금속물 사이에 streak artifact가 크게 발생하고, 금속물 근방 뿐 아니라 사이에 위치한 image를 전부 왜곡시킨다.
예를 들어 큰 수술 이력이 있어 뼈에 금속 보형물이 있다거나, 임플란트나 브릿지와 같은 치과 시술을 받은 경우가 문제가 된다.

![Distorted Image](resource/ct_mar/dictorted_image.png)

왜 이런 강한 artifact가 발생하게 될까?

### 


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