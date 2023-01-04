---
title: Dental CT - 1
date: 2023-01-02 22:47:09
tags: [CT]
categories: projects
---
# Dental CT - 1

<aside>

### 💡 **험난했던 학부연구생 시절**

    학부 시절 신호 및 시스템 수업을 너무 재밌게 들은 것이 화근이였다.
    영상처리가 재밌어 보여서 연구실에 들어갔더니 웬걸?
    Dental CT 제조기업과 연계된 과제에 참여하게 되었다.
    그래도 제품에 기여한 좋은 경험이니 정리해본다.

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
<br><br>


## Task: Metal Artifact Reduction
<hr>
<p>
    주 과제는 distorted image의 <b>artifact를 제거하는 것</b>이다. <br>
    <ol>
    <li>Beam hardening</li>
    <p>
        서술한 것과 같이 CT imaging은 X-선의 감쇠로부터 시작한다.
        ㅁㄴㅇ
    </p>
    <li>Streak artifact</li>
    <p>
        photon에서 출발한 X-선은 object를 지나 detector에 도달한다.
        X-선은 서술한 것과 같이 CT imaging은 X-선의 감쇠로부터 시작한다.
        ㅁㄴㅇ
    </p>
    </ol>
    이 artifact들은 주로 뼈, 치아, 금속물과 물질들로부터 발생하는데, 이러한 물질들은 x-선에 대한 저항값이 너무 커서 attenuation coefficient가 너무 높기 때문이다.
    감쇠정도가 너무 detector에서 energy를 획득하지 못하는 경우이다. 는데, 감쇠정도가 너무.
    Dental CT는 imaging 원리에 따라 x-선의 감쇠정도를
    이처럼 <b>beam hardening</b>에 의해 발생하는 artifact를 <b>streak artifact</b>라 한다.
</p>
<p>
    medical segmentation에서 자주 사용되는 <b>U-net</b>을 사용하기로 했다.<br>
    문제는, 크라운같은 금속물은 탈부착이 불가능해서 <br> <b>distorted image & ground truth image</b> 쌍을 만들 수 없다는 점이다.
</p>
<br>

![실제 distorted image](resource/ct_1/Untitled.png){width: 40%}
<br>
<div style='grid-template-columns: 4fr 1fr;'>
    <p style='float: right'>
        <img src='resource/ct_1/Untitled.png'>
    </p>
    <div style='float: left;'>
        <p>
            주 과제는 구강 내 금속물로 인한 <b>artifact를 제거하는 것</b>이다. <br>
            이처럼 <b>beam hardening</b>에 의해 발생하는 artifact를 <b>streak artifact</b>라 한다.
        </p><hr>
        <p>
            medical segmentation에서 자주 사용되는 <b>U-net</b>을 사용하기로 했다.<br>
            문제는, 크라운같은 금속물은 탈부착이 불가능해서 <br> <b>distorted image & ground truth image</b> 쌍을 만들 수 없다는 점이다.
        </p>
    </div>
</div><br>

<p>
    그래서 연구실 자체적으로 CT imaging와 physics에 대한 세미나를 진행했다. 결과는…<br>
    <p>
        → <b>금속물이 없는 dental CT 영상에 가상으로 금속물을 추가해서 artifact를 생성하자</b> 이다.<br>
        → 불행 중 다행은 **fan beam CT**라 근방의 image에서 **간섭하는 artifact**는 무시해도 된다는 점이다.
    </p>
</p><hr>

<details markdown="1">
    <summary><strong style='background: gray'> Data generation</strong></summary>
        <ol>
            <li>raw image 준비하기</li>
            <div style='float: right; width: 10%'>
                <br><img src='resource/ct_1/Untitled_1.png'>
            </div>
            <p style='float: left;'>
                <p>
                metal은 가상으로 추가할 수 있으니 raw image가 필요하다.<br>
                기업에 간곡히 요청하면 (구걸 x) metal free image를 받을 수 있다.<br>
                → 혹시 모르니 상악~하악을 다양한 해상도로 요청한다.
                </p>
            </p>
            <li>metal 추가하기</li>
            <div style='float: right; width: 10%'>
                <br><img src='resource/ct_1/Untitled_3.png'>
            </div>
            <p style='float: left;'>
                <p>
                    원하는 부위(치아)에 원하는 metal을 추가할 수 있어야 한다.<br>
                    <ul>
                        <li>치아 선택 기능 → gui 제작하여 해결</li>
                        <li>metal 생성 → 선택된 치아의 edge를 찾아 생성</li>
                        <ul>
                            <li>edge → 크라운</li>
                            <li>fill → 임플란트</li>
                            <li>edge 내에 작은 metal → 땜빵</li>
                        </ul>
                    </ul>
                    <p style="text-align:center;">
                        <img src='resource/ct_1/Untitled_2.png'>
                        <figcaption>강한 artifact를 위해 연속된 metal도 필요하다.</figcaption>
                    </p>
                    
                </p>
            </p>
        </ol>
</details>


1. raw image 준비하기
    
    metal은 가상으로 추가할 수 있으니 raw image가 필요하다.
    
    기업에 간곡히 요청하면 (구걸 x) metal free image를 받을 수 있다.
    
    - 혹시 몰라서 상악~하악까지의 모든 slice를 요청하였다.
    - 또 혹시 몰라서, 다양한 해상도의 image를 요청하였다.

