---
title: Dental CT - 2
date: 2023-01-02 22:47:09
tags: [CT]
categories: projects
mathjax: true 
---

# Dental CT - 2

<aside>
💡 **험난했던 학부연구생 시절**

---

학부 시절 신호 및 시스템 수업을 너무 재밌게 들은 것이 화근이였다.

영상처리가 재밌어 보여서 연구실에 들어갔더니 웬걸?

Dental CT 제조기업과 연계된 과제에 참여하게 되었다.

그래도 제품에 기여한 좋은 경험이니 정리해본다.

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