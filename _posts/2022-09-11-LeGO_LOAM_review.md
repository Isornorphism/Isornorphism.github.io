---
layout: post
title: "[LeGO-LOAM] LeGO-LOAM: Lightweight and Ground-Optimized Lidar Odometry and Mapping on Variable Terrain 논문 리뷰"
date: 2022-09-13
last_modified_at: 2022-09-13
categories: [SLAM]
tags: [lego-loam, loam, lidar, odometry, mapping, ground, segmentation, Levenberg, Marquardt, LM, loop closure]
math: true
published: true
---

LeGO-LOAM은 ground를 이용하여 LOAM 알고리즘을 경량화하고 성능과 속도를 향상시켰다. LOAM과 비교하여 개선된 점을 위주로 논문을 정리해보았다.

>*Original paper*  
* [LeGO-LOAM: Lightweight and Ground-Optimized Lidar Odometry and Mapping on Variable Terrain](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=8594299)

>*다음 자료를 참고하였습니다.*
* [LOAM, Lego-LOAM](https://dreambreaker-ds.tistory.com/entry/LOAM-Lego-LOAM)
* [SLAM 온라인 스터디 SLAM DUNK 2020 \| 정진용님 발표](https://www.youtube.com/watch?v=snPzNmcbCCQ)
* [LeGO-LOAM Line by Line](https://limhyungtae.github.io/2022-03-27-LeGO-LOAM-Line-by-Line-1.-Introduction/)

<br>

## 1. Introduction

LOAM은 smoothness를 기준으로 edge, planar feature를 분류한 다음, 점들 사이의 scan-matching을 이용하여 두 scan 간의 대응 관계(correspondence)를 탐색한다. 실시간으로 알고리즘이 구동될 수 있게끔 추정 문제를 2개의 개별 알고리즘으로 나누어 해결한다. 하나(LiDAR Odometry)는 높은 주파수와 낮은 정확도로 센서의 속도를 추정하고, 다른 하나(LiDAR Mapping)는 낮은 주파수와 높은 정확도로 모션 추정을 수행한다. 이 두 추정값은 Transform Integration 단계에서 융합되어 높은 정확도와 낮은 drift를 보이게 된다.

본 논문은 UGV(Unmanned Ground Vehicle)에서 구동할 수 있는 LOAM을 제안하는데 목적을 두었다. LOAM은 모든 points에서 smoothness를 계산해야하므로 계산 리소스가 제한된 상황에 부적절하다. LiDAR가 지면에 가깝게 장착되는 UGV 특성 상 잔디 등의 요인으로 신뢰할 수 없는 edge feature가 추출될 수 있다. 이러한 요인으로 인해 부정확한 registration과 큰 drift가 발생할 우려가 있다.

이를 해결하고자 논문에서는 몇 가지 contribution points를 제안한다.

* Pointcloud segmentation을 거친 후 ground와 신뢰할 수 없는 points를 제거
* Pose estimation 단계에서 optimization을 2 단계로 수행하여 계산 복잡도를 감소
* Drift를 보정하기 위해 loop closure를 수행

<br>

## 2. Notation and System Block Diagram

![system_block_diagram](/assets/img/LeGO_LOAM_review/block_diagram.png){: width="80%"}
_출처: 원 논문_

* Right subcription $t$ : time of Sweeps
* $P_t$ : time $t$에 얻어진 pointcloud, $P_t = \\{p_1, p_2, \cdots, p_n\\}$
* $r_i$ : i번째 점의 depth
* $\mathbb{F}_e^t, \mathbb{F}_p^t$ : time $t$에서 얻은 모든 sub-images에서의 edge, planar features 집합

<br>

## 3. Segmentation

### 3.1 Ground Removal

본 논문은 LiDAR가 지면과 평행하게 부착되어 있다고 가정한다. VLP-16을 예시로 들자면 한 각도에서 vectical한 방향으로 16개의 laser가 발사되어 거리가 측정된다. 이때 아래(ground) 방향으로 발사되는 8개 channel에 대해 Ground Removal 단계가 수행된다.

![ground_removal](/assets/img/LeGO_LOAM_review/ground_removal.png){: width="90%"}
_출처: [LeGO-LOAM Line by Line - 2. ImageProjection (2)](https://limhyungtae.github.io/2022-03-27-LeGO-LOAM-Line-by-Line-2.-ImageProjection-(2)/)_

Vectical하게 인접한 두 픽셀에 대해 LiDAR 좌표계에서의 $(x, y, z)$를 계산한다. 다음으로 그림과 같이 지면과 이루는 각도를

$$\theta = \arctan \frac{\Delta z}{\sqrt{\Delta x^2 + \Delta y^2}}$$

로 계산한다. 그 각도가 10도보다 낮은 점을 ground로 판별한다.

> **[Note]** Ground segmentation 성능  
> 논문에서 제시하고 있는 방법은 효율적이지 않다고 하는데 그 원인은 다음과 같다.
> * 한 픽셀 내에 여러 points가 찍히는 경우 실제 ground points가 ground가 판별되지 않음
> * 기울기가 불규칙한 지역에서 성능이 상당히 저하됨
> 
> 따라서 다른 용도로 ground segmentation을 적용할 필요가 있을 경우, 다른 알고리즘을 사용하는 것이 좋을 듯 하다.

### 3.2 Cloud Segmentation

지면을 분리한 pointclound에 image-based segmentation을 적용한다. 본 논문은 _"Fast Range Image-Based Segmentation of Sparse 3D Laser Scans for Online Operation"_ 을 이용하여 segmentation을 수행하였다고 밝혔다.

>*Reference paper*  
* [Fast Range Image-Based Segmentation of Sparse 3D Laser Scans for Online Operation](http://www.ipb.uni-bonn.de/pdfs/bogoslavskyi16iros.pdf)


![image_segmentation](/assets/img/LeGO_LOAM_review/image_segmentation.png){: width="80%"}
_출처: "Fast Range Image-Based Segmentation of Sparse 3D Laser Scans for Online Operation"_

$\beta$를 laser 빔(line $OA$)과 인접한 두 픽셀을 이은 선(line $AB$)과의 각도로 정의한다. 이는 기하학적으로

$$\beta = \arctan \frac{d_2 \sin{\alpha}}{d_1 - d_2 \cos{\alpha}}$$

로 계산 가능하다. 이때 $d_1, d_2$는 각각 점 A, B의 depth를 의미한다. $\beta$가 특정 threshold보다 작은 경우, 즉 오른쪽 그림에서 빨간색으로 표시한 경우, 경계면이 급격하게 변화하므로 두 점은 다른 물체에 포함되어 있다고 간주할 수 있다. 이러한 heuristic한 방법을 이용하여 BFS 기반으로 image segmentation & labeling을 수행할 수 있다.

수가 충분히 많지 않는(30개 이하) cluster는 robustness를 위해 유효한 점에서 제외한다.

<br>

## 4. LiDAR Odometry

![lidar_odometry_diagram](/assets/img/LeGO_LOAM_review/lidar_odometry_diagram.png){: width="80%"}
_출처: 원 논문_

Feature Extraction 단계는 LOAM과 동일하여 생략한다. LiDAR Odometry 단계 역시 유사하지만, 몇 가지 차이점이 존재한다.

### 4.1 Label matching

Segmented cluster에서는 edge feature($\mathbb{F}_e$)의 correspondence만 탐색하고, ground cluster에서는 planar feature($\mathbb{F}_p$)의 correspondence만 탐색한다. 즉, feature extraction 단계에서 planar feature는 ground에서만, edge feature는 segmented cluster에서만 뽑으면 되므로, 계산 시간을 단축할 수 있다.

### 4.2 Two-step L-M optimization

Planar feature를 이용하여 $[t_z, {\theta} _ {roll}, {\theta} _ {pitch}]$를 optimize한 후 edge feature를 이용하여 $[t_x, t_y, {\theta} _ {yaw}]$를 optimize한다. 각각 과정에서 optimize하지 않는 parameter는 상수로 취급한다. 이를 통해 계산 시간을 35% 감소시키면서 유사한 정확도를 얻을 수 있다고 한다.

<br>

## 5. LiDAR Mapping

기본 아이디어는 LOAM과 동일하지만, pointcloud map을 저장할 때 feature set $\\{ \mathbb{F} _ {e}, \mathbb{F} _ {p} \\}$을 저장한다는 차이가 있다. $M_t = \\{\\{\mathbb{F} _ {e}^1, \mathbb{F} _ {p}^1\\}, \cdots, \\{\mathbb{F} _ {e}^t, \mathbb{F} _ {p}^t\\}\\}$라 정의할 때, $M_{t-1}$에서 $\bar{Q}_{t-1}$을 얻는 방법을 논문에서는 2 가지 제시하였다.

* 현재 pose를 기준으로 저장된 feature 중 주변 100m 이내에 있는 모든 pose들의 feature들을 불러온 다음, 모든 feature들을 각 pose로 transform하여 얻는다.
* LeGO-LOAM을 pose-graph SLAM으로 통합하여 생각한다. 즉, 로봇의 pose는 graph node로, feature set은 node의 measurement로 모델링한다.

그 후 LOAM과 마찬가지로 L-M optimization을 수행하여 transform matrix를 얻는다. 

추가로 논문에서는 loop closure를 수행하여 추가적인 contraint를 얻으면 drift를 줄일 수 있을 것이라 한다.