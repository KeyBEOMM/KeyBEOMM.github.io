---
title: "Vision Blackboard Refactoring"
date: 2025-05-09 20:25:00 +0900
categories: [Project, Refactoring]
tags: [refactoring, mediapipe, calibration, depth estimation, vision]
description: 2023년에 진행했던 Vision Blackboard 프로젝트를 리팩토링 해보자!
media_subpath:
---
## 전 프로젝트
stereo 비디오에 대해 비디오 속 손의 추적하여 특정 depth이상의 값을 시각화하는 프로젝트. camera에 대해 Calibration 후 비디오의 일정 프레임의 image에 대해 undistortion 하였다. 각 이미지를 python과 Mediapipe로 2D 랜드마크를 따고, 직접 삼각측량으로 깊이를 계산해서 3D 포인트를 생성하고 이를 Spline 보간으로 시각화했던 프로젝트. 하지만, 병목 현상으로 실시간성이 떨어지는 문제 생김

## refactoring 방법

### 1.핵심 연산 C++로 이전:

- **Mediapipe Hand Landmark Detection**:
    가장 큰 병목 지점일 가능성이 높아. Mediapipe C++ API를 사용해서 이 부분을 C++로 구현

- **삼각측량 (Triangulation)**:
    OpenCV는 C++ 라이브러리가 기본이고, cv::triangulatePoints 함수가 이미 최적화되어있음. 파이썬에서 cv2.triangulatePoints를 썼다면 C++에서도 거의 동일하게 사용할 수 있고, 데이터 복사 오버헤드를 줄일 수 있을듯.

## 구현

### 1. **Mediapipe Hand Landmark Detection Refactoring**:

> [MediaPipe 사이트](https://ai.google.dev/edge/mediapipe/solutions/guide)
{: .prompt-info }

hand tracking 및 landmark 따는 방법 찾아봤지만 mediapipe가 제일 나을 듯해 업데이트 된 mediapipe 기술 logic 파악을 해보자.

> 2023년 3월 1일부터 아래에 나열된 MediaPipe 기존 솔루션에 대한 지원이 종료되었습니다. 다른 모든 MediaPipe 기존 솔루션은 새 MediaPipe 솔루션으로 업그레이드됩니다. 
{: .prompt-info }

새로운 Solution으로 업그레이드되었다고 한다..!