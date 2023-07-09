---
title: CT - Compressive Senging
date: 2023-01-15 19:45:10
tags: [CT, compressed sensing, compressive sensing, sparse sensing]
categories: CT
mathjax: true 
---

### Contents
- [Compressed sensing](#compressed-sensing)
  - [Reconstruction](#reconstruction)
  - [MAR + CS](#mar--cs)

이전 포스트에서는 dental CT에서 치아 내 금속 조형물이 발생시키는 artifact와 그 artifact를 deep learning으로 제거하는 내용을 다루었다.
이번 포스트에서는 적은 방사선으로 CT를 촬영할 때 image가 어떻게 변화하는지, 그리고 이를 어떻게 보정하는지에 대한 내용을 다룬다.

---

## Compressed sensing

CT는 대부분 큰 사고를 경험했거나 건강검진 시 촬영하게 되는데, 항상 촬영 전에 방사선사들은 환자들이 이전에 CT를 촬영한 전적이 있는지 물어보거나 조사한다.
이는 CT 촬영이 환자의 채네에 방사선을 피폭시키기 때문이며, 최근에 방사선 촬영을 한 환자라면 채내에 누적된 방사선량이 일정 수치를 넘지 않게 조사량을 조절해야 한다.

![Sv](resource/ct_cs/sv.jpg)

여러 각도에서 X-ray를 촬영하는 CT는 조사량을 조절하기 위해 회전각 $\Delta\theta$를 조절한다. 당연하게도, 회전각과 획득하는 정보량은 반비례하고, 회전각이 커질수록 영상의 품질이 저하된다. 아래는 metal이 없는 raw image에 회전각을 변화시키며 image를 재구성하는 경우 영상 품질을 비교한 것이다.
그림 중 상단의 회전각이 $5\degree$인 경우는 총 72개의 sinogram을, $10\degree$인 경우에는 36개의 sinogram을 사용하여 영상을 재구성하게 된다.

![CS](resource/ct_cs/cs.png)

### Reconstruction

![Untitled](resource/ct_cs/CS_recon.png)

MAR과 마찬가지로 `distorted image`와 `difference image`를 쌍으로 학습하였다. 생각보다 image reconstruction이 잘 되었는데, 해석하자면 이 문제는 전체 sinogram중 missing된 부분을 근방의 획득한 sinogram을 사용하여 interpolation하는 문제이고, deep learning이 풀기 쉬운 문제였지 않나.. 하고 있다.

### MAR + CS

CS가 생각보다 쉬운 문제였다면, 자연스럽게 이전 과제인 MAR과의 결합을 생각해보게 된다. MAR과 CS가 image에 끼치는 영향이 서로 독립이라면 아무런 무리가 없겠지만, 단순하게 생각해도 metal artifact가 sinogram의 interpolation을 심하게 방해할 것 같다. 실제로, 아래와 같이 매우 심한 distortion이 발생했다.

![Untitled](resource/ct_cs/CS_MAR.png)

아래는 여태까지의 방법과 마찬가지로 image를 reconstruction한 것이다.
회전각이 $5\degree$일 때 corrected image가 reference image와 어느 정도 일치하는 반면, $10\degree$인 경우에는 치아끼리 붙어버리거나 모양이 변형되는 모습을 볼 수 있다. 육안으로 봐도 후자의 distorted image는 영상 복원이 불가능할 정도로 원형의 모습을 잃어버린 듯 하다.

![Untitled](resource/ct_cs/CS_MAR_recon.png)

---

본 포스트를 마지막으로 CT 프로젝트를 진행했던 내용을 모두 다루었다.
이 후에는 cardiac MRI에 대한 연구를 진행하며 compressed sensing, transfer learning 등을 연구하였는데, 이들은 논문 게재 혹은 proceeding으로 출판된 것들이 있어 포스팅 하지 않을 예정이다.
