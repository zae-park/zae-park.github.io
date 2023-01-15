---
title: Dental CT - 2
date: 2023-01-15 19:45:10
tags: [CT]
categories: projects
mathjax: true 
---

# Dental CT - 2

<aside>
💡 **생각보다 일찍 끝난 프로젝트**

---

주 과제인 MAR이 너무 일찍 끝나버렸다.

사실 소속된 연구실은 MRI를 연구하고 있었는데, 마침 압축센싱을 연구하던 중이더라.

CT에도 압축센싱을 적용할 수 있다. 심지어 저선량 촬영이 가능하다는 장점이 명확하다.

</aside>

## Task: Compressed sensing

건강검진이나 부상의 이유로 병원에서 CT를 찍게 되면 병원에선 일정 기간 내에 CT를 촬영한 전적이 있는지 조사한다. 채네 축적된 방사선량이 너무 높으면 위험하기 때문이다. 하지만 CT image의 품질은 촬영시 조사하는 방사선량에 비례한다.

적은 조사선량으로도 고품질 CT image를 획득할 수 있으면 얼마나 좋을까?

---

CT는 photon과 detentor를 회전시키며 X-선을 다양한 각도에서 조사하고 imaging한다. 따라서 X-선의 조사량을 줄이면 저선량 촬영이 가능하고 이는 photon의 회전각 변화량을 키우면 된다.

$$
\frac{1}{resolution} \propto Dose \propto \Delta \theta
$$

아래는 $\Delta\theta$에 따른 영상품질을 나타낸 것이다.

![**3D reconstruction based on compressed-sensing (CS)-based framework by using a dental panoramic detector**](resource/ct_2/Untitled.png)

**3D reconstruction based on compressed-sensing (CS)-based framework by using a dental panoramic detector**

### Data generation

사실 CT image의 품질은 $\Delta\theta$ 뿐 아니라 object의 크기, object와 photon의 거리와도 관계가 있다. 그래서 phantom image를 기반으로 시뮬레이션을 좀 해보고 적당한 $\Delta\theta$를 찾아보았다.

![Untitled](resource/ct_2/Untitled_1.png)

![Untitled](resource/ct_2/Untitled_2.png)

윗줄이 object 크기가 작은 경우, 아랫줄이 object 크기가 큰 경우이다.
잘 보이지 않겠지만… object가 클수록 광원과의 거리가 가까워지고 artifact가 심해진다.
그래도 artifact가 영 심하지 않아서 적당히 해볼만 한 듯 하다.

![Untitled](resource/ct_2/Untitled_3.png)

![Untitled](resource/ct_2/Untitled_4.png)

실제 CT image는 object size가 phantom보다 훨씬 크기 때문에 훼손 정도도 심하다.
그래도 잘될거라 기대하며 실험해볼 수 밖에…

### Method & Result

![Untitled](resource/ct_2/Untitled_5.png)

MAR과 마찬가지로 U-net을 residual 방식으로 학습하였다.  $\Delta\theta$은 과감하게 $10\degree$이다.

![Untitled](resource/ct_2/Untitled_6.png)

오잉?? 생각보다 reconstruction이 너무 잘 되었다. 
꿈보다 해몽: CS는 결국 sinogram의 angle 축 resolution만 줄어든 것이니 interpolation 문제이다.

## Task: Compressed sensing + MAR

CS가 생각보다 잘먹힌다는 사실을 알았다. 심지어 $\Delta\theta = 10\degree$인데도.
그러면 MAR을 안 합쳐볼수가 없다.

![Untitled](resource/ct_2/Untitled_7.png)

distortion이 너무 심하다…

## Result

- DegInc 5
    
    ![Untitled](resource/ct_2/Untitled_8.png)
    
- DegInc 10
    
    ![Untitled](resource/ct_2/Untitled_9.png)
    

당연스럽게도, 광원의 회전각이 작을 때 distortion이 덜하고 reconstructed image도 좋다.

DegInc가 10일 때도 썩 봐줄만하다고 생각했는데… 

Metal artifact가 심한 경우에는 좀 위험한지 DegInc가 5일 때 까지만 허용하기로 했다.