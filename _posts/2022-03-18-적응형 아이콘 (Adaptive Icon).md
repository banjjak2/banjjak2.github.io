---
title: "적응형 아이콘 (Adaptive Icon)" 
date: 2022-03-18 02:01:28 +0900
categories: [Android]
tags: [Android]
---

# 적응형 아이콘 (Adaptive Icon)

- API 26 (Android 8.0) 이상부터 사용가능함
- 전경(foreground), 배경(background), OEM Mask 레이어들로 아이콘을 구성함
  - 전경 : 실제 아이콘
  - 배경 : 아이콘 뒤의 배경 이미지
  - OEM Mask : 핸드폰 제작사마다 아이콘을 어떻게 표현해줄지에 대한 레이어

    ![적응형아이콘](https://user-images.githubusercontent.com/29175138/158853237-30e43c36-8714-4976-bdb6-26ef6b6662e3.png)

-  리소스 매니저에서 적응형 아이콘을 추가해준 후 프로젝트에 drawable-anydpi-v26 리소스 폴더를 생성한다.
  - -v24, drawable에 있는 적응형 아이콘에 필요한 것들을 drawable-anydpi-v26 에 옮겨 쉽게 관리할 수 있다.

- 적응형 아이콘을 사용해야하는 이유
  - 해상도에 따라 유연하게 변경되므로 기존처럼 따로 해상도별로 아이콘을 제작하지 않아도 된다.
    - 그러나 API 26 이상에서만 가능하다는 점.
    
- 아이콘은 108dp의 크기로 제작해야하며 safe zone 내에 표현할 실제 아이콘(전경)이 위치해야 함
  - safe zone 이란 위에서 언급한 OEM Mask가 제작사별로 다른데 저 safe zone 안에 위치하기만 하면 어느 기기에서든 아이콘을 의미있게 표현할 수 있음.
  - safe zone 은 위 사진에서 아이콘 주변에 동그라미 부분이 됨
    