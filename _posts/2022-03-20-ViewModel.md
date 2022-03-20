---
title: "ViewModel" 
date: 2022-03-20 15:36:43 +0900
categories: [Android]
tags: [Android]
---

## 작성중..

# ViewModel

- `ViewModel`은 View의 Lifecycle을 고려해 UI와 관련된 데이터들을 저장하고 관리하는 클래스이다.
- 따라서, 화면 전환과 같은 변경 사항(UI를 다시 그리는 등)이 발생할 경우에도 데이터를 유지시켜준다.
    - 화면 전환이나 멀티 윈도우 모드로 들어갈 경우 액티비티나 프래그먼트는 `onDestroy` 콜백을 실행한 후에 다시 UI를 그리를 작업을 진행하므로 기존에 있던 데이터가 사라진다. 

    ![예제](https://user-images.githubusercontent.com/29175138/159150228-8984d115-a7c9-4c86-aebc-740ac76edb9a.gif)

- `onSaveInstanceState()`를 사용해 UI 데이터를 저장하기 위해서는 데이터가 크지 않고, 직렬화 후 역직렬화를 할 수 있는 데이터만 사용해야 한다.
- UI의 목적은 데이터를 표시하거나 사용자의 이벤트에 반응하는 등을 처리하기 위함이므로 UI에 과도한 책임이 할당되면 클래스의 부피가 커지기 때문에 분리할 필요가 있다.
    - UI 로직에서 View의 데이터를 ViewModel에 보관하도록 책임을 할당해야 한다.
- ViewModel은 View, Lifecycle 또는 액티비티의 Context를 참조를 포함하는 클래스를 참조하면 안된다.

