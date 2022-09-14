---
layout: post
title: "[Multiple View Geometry] 1. Introduction (1)"
date: 2022-03-04
last_modified_at: 2022-03-04
categories: [전공서적 리딩, Multiple View Geometry in Computer Vision]
tags: [multiple view geometry in computer vision, computer vision, cv, introduction]
math: true
published: false
---

## 1.1 Introduction
- __Projective transformation(사영 변환)__ 에서는 무엇이 보존될까?
  - 모양, 길이, 각도, 거리 등 많은 것이 보존되지 않음.
  - 하지만 straightness(직진성, 직선을 유지하는 성질)은 보존되는데, 이는 projective transformation의 general requirement임.

- Euclidean geometry → Projective geometry
  - Euclidean 2차원 공간 상 임의의 두 직선은 웬만하면 교차하지만, 두 직선이 평행한 경우 만나지 않음.
  - 일관성을 위해 평행한 두 직선이 무한대의 거리에 있는 __ideal point__ 에서 만난다는 개념을 도입.
  - Euclidean space에 ideal points를 더하여 projective space로 확장할 수 있음.
 
- 좌표
  - Euclidean 2차원 공간에서 $(x, y)$와 하나의 성분을 추가한 $(x, y, 1)$를 같은 점을 나타낸다고 정의.
  - $(kx, ky, k)$는 모두 동일한 점을 나타냄. 따라서 하나의 점은 __equivalance class__ 라 불리는 triplet의 집합으로 표현 가능함. 이를 __homogeneous coordinate__ 라 부름.
  - $(x, y, 0)$은 무한대에 있는 ideal points를 의미.
  - Homogeneous vector 표기를 이용하여 $\mathbb{R}^n$ Euclidean 공간을 $\mathbb{P}^n$ projective 공간으로 확장 가능.
  - 2차원 projective 공간에서 무한에 위치한 점들은 __line at infinity__ 라 부르는 한 직선에 위치하고, 3차원 projective 공간에서는 __plane at infinity__ 를 이루는 것으로 알려짐.

- 동질성(Homogeneity)
  - Euclidean geometry에서는 특별히 구분되는 점이 따로 없고 균질함(homogeneous). 특별한 좌표 프레임이 선호되지 않음.
  - 좌표계를 평행 이동하고, 회전하고, 늘리거나 수축시키는 등의 transformation이 가능함. 일반적으로 이를 __affine transformation__ 이라고 부름.
  - 이러한 변환에서도 무한에 있는 점들은 무한에 남아있음. 
  - Projective space 역시 homogeneous. 마지막 성분이 0인 무한대에 위치한 점도 좌표 프레임 선택에 의한 우연일 뿐, 다른 점들과 구분되지 않음. 
  - $\mathbb{R}^n$ Euclidean 공간에서 linear transformation을 정의하는 것처럼, $\mathbb{P}^n$ projective 공간에서 projective transformation을 정의할 수 있음. $(n+1)-vector$로 나타내지는 homogeneous coordinates는 같은 차원의 벡터로 mapping.
  - 이러한 변환은 행렬곱 $\mathbf{X'} = \mathrm{H_{(n+1)\times(n+1)}} \mathbf{X}$으로 표현됨.
  - CV 분야에서 projective space는 편리한 도구로 사용함. 대신 line and plane at infinity를 조심히 다룰 필요가 있음.


### 1.1.1 Affine and Euclidean geometry
- Affine geometry
  - Projective space에서는 평행이라는 개념이 존재하지 않음. 2차원 상 임의의 두 직선은 항상 homogeneous한 공간 상 한 점에서 만남.
  - 우리는 2차원 종이에 직선 하나를 그어, 이를 line at infinity라 정하고, 임의의 두 직선이 line at infinity 위에서 교차할 경우 이를 평행하다고 정의할 수 있음.
  > e.g. 지구 상 평평한 장소에서 사진을 찍을 때, 평행한 기차 철로는 수평선에서 만남. 이때 line at infinity는 수평선.
  - 이러한 관점에서, world plane과 그들의 상들은 projective plane과 특별한 distinguished line로 해석할 수 있음. 이를 __affine geometry__ 라고 함.
  - 한 공간의 distinguished line를 다른 공간의 distinguished line로 mapping하는 변환을 __affine transformation__ 이라고 함.
  - Line at infinity로 평행을 정의하면, 그와 관련된 다른 개념들도 정의 가능함.
  > e.g. 평행선 사이의 간격  
  > e.g. AB와 CD가 평행하고, AB와 CD의 길이가 같다면, AC와 BD도 평행.

- Euclidean geometry
  - 2차원 상에서 원은 affine geometry에서의 개념이 아님. Line at infinitiy를 보존하는 임의의 stretching 변환에 의해 타원으로 바뀔 수 있기 때문.
  - Euclidean geometry에서 원은 타원과 구별됨.
  - 대수적으로 원과 타원은 동일한 2차식으로 표현되지만, 임의의 두 타원은 4개의 점에서 만나는데 비해 임의의 두 원은 2개의 점에서 만나므로 기하학적으로 차별점이 존재함.
  - 사실 두 원의 교점의 해는 복소수 공간에서 2개 더 존재함.
    - Homogeneous coordinates $(x, y, w)$에서 중심이 $(a, b, 1)^T$인 원은 $(x-aw)^2 + (y-bw)^2 = r^2 w^2$로 나타낼 수 있음.
    - 점 $(1, \pm i, 0)^T$은 모든 원이 항상 지나므로, 항상 두 원의 교점. 마지막 성분이 0이므로 line at infinity에 놓여진 점임. (일반적으로, $x^2 + y^2 = 0, w=0$을 만족하는 점들)
  - 이러한 점들을 __circular points__ 라고 부름.
  - 