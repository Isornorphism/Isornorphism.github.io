---
layout: post
title: "[Lie group] SO(3), SE(3) group jacobian"
date: 2022-09-19
last_modified_at: 2022-09-19
categories: [SLAM]
tags: [lie group, SO(3), SE(3), jacobian]
math: true
published: false
---


# 2. Related works

## 2.1 라이다-관성 SLAM
주위 환경을 점군 지도로 정밀하게 파악 가능하지만 측정 주파수가 낮은 라이다와 측정 주파수가 높지만 시간에 따라 drift가 발생하는 IMU는 상호 보완적인 관계에 놓여있다. 이 둘을 융합한 라이다-관성 SLAM 분야는 활발히 발전되었다. LIOM [#1]은 라이다, IMU 센서를 강결합(tightly coupled)으로 융합한 최초의 시도로서 IMU-preintegration 측정값을 바탕으로 라이다 포즈와 점군 지도를 보정하였다. LIO-SAM [#2]은 factor graph 기반으로 구현되었다. Real-time으로 구동할 수 있고 정확도가 높으며 안정적이라는 이유로 대표적으로 사용되는 라이다-관성 SLAM이다. 로봇의 포즈가 특정 threshold보다 크게 변화하였을 시 keyframe을 노드로 형성하고, 노드는 IMU-preintegration factor, lidar odometry factor, GPS factor, loop closure factor로 연결된다. Lidar odometry factor는 LOAM [#3], LeGO-LOAM [#4]와 유사하게 edge feature와 plane feature를 특징점으로 추출하여 최적화를 수행한다. GPS factor는 초기값을 제공하는 역할만 수행할 뿐 강결합되어있지는 않다. 그 외에 LINS [#5]와 Fast-LIO [#6]은 iterated EKF(Extended Kalman Filter)를 제시하여 라이다-관성 SLAM을 해결하려고 시도하였다.

[#1] Tightly Coupled 3D Lidar Inertial Odometry and Mapping (2019)
[#2] LIO-SAM
[#3] LOAM
[#4] LeGO-LOAM
[#5] LINS
[#6] Fast-LIO

## 2.2 GNSS 센서 융합

### 2.2.1 카메라-관성-GNSS 센서 융합
카메라-관성 센서 융합 SLAM은 주위 환경으로부터 직접 거리를 측정하는 방식을 사용하지 않기 때문에 drift 현상이 필연적으로 발생한다. 이를 해결하는 방법으로 VIO(Visual-Inertial odometry)에 절대 위치 정보를 제공하는 GNSS를 융합하는 시도가 많이 진행되어왔다. VINS-Fusion [#1]은 카메라, IMU 등 local 센서와 GPS, magnetometer 등 global 센서를 융합하는 프레임워크를 제시하였다. [#2]는 스테레오 VINS와 GNSS PPP(Precise Point Positioning)을 결합하여 측위 정확도를 크게 향상시킴과 동시에 drift 현상을 감소시켰다. GVINS [#3]는 GNSS raw 측정값을 사용해 의사 거리(pseudorange)와 Doppler shift를 모델링하여 카메라-관성 센서 시스템과 factor graph 프레임워크에서 융합하였다. GNSS 신호가 자주 끊기는 환경이거나 사용 가능한 위성의 개수가 적을 때에도 VIO의 drift 현상을 효과적으로 억제하는 성과를 보였다. IC-GVINS [#4]는 FGO 프레임워크에서 keyframe 기반 추정을 사용하고 IMU 이상치 제거 시 GNSS 측정값을 활용하여, GVINS보다 외란에 대해 강건하고 우수한 real-time 성능을 보였다.


[#1] VINS-Fusion
[#2] Semi-tightly coupled integration of multi-GNSS PPP and S-VINS for precise positioning in GNSS-challenged envirnemnts
[#3] GVINS, Tightly Coupled Integration of GNSS and Vision SLAM Using 10-DoF Optimization on Manifold (2021)
[#4] IC-GVINS: A Robust~~


### 2.2.2 라이다-관성-GNSS 센서 융합

LIO-SAM [#0]과 [#1]과 같이 대부분의 라이다-관성-GNSS 센서 융합은 GNSS 수신기에서 자체적으로 global 위치를 계산한 다음 라이다-관성 시스템의 로봇 위치를 보정하는 2단계로 구성된다. 이러한 약결합(loosely-coupled) 기반 센서 융합은 일반적으로 GNSS와 연결된 위성 및 상태에 따라 불안정적이고 정확도가 떨어진다는 한계가 있다. GVINS처럼 GNSS raw 측정값을 활용하여 강결합 시스템을 구축한 사례로는 [#2]가 존재하지만, localization에 초점을 맞추었고 mapping까지 수행하는 알고리즘은 제시하지 못하였다. 또한 NLOS(Non-line-of-sight) 신호를 탐지하여 후처리하는 과정이 없어 의사 거리 산출에 큰 불확실성을 내포하고 있다. 반면 NLOS 탐지 과정을 거치는 대부분의 연구는 ([#ff]-[#ff]) GNSS raw 측정값을 기반으로 강결합되어있지 않거나 마찬가지로 mapping 방법을 보이지 못하였다. 이에 본 연구는 GNSS raw 측정값에서 NLOS 신호를 필터링한 라이다-관성-GNSS 강결합 기반 localization & mapping 방법을 최초로 제안한다.

[#0] LIO-SAM
[#1] Graph-based adaptive fusion of GNSS and VIO under intermittent GNSS-degraded environment
[#2] Factor Graph Fusion of Raw GNSS Sensing with IMU and Lidar for Precise Robot Localization without a Base Station


## 2.3 GNSS NLOS Detection
도시 협곡(Urban canyon)이나 숲과 같이 하늘이 트여있지 않은 환경에서는 GNSS 신호가 구조물에 막히거나 반사되는 다중 경로 효과(Multipath effects)가 발생한다. 구조물과 다중 반사를 거쳐 GNSS 수신기에 도달하는 NLOS 신호는 의사 거리(Pseudorange)를 산출할 때 정확도를 현저히 감소시키는 원인으로 작용한다. [#1]에 의하면 NLOS 신호를 제거하고 LOS 신호만을 사용하였을 때 3D position error가 원본 신호를 사용하였을 때보다 2.025m에서 1.400m로 정확도가 30.86% 향상되었다고 밝혔다. [#2]에서는 NLOS 신호에 의한 의사 거리 오차가 고도각과 수신기로부터 주변 구조물까지 거리에 따라 최대 10m까지 모델링될 수 있다고 주장하였다. 따라서 어떤 위성에서 온 신호가 NLOS인지 탐지하고, 적절한 처리를 해주는 것은 GNSS의 정확도를 향상시키는데 있어 매우 중요한 과제이다.

[#3]은 통계적인 방법을 이용하여 NLOS 신호를 검출하는 방법을 제시하였으나, PVT를 제공하는 VTL(Vector Tracking Loop) 기반 수신기를 필요로 한다는 한계점이 있다. [#4]는 NLOS를 따로 분류하지 않고 의사 거리를 산출한 다음 outlier에 강건한 비선형 최소 자승법을 적용하는 방법을 제시하였으나, 위성 수가 적은 환경 등 데이터 상태에 정확도가 크게 좌우된다는 한계점이 있다. 통계적 방법을 기반으로 한 솔루션은 명확한 한계가 존재하므로, 본 연구에서는 multi sensor를 기반으로 한 직접적인 경로 추정 모델을 사용하였다.

[#5]-[#7]는 다양한 센서(fish-eye 카메라, 적외선 카메라)로 하늘을 촬영한 다음 구조물에 의해 차폐된 영역에 위성이 있는 경우 그 위성으로부터 받은 신호를 NLOS로 분류한다. 직접적인 경로 추정으로 간단하게 NLOS 신호를 분류한다는 장점이 있지만, 오로지 GNSS 필터링을 위해 일반적인 자율 주행 센서 시스템에 어울리지 않는 센서가 추가로 요구된다는 점에서 경제성이 떨어진다. 라이다는 자율 주행 시스템에 대표적으로 활용되는 센서이자 주변 구조물의 형상을 점군 지도로 쉽게 파악할 수 있으므로, 직접적인 경로 추정 방식으로 NLOS 신호를 효과적으로 분류할 수 있다. 따라서 본 연구는 라이다 점군 지도를 기반으로 한 NLOS 탐지 방식을 채택하였다.

[#8]은 라이다에 의해 생성된 점군 지도를 GNSS Skyplot에 투영하여 위성 신호가 구조물에 차폐되는지 여부를 통해 NLOS를 분류하였다. NLOS 신호는 제외하고 LOS 신호를 기반으로 가중 최소 제곱법 방법을 수행하였다. [#9], [#10] 역시 수신기와 위성을 이은 선이 점군 지도와 겹치는지 여부를 판단하여 NLOS를 분류하고, 의사 거리 계산 시 NLOS에 작은 가중치를 곱하여 함께 반영하였다. 더 나아가 [#11]은 구조물 상 반사점을 탐색하여 NLOS 신호에 의한 의사 거리를 보정하는 방법을 제시하였다. 그러나 숲과 같이 불규칙한 형태의 구조물이 많은 환경에서는 위성 신호가 난반사할 가능성이 크기 때문에 [#11]의 방법론은 본 연구에 적절하지 못하다고 판단하였다. 이에 본 연구는 [#9], [#10]과 유사하게 NLOS와 LOS 신호 간에 서로 다른 가중치를 부여하여 의사 거리를 추정하는 방법을 사용하였다.


[#1] How NLOS signals affect GNSS relative positioning
[#2] Analysis and modeling GPS NLOS effect in highly urbanized area
[#3] Probabilistic approach to detect and correct GNSS NLOS signals using an augmented state vector in the extended Kalman flter
[#4] Towards Robust Graphical Models for GNSS-Based Localization in Urban Environments
[#5] Tightly coupled GNSS/INS integration via factor graph and aided by fish-eye camera
[#6] Real-time GNSS NLOS detection and correction aided by sky-pointing camera and 3D LiDAR
[#7] High-accuracy GPS and GLONASS positioning by multipath mitigation using omnidirectional infrared camera
[#8] GNSS NLOS Exclusion Based on Dynamic Object Detection Using LiDAR Point Cloud
[#9] Improved-UWB/LiDAR-SLAM Tightly Coupled Positioning System with NLOS Identification Using a LiDAR Point Cloud in GNSS-Denied Environments
[#10] SLAM with 3Dimensional-GNSS
[#11] 3D LiDAR Aided GNSS NLOS Mitigation in Urban Canyons


** 용어 통일 필요
융합 - fusion / 결합 - coupled
의사 거리 - pseudorange
탐지 - detection
drift (그대로 씀)
NLOS 신호라고 쓸까요? NLOS라고 쓸까요? (LOS도)
IMU pre-integration
최소 제곱법? 최소 자승법?
NLOS(Non-line-of-sight)처럼 약어 설명은 최초 용어 등장 때 한번 언급해주도록 바꿔야 하지 않을까 싶습니다
Dopler shift? frquency shift?