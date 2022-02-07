---
layout: post
title: "[Multiple View Geometry in Computer Vision] 0. Foreword, Preface"
date: 2022-02-07
categories: [전공서적 리딩, Multiple View Geometry in Computer Vision]
tags: [multiple view geometry in computer vision, computer vision, cv, foreword, preface]
---

Computer vision, SLAM 분야를 공부해보고자 교수님께 여쭤봤더니 _Multiple View Geometry in Computer Vision_ 을 무조건 읽어보라고 추천해 주셨다. 이번 포스팅부터 전공 책을 읽고 나름대로 요약한 내용을 정리해볼까 한다.


## Foreword
- 컴퓨터의 시각 처리 기술(Making a computer see)은 AI 학계에서 여름 학생들의 프로젝트 정도로 가볍게 여겨졌지만, 아직까지 해결되지 못한 난제임.
- CV 분야는 수학, 컴퓨터 과학과 강한 관련이 있고, 물리, 심리학, 신경학과 약한 연관이 있음.
- 생물학적으로 시각 처리 작업은 많이 알려져 있지 않아 CV 분야에서 모방하기 어렵지만, 이러한 요소를 무시한 시도는 성공적이지 못함.
- CV 분야의 성취는 실무적 측면과 이론적 측면으로 나눌 수 있음.
  -  실무적 측면의 사례로, 차량을 일반 도로나 거친 노면에서 안내하는 기술이 있는데, 이는 매우 정교한 실시간 3D dynamic scene 분석 기술을 필요로 함.
  - 이론적 측면에서 __geometric Computer Vision__ 라는 분야가 성취됨.
    > e.g. 시점에 따른 물체의 appearance를 물체의 모양과 카메라 parameters의 함수로 묘사.
- 이러한 종류의 작업이 컴퓨터의 시각 처리 기술을 위한 옳은 방향이 맞는지에 대한 질문은 독자에게 남김.

## Preface
- CV 중 __geometry of multiple views__ 분야는 매우 빠르게 발전.
  - 2장의 이미지가 주어지고, 다른 정보가 없을 때, 이미지 사이의 matching을 계산하고 이미지를 생선한 카메라의 3D 상 위치를 계산.
  - 3장 이미지가 주어지고, 다른 정보가 없을 때, 유사하게 이미지의 점, 선들의 매칭을 계산하고, 카메라의 위치 계산.
  - Stereo rig의 epipolar geometry와 trinocular rig의 trifoal geometry를 보정 없이 계산.
  - 자연 장면에서의 연속적인 이미지로부터 카메라 내부 보정 계산.
- 이런 알고리즘의 독특한 점은 __uncalibrated__ (카메라 parameter 등을 알 필요가 없음). 알고리즘은 view로부터 상이 맺힌 점, 선들에 의한 contraints와 각 이미지에 대응되는 3차원 상 카메라의 위치에 기반.
  > e.g. Stereo rig의 epipolar geometry를 계산할 때는 카메라 보정을 제외한 7개의 parameters만 필요. 과거의 알고리즘은 각각의 카메라가 보정이 필요했고, 이러한 보정은 11개의 parameters가 요구됨.
- 적절한 geometry representation을 사용한 uncalibrated 접근은

  1. 알고리즘 각 단계에 필요한 parameters를 명시.
  2. 더 간단한 알고리즘.
  3. 3차원 상 점과 같은 entities의 ambiguity를 복구함.
  4. 이론적 측면에서, 종종 보정이 불가능한 상황(카메라가 이동, 카메라 내부 parameters가 변화 등)에도 적용 가능.

    이라는 이점이 있음. 

- 이러한 성취를 거두기 위해

  1. Over-determined system에서 error 최소화 기법.
  2. RANSAC과 같은 robust한 estimation 사용.

    와 같은 수학적 접근을 사용.

- __Reconstruction 문제__ 역시 우리가 풀었다고 주장할만한 수준까지 발전.
  - 이미지 점들의 대응으로부터 multifocal 텐서 추정.
  - 이러한 텐서로부터 camera matrices 추출, 2, 3, 4 views로부터 subsequent projective 재구성.
- 유의미한 성취가 있지만, 아직 더 연구할 문제가 존재.
  -  일반적인 reconstruction 문제 해결을 위한 bundle adjustment 적용.
  -  카메라 행렬에 대한 최소한의 가정 하에 metric 재구성.
  -  multifocal tensor 관계를 이용하여 이미지 sequence의 대응 관계 자동 감지, 이상값, false matches 제거.


### 목차 설명
- __Part 0 Background:__ 2-space, 3-space에서 projective geometry, 이러한 geometry의 표현, 조작, 추정 방법, geometry와 CV의 다양한 목표 사이의 관계 설명.
- __Part 1 Single view geometry:__ 3-space에서 perspective projection을 모델링하는 다양한 카메라를 정의, 보정된 물체에 전통적인 기술을 사용한 추정, 소실점과 소실선에서의 카메라 보정에 대해 학습.
- __Part 2 Two view geometry:__ 두 카메라의 epipolar geometry, 이미지 점 대응으로부터 projective reconstruction, projective ambiguity 해결 방법, 최적 triangulation 등에 대해 학습.
- __Part 3 Three view geometry:__ 세 카메라의 trifocal geometry, two views에서 three views로 점, 선 대응, 점과 선의 대응에서 geometry 계산, 카메라 행렬 등 학습.
- __Part 4 N-views:__ three view에서 four views로 확장, N-views에 적용 가능한 Tomasi, Kanade factorization 알고리즘 학습, 앞선 장을 확장한 공통적인 내용을 다룸.
  > e.g. multi-linear view contraints, auto-calibration, ambiguous solutions