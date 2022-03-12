---
title: "옵저버 패턴 (Observer Pattern)" 
date: 2022-03-12 14:53:45 +0900
categories: [Design Pattern]
tags: ["Design Pattern"]
---

# 옵저버 패턴 (Observer Pattern)

- 옵저버 패턴은 관찰자(Observer)들이 어떤 객체(Subject)를 구독함으로써 객체(Subject)에 변화가 발생했을 때 관찰자(Observer)들에게 상태 변화나 이벤트가 발생했음을 알리는 디자인 패턴이다.

    ![854px-Observer svg](https://user-images.githubusercontent.com/29175138/158004705-62c32536-d613-490e-a180-dca6b6da77b8.png)


- 관찰을 받을 대상(Subject)에는 관찰자(Observer)들이 여러개 등록될 수 있다.

- 옵저버는 인터페이스로 구성되며, 해당 인터페이스를 상속받아 concrete 옵저버 객체를 만들어 관찰대상을 구독하는 형태를 취한다.

## 장점

- 런타임 시에 객체를 구독하거나 취소할 수 있기 때문에 유연하다.
- 개방/폐쇄 원칙(OCP)을 지키므로 관찰대상의 코드 변경없이 새로운 옵저버를 등록할 수 있다.
- 관찰대상과 관찰자 객체들 사이의 결합도(의존성)를 낮춰준다.
    - 인터페이스로 통신하기 때문

## 단점

- 제때 옵저버를 해제해주지 않으면 메모리 누수가 발생할 수 있다.
    - 구독이 된 상태에서 해제하지 않고 옵저버가 종료될 경우

- 비동기 상태에서 옵저버들이 원하는 순서대로 상태변화를 얻어올 수 없다.
    - 원하는 순서대로 옵저버를 등록할 수는 있지만 비동기(예를 들어 쓰레드 등)에서는 언제 등록이 되고 해제가 될지 모르기 때문

## 예시

- 이번 예시는 간단히 코틀린으로 구현했다.
- 아래 예시는 다음과 같은 구조로 되어있다.
    1. 메인 객체에서 관찰대상이 될 `Notice` 클래스와 관찰자가 될 `User`클래스를 인스턴스화 한다.
    2. 관찰대상에게 관찰자들을 등록한다. (registerObserver 메소드 이용)
    3. 메인 객체에서 관찰대상에게 메세지를 전달한다.
    4. 관찰대상은 메시지가 들어왔으므로 등록된 옵저버들에게 메시지를 전달한다.

<details>
<summary>옵저버 인터페이스</summary>

<div markdown="1">

```kotlin
package observer

interface Observer {
    fun updateNotice(message: String)
}
```

</div>
</details>

<details>
<summary>옵저버를 상속받는 유저 클래스 (User)</summary>

<div markdown="1">

```kotlin
package observer

class User(private val username: String) : Observer {
    override fun updateNotice(message: String) {
        println("$username : $message")
    }
}
```

</div>
</details>

<details>
<summary>관찰대상 클래스 (Notice)</summary>

<div markdown="1">

```kotlin
package observer

class Notice {
    private val observerList = mutableListOf<Observer>()

    fun registerObserver(observer: Observer) {
        observerList.add(observer)
    }

    fun unregisterObserver(observer: Observer) {
        observerList.remove(observer)
    }

    fun noticeAllUser(message: String) {
        for (observer in observerList) {
            observer.updateNotice(message)
        }
    }
}
```

</div>
</details>

<details>
<summary>메인 클래스</summary>

<div markdown="1">

```kotlin
package observer

fun main() {
    val notice = Notice()
    val user1 = User("스폰지밥")
    val user2 = User("뚱이")
    val user3 = User("징징이")
    val user4 = User("핑핑이")

    notice.registerObserver(user1)
    notice.registerObserver(user2)
    notice.registerObserver(user3)
    notice.registerObserver(user4)

    notice.noticeAllUser("게살버거 먹고싶다")
    notice.unregisterObserver(user3)
    println()
    notice.noticeAllUser("배부르다")
}
```

</div>
</details>

<details>
<summary>결과</summary>

<div markdown="1">

```text
스폰지밥 : 게살버거 먹고싶다
뚱이 : 게살버거 먹고싶다
징징이 : 게살버거 먹고싶다
핑핑이 : 게살버거 먹고싶다

스폰지밥 : 배부르다
뚱이 : 배부르다
핑핑이 : 배부르다
```

</div>
</details>

---

- **참고**

    - https://ko.wikipedia.org/wiki/옵서버_패턴
    - https://en.wikipedia.org/wiki/Observer_pattern
