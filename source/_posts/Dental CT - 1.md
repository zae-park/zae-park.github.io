---
title: Dental CT - 1
date: 2023-01-02 22:47:09
tags: [CT]
categories: projects
mathjax: true 
---


학부 연구생 생활 중 Dental CT 제조기업과 함께 프로젝트를 진행한 적이 있다. </br>
연구실 선배들과 같이 열심히 연구하고 실제 제품에도 기여한 좋은 경험을 정리해본다.

---

## Computed Tomography.

CT는 X-ray 영상을 다각도에서 촬영하는 기법이다. </br>
그러면 대체 투과 영상인 X-ray를 어떻게 해야 CT 같은 단층 촬영 영상이 나올까?

먼저, X-ray 영상을 떠올려 보면, (대개는 흉부 X-ray 영상을 상상한다) 지방, 근육등의 조직은 흐리거나 투명하게, 뼈와 같은 밀도가 높은 조직은 하얗게 나타난다. 이는 X-ray 영상이 조직이 `감쇄된 energy`를 나타내기 때문이며, 감쇄된 정도는 `조사한 energy` - `검출된 energy`로 계산한다.
$$
E_{attenuated} = E_{source} - E_{detector}
$$

CT 촬영기법은 이 **감쇄된 energy**를 모든 각도에서 촬영하고 이를 다시 회전각-감쇄도 축에 그려 `sinogram` 이라는 그래프를 만든다. 다시말해 이 `sinogram`은 "각도에 따라 느껴지는 감쇄정도"로 해석할 수 있다. 마치 초등학생 시절 블럭으로 이상한 모양의 탑을 쌓아두고 다각도에서 바라보았을 때 몇개의 블럭이 보이는지 묻는 문제와 같은데, 이것이 라돈 변환이랑 매우 비슷하다. </br>

![Sinogram of Phantom](resource/ct_1/sinogram.png)

아래 그림은 라돈변환의 예시인데, 그림과 함께 간단하게 설명하자면 "어떤 점과 점 사이를 선적분한 값" 을 표현한 것이다. 이 선적분된 값들을 회전각 축으로 모아둔다면 위의 `sinogram`과 비슷한 그래프가 그려질 것이다.

![Radon Transform](resource/ct_1/radon.png)

이렇게 CT에서 획득한 `sinogram`은 라돈 변환, 즉 선적분의 결과를 물리적으로 획득한 것이다. 



</aside>

</aside>
<details markdown="1">
    <summary><strong> How to imaging? 간단 정리</strong></summary>
        <ul>
            <li>object를 중심으로 회전하며 여러 각도에서 X-선을 촬영한다.</li>
            <li>detector에서 획득한 attenuated energy들을 sinogram으로 표현한다.</li>
            <li>sinogram은 object를 투과하며 선적분되었다.</li>
            <ul><li>이미 radon transform되어있다.</li></ul>
            <li>Inverse radon transform으로 CT image를 재구성할 수 있다.</li>
            <ul>
                <li>sinogram의 각 angle의 energy를 back-projection한다.</li>
                <li>이 떄, filter를 사용하지 않으면 angle의 수와 비례하여 image가 blur해진다.</li>
            </ul>
        </ul>
</details>
<br>

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