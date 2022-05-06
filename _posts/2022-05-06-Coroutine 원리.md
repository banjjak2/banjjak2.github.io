---
title: "Coroutine 원리" 
date: 2022-05-06 15:18:24 +0900
categories: [Kotlin, Coroutine]
tags: [Kotlin, Coroutine]
---

최종 수정 날짜 : 2022-05-06 16:58:38 +0900

# Coroutine 원리

코루틴이 어떻게 작동되는가에 대해 공부한 내용을 정리해보았다.<br>

[이전 글](https://banjjak2.github.io/posts/Coroutine/)을 참고해보면, 코루틴은 처음 생성될 때 사용한 스레드가 아닌 다른 스레드에서도 실행될 수 있다고 했다. 그게 가능한 이유는 아래 gif를 참고해보자.

![SuspensionProcess](https://user-images.githubusercontent.com/29175138/167078371-c5aa8db6-a7bb-435d-94b7-5220846d547b.gif)

스레드에서 코루틴이 suspend 상태가 되면 해당 스레드는 다른 resume이 가능한 상태인 코루틴을 가지고 와서 실행한다. 그리고 suspend된 코루틴이 다시 resume 가능한 상태가 되면 다시 실행한다. 단, suspend된 코루틴이 다시 resume될 때 동일한 스레드에서 실행되지 않을 수 있다. <br>
스레드는 다른 스레드를 실행할 때 `context switching`이 발생하게 되므로 오버헤드가 발생하지만, 코루틴에서는 사용하는 스레드 자체에서 실행가능한 다른 코루틴을 찾아 실행하기 때문에 스레드를 `switch`하는 작업을 하지 않아 오버헤드가 발생하지 않는다. 다른 `Dispatcher`로 변경하게 된다면 `context switching`이 발생할 수 있는데, `withContext`를 이용해 `Dispatcher`를 변경하게 되면 `context switching`을 최대한 방지한다고 한다.<br>

그럼 suspend와 resume이 어떤 방식으로 작동하는지에 대한 gif를 참고해보자.

![1_U24_ZyMJKI_c2qMspCXxZw](https://user-images.githubusercontent.com/29175138/167080868-a598f82e-f944-484c-a790-09ec60437dde.gif)

코루틴이 suspend될 때마다 함수와 변수를 추적하는데 사용하는 스택 프레임에 해당 코루틴에 대한 정보를 저장한다. 그리고 다시 resume될 때 해당 스택 프레임에서 코루틴에 대한 정보를 가지고와 다시 진행하게 된다. 메인 스레드에 있는 모든 코루틴이 suspend 상태가 되면 메인 스레드는 다른 작업을 할 수 있게 되어 UI를 업데이트하거나 사용자와 상호작용할 수 있게 된다. 

---

## Dispatcher

자주 사용하는 디스패처의 종류는 아래와 같다.
1. Dispatchers.Main
    - UI와 상호작용해야할 때
    - suspend 함수를 호출할 때
    - UI 관련 함수를 호출할 때
    - LiveData를 업데이트할 때

2. Dispatchers.IO
    - 디스크 또는 네트워크 IO를 처리할 때
    - 데이터베이스
    - 파일 입/출력
    - 네트워크

3. Dispatchers.Default
    - CPU를 많이 사용하는 작업을 처리할 때
    - List 정렬
    - JSON 파싱
    - DiffUtils

먼저 `Dispatchers.Default`에 대해 알아보자.

- `Dispatchers.Default`
    - `launch`나 `async`같은 코루틴 빌더에서 디스패처나 `ContinuationInterceptor`를 주지 않으면 기본적으로 적용된다.
    - 기본적으로 이 디스패처가 사용하는 최대 병렬 처리는 CPU 코어 수와 동일하지만 최소 2개 이상이다.

- `Dispatchers.Main`
    - UI와 관련된 작업을 진행할 때 사용할 수 있으며, 대체로 Single Thread로 작동된다.
    - 이 디스패처는 직접적으로 사용하거나 `MainScope` 팩토리를 이용해 사용할 수 있다.

- `Dispatchers.Main.immediate`
    - 안드로이드의 `viewModelScope`이나 `lifecycleScope`에서 사용하는 디스패처이다.
    - 메인 스레드에서 즉시 실행하기 때문에 성능을 최적화할 수 있다.

- `Dispatchers.IO`
    - 기본적으로 64개의 스레드 또는 코어 수 중 더 큰 수로 스레드 수가 제한된다.
    - 이 디스패처는 Default 디스패처와 스레드를 공유하므로 `withContext(Dispatchers.IO)`를 사용하면 다른 스레드로 전환되지 않고 동일한 스레드에서 실행이 계속된다.

---

### [출처]
- https://en.wikipedia.org/wiki/Coroutine
- https://kotlinlang.org/docs/coroutines-basics.html
- https://play.kotlinlang.org/hands-on/Introduction%20to%20Coroutines%20and%20Channels/01_Introduction
- https://medium.com/androiddevelopers/coroutines-on-android-part-i-getting-the-background-3e0e54d20bb
- https://medium.com/androiddevelopers/coroutines-on-android-part-ii-getting-started-3bff117176dd
