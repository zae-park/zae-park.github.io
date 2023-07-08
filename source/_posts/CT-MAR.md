---
title: CT - Metal Artifact Reduction
date: 2023-01-02 22:47:09
tags: [CT, MAR, U-net, deep learning, image reconstruction]
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
또, 만약 촬영하려는 부위에 금속물이 있다면 그 근방에 **streak artifact**라는 흰색-검은색 줄무늬가 발생하게 되어 image를 왜곡시킨다. 게다가, 이 금속물의 개수가 2개 이상이 되면, 각 금속물 사이에 streak artifact가 심해지면서 금속물 근방 뿐 아니라 사이에 위치한 image를 전부 왜곡시킨다.
예를 들어, 큰 수술 이력이 있어 뼈에 금속 보형물이 있다거나, 임플란트나 브릿지와 같은 치과 시술을 받은 경우가 문제가 된다.

![Distorted Image](resource/ct_mar/distorted_image.png)

**그렇다면 streak artifact는 어떻게 발생하게 되는 걸까?**

지난 포스트 [CT imaging 기초](resource/ct-imaging-basic)에서 CT는 `조직에 의해 감쇠된 energy`를 image로 그린다고 했다. 이는 streak artifact의 흰색-검은색 줄무늬가 매우 높은 값, 또는 매우 낮은 값을 갖는다는 것을 뜻한다.
X-ray는 진행 경로 상에서 object를 만나면 투과, 굴절, 흡수 등의 상호작용을 하게된다. 만약, X-ray가 진행 경로 중 금속을 만나 일부 입자가 굴절되면, 그 경로에 있던 detector가 수신하는 energy는 그만큼 작아지게 되며 굴절된 energy는 근방의 다른 detector가 수신하게 되어 더 큰 energy를 수신하게 된다.
이를 detector 입장에서 해석하면, 전자의 경우는 경로상에 HU이 큰 어떤 object에 의해 energy의 큰 감쇠가 있었으니 흰색 줄무늬를, 후자의 경우는 경로상에 energy 감쇠가 적었으니 검정 줄무늬를 그리게 된다.
> HU이란 물질마다 갖는 고유한 선형 감쇠계수 $\mu$를 물과 상대적인 수치로 나타낸 값이다.


### Data
![Distorted Image](resource/ct_mar/dataset.png)

프로젝트의 목표는 `metal artifact 줄이기`이다. 이를 supervised learning으로 풀기 위해서는 위와 같은 distorted image와 쌍이 되는 artifact-free image가 필요하다.
하지만, 치아에 있는 금속물들은 탈부착이 불가능할 뿐 아니라, 만약 탈부착하더라도 같은 위치의 단층 촬영 image를 얻기는 불가능하다. 그래서 아래와 같은 순서로 이 data를 직접 만들기로 했다!

- metal이 없는 건강한 치아의 단층 촬영 CT image, raw image를 얻는다.
- raw image의 치아 영역에 크라운, 브릿지 등의 금속 조형물을 추가한다.
  - 크라운은 1개의 치아를 선택하고, 이들의 edge를 선택한다.
  - 브릿지는 n개의 연속된 치아를 선택하고, 이들의 edge를 선택한다.
  - 임플란트는 1개의 치아를 선택하고 fill 한다.
  - 필링(땜빵)은 치아 영역 내부에 사각형, 원형 등의 작은 조각을 추가한다.
- 추가한 조형물들의 HU를 결정하고 Radon transform을 통해 sinogram을 생성한다.
  - 레진, 금 등 조형물에 사용된 물질에 따라 HU가 다른 점을 주의한다.
- FBP을 통해 sinogram을 distorted image로 재구성한다.
  - 기존 image의 dynamic range가 유지되지 않을 수 있으니 주의한다.

![GUI](resource/ct_mar/data_gen_ui.png)

위의 과정은 그리 복잡하지 않지만, 모든 image를 작업하기에는 무리가 있다. 특히, 원하는 치아에 원하는 조형물을 추가하려면 GUI가 필수적이다. 다행히 MATLAB에서 간단하게 GUI를 만들 수 있는 기능을 제공해서 이를 활용했다. 이름은 guide 였던 것 같은데... 이게 없었으면 MFC로 GUI를 만들 생각을 하니 좀 끔찍하다.


### Train Artifact

이제 metal image와 distorted image 쌍을 확보한 상태이다. 이제 이 dataset을 사용하여 신경망을 학습해야 하는데, 두가지 방법이 있었다.

- `input` Distorted image & `output` Artifact-free image
- `input` Distorted image & `output` Difference image

> 일부 논문에서는 후자의 방법을 residual learning이라고도 한다.
> 아래의 image에서 왼쪽은 distorted image를, 오른쪽은 difference image를 보여준다. difference image는 metal image에서 distorted image를 뺀 것이다.
> 
> ![result](resource/ct_mar/residual.png)


두 학습방법 중 실험적으로 후자를 선택하였다.
뒤늦게나마 해석하자면, 신경망에게 metal의 개수나 크기, HU에 따라 천차만별인 metal artifact를 추론하고 이를 제거하는 역할보다는, 비교적 변화 폭이 작은 raw image와 metal 정도를 추론하는 것이 더 쉽고 안정적이지 않았을까 생각해본다.


![result](resource/ct_mar/test.png)

협업하는 기업에게 신경망의 weight를 전달하기 위해 위와 같이 여러 ablation study를 진행했다. CT image에 관련된 voxel size, resolution, dynamic range 등을 조절해보기도 하고, 신경망의 hyper parameter 및 seed를 조절하며 weight 초기화에 따른 성능 변화를 검토하기도 했다. 결국 4개의 weight가 후보군에 올랐고, 실제 clinical data에 이를 적용해보며 최종 weight를 선정하게 되었다.
위 그림에서 오른쪽이 각각의 weight를 통해 재구성한 영상들 중 일부 ROI를 자세하게 나타낸 것이다. 어렴풋한 기억으로는, 치아 가운데 부비동 부분의 artifact는 중요하지 않아 W3이 선택되었던 것 같다.

---

본 포스트에서 다룬 내용은 치아에 금속 조형물이 있는 경우 발생하는 artifact와 이를 줄이기 위한 deep learning project이다. 사용한 model, 정확한 parameter는 기억도 나지 않고, 공개하기도 부담되니 생략하였다. 이후 포스트에서는 적은 조사량으로 CT image를 재구성하는 compressive(sparse) sensing에 대한 내용을 다루려고 한다.
