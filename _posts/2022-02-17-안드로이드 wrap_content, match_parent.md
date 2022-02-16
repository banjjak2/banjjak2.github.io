---
title: "안드로이드 wrap_content, match_parent" 
date: 2022-02-17 00:11:17 +0900
categories: [Android]
tags: [Android]
---

# 안드로이드 wrap_content, match_parent

## 속성
- `layout_width`, `layout_height`에 `수치값(dp, px)`, `wrap_content`, `match_parent`값을 줄 수 있다.

## wrap_content
- `layout_width`, `layout_height`속성에 `wrap_content`가 설정되면 설정된 `View`의 높이와 넓이가 기본 `View`의 크기와 같다는 의미가 된다.
- 만약 `TextView`와 같은 View에서 글이 길어져 다음줄로 넘어가거나 글씨 크기가 커진다면 컨트롤의 크기 또한 같이 커진다.

## match_parent
- `layout_width`, `layout_height`속성에 `match_parent`가 설정되면 설정된 `View`의 높이와 넓이가 부모의 높이, 넓이와 같은 크기를 가진다는 것이 된다.
- 단, `ConstraintLayout`에서는 `match_parent` 값을 설정하면 안된다.
    - 이미 제약조건(Start_toStartOf, End_toEndOf)에 의해 길이가 정해졌기 때문
    - `match_parent`를 써야한다면 `0dp`로 입력해주어야 한다.
    - 대체로 `layout_width`에서 `match_parent`대신 `0dp`를 사용한다.

## 두 속성을 사용하는 이유
- 안드로이드 기기들의 크기가 모두 다양하기 때문에 호환성을 생각해 화면을 구성해야하기 때문
- 한 가지의 크기로만 구현하게 해두었다면 화면이 더 커지거나 작아졌을 경우 화면이 깨질 가능성이 있다.

---