![Untitled](resource/ct_1/Untitled_1.png)

1. metal 추가하기
    
    원하는 부위(치아)에 원하는 metal을 추가할 수 있어야 한다.
    
    - 치아 선택 기능 → gui 제작하여 해결
    - metal 생성 → 선택된 치아의 edge를 찾아 생성
        - edge → 크라운
        - fill → 임플란트
        - edge 내에 작은 metal → 땜빵
        
        ![강한 artifact를 위해 연속된 metal도 필요하다. ](resource/ct_1/Untitled_2.png)
        
        강한 artifact를 위해 연속된 metal도 필요하다. 
        

![Untitled](resource/ct_1/Untitled_3.png)

1. CT imaging 시뮬레이션 하기
    
    MATLAB의 도움을 아주 많이 받았다! 최고!
    
    ![**Left** : 시뮬레이션으로 생성한 distorted image
    **Right** : difference image = `raw image + metal - distorted image`](resource/ct_1/Untitled_4.png)
    
    **Left** : 시뮬레이션으로 생성한 distorted image
    **Right** : difference image = `raw image + metal - distorted image`
    

---


<br><br><br><br><br>

medical segmentation에서 자주 사용되는 **U-net**을 사용하기로 했다.

문제는 임플란트같은 금속물은 탈부착이 불가능해서 `distorted image` 와 `ground truth image` 쌍을 만들 수 없다는 점이였다.



주 과제는 구강 내 **금속물로 인한 artifact를 제거하는 것**이다 `참조 →`

금속 외에 뼈(치아)에 의해서도 **beam hardening**이 발생하고, 이렇게 생성된 artifact를 **streak artifact**라고도 한다.

---

medical segmentation에서 자주 사용되는 **U-net**을 사용하기로 했다.

문제는 임플란트같은 금속물은 탈부착이 불가능해서 `distorted image` 와 `ground truth image` 쌍을 만들 수 없다는 점이였다.

![실제 distorted image](resource/ct_1/Untitled.png)

실제 distorted image

연구실 자체적으로 CT imaging에 관련된 physics를 열심히 공부한 결과는…

→ **금속물이 없는 dental CT 영상에 가상으로 금속물을 추가해서 artifact를 생성하자** 이다.

→ 불행 중 다행은 **fan beam CT**라 근방의 image에서 **간섭하는 artifact**는 무시해도 된다는 점이다.

---

### Data generation

1. raw image 준비하기
    
    metal은 가상으로 추가할 수 있으니 raw image가 필요하다.
    
    기업에 간곡히 요청하면 (구걸 x) metal free image를 받을 수 있다.
    
    - 혹시 몰라서 상악~하악까지의 모든 slice를 요청하였다.
    - 또 혹시 몰라서, 다양한 해상도의 image를 요청하였다.

![Untitled](resource/ct_1/Untitled_1.png)

1. metal 추가하기
    
    원하는 부위(치아)에 원하는 metal을 추가할 수 있어야 한다.
    
    - 치아 선택 기능 → gui 제작하여 해결
    - metal 생성 → 선택된 치아의 edge를 찾아 생성
        - edge → 크라운
        - fill → 임플란트
        - edge 내에 작은 metal → 땜빵
        
        ![강한 artifact를 위해 연속된 metal도 필요하다. ](resource/ct_1/Untitled_2.png)
        
        강한 artifact를 위해 연속된 metal도 필요하다. 
        

![Untitled](resource/ct_1/Untitled_3.png)

1. CT imaging 시뮬레이션 하기
    
    MATLAB의 도움을 아주 많이 받았다! 최고!
    
    ![**Left** : 시뮬레이션으로 생성한 distorted image
    **Right** : difference image = `raw image + metal - distorted image`](resource/ct_1/Untitled_4.png)
    
    **Left** : 시뮬레이션으로 생성한 distorted image
    **Right** : difference image = `raw image + metal - distorted image`
    

---

### Method & Result

metal free image를 추정하기 위해 2가지 학습 방법이 있다.

- `Direct` : distorted image → metal free image
- `Residual` : distorted image → difference image
    - metal free image = distorted image - difference image

![Residual 학습 방법](resource/ct_1/Untitled_5.png)

Residual 학습 방법

여러 hyper parameter, data 구성, dynamic range등을 조절하며 두 학습 방법을 비교한 결과

일관적으로 `Residual` 학습 방법이 더 좋은 성능을 보였다 → **criteria = NMSE**

결국, model의 출력은 아니지만, 최종적으로 아래의 reconstructed image를 얻었다. 짜잔~

![distorted image](resource/ct_1/Untitled_6.png)
![reconstructed image](resource/ct_1/Untitled_7.png)

엄청 잘되는 것 같아 보이지만, metal이 너무 가까이 붙어있거나 많으면 artifact가 남는다.

- 치아 사이 입천장 즈음에 dark band는 ROI가 아니라 괜찮지만…
- metal과 metal 사이에 위치한 치아는 형태를 알아보기 어려워지기도 한다.

![여러 방법을 시도하고 NMSE 기준 TOP-4를 선정.
양치를 너무 안하면 그래도 artifact가 많이 남는다…](resource/ct_1/Untitled_8.png)

여러 방법을 시도하고 NMSE 기준 TOP-4를 선정.
양치를 너무 안하면 그래도 artifact가 많이 남는다…