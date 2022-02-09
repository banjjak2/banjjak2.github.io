---
title: "[Kotlin] object 키워드" 
date: 2022-02-09 11:36:17 +0900
categories: [Kotlin]
tags: [Object, 키워드, 코틀린, Kotlin]
---

# 📚 [Kotlin] object 키워드

- object 키워드에는 `Object expression(객체 표현식)`과 `Object declaration(객체 선언)` 이라는 개념이 존재한다.
- 일부 클래스를 약간 수정한 개체를 만들어야할 때 새로운 하위 클래스를 만드는 것 보다 낫다고 판단될 때 사용할 수 있다.

## 🟢 Object expressions (객체 표현식)
- `객체 표현식`은 이름이 정해지지 않은 `익명 클래스`를 만들 때 사용한다.

    - `익명 클래스`란 이름이 정해지지 않았다는 뜻으로 클래스명이 없다.

    - `익명 클래스`이므로 클래스를 선언함과 동시에 객체가 생성된다.

        ```kotlin
        val helloWorld = object {
            val hello = "Hello"
            val world = "World"

            override fun toString() = "$hello $world"
        }

        fun main() {
            println(helloWorld)
        }
        ```

    - 위 코드에서 `object` 다음의 `{`부터 `}`까지가 익명 클래스가 된다.

- 이런 `익명 클래스`들은 **일회성**으로 사용할 필요가 있을 때 유용하다.

- 처음부터 익명 클래스를 구현하거나 기존 클래스나 인터페이스를 상속받아 구현할 수 있다.
    - 위 코드는 처음부터 익명 클래스를 구현한 코드이다.

- 익명 클래스의 인스턴스는 표현식이기 때문에 `익명 객체`라고도 한다.

### 🟢 처음부터 익명 클래스 구현

- 처음부터 객체 표현식을 이용한 익명 클래스를 구현하게 되면 타입이 선언되어있지 않기 때문에 `Any`로 취급된다.

    1. 따라서 아래와 같은 코드는 결국 `Any`클래스에서 `test`라는 메소드가 없기 때문에 에러가 발생하게 된다. 

        ```kotlin
        val helloWorld = object {
            fun test() = println("test")
        }

        fun main() {
            helloWorld.test() // 에러발생
        }
        ```

    2. 사용자가 정의한 메서드나 속성에 접근하려면 아래와 같이 `private`나 `local`에서 사용이 가능하다.

        ```kotlin
        private val helloWorld2 = object {
            fun test() = println("helloWorld2")
        }

        fun main() {
            val helloWorld3 = object {
                fun test() = println("helloWorld3")
            }

            println(helloWorld2.test())  // 접근가능
            println(helloWorld3.test())  // 접근가능
        }
        ```

- **왜 2번은 사용자가 정의한 메서드나 속성에 접근이 가능한지는 잘 모르겠다..**

### 🟢 상속을 이용한 익명 클래스 구현

- 기존에 구현된 클래스나 인터페이스를 상속받아 익명 클래스로 구현할 수 있다. 

    ```kotlin
    window.addMouseListener(object : MouseAdapter() {
        override fun mouseClicked(e: MouseEvent) { /*...*/ }

        override fun mouseEntered(e: MouseEvent) { /*...*/ }
    })
    ```

--- 

## 🟢 Object declarations (객체 선언)

- 싱글톤(Singleton) 패턴을 사용할 때 이용할 수 있는 기능중의 하나이다.
    - 싱글톤이란 프로그램이 실행 도중에 `생성되는 객체는 단 하나` 라는  점

    ```kotlin
    object DataProviderManager {
        fun registerDataProvider(provider: DataProvider) {
            // ...
        }

        val allDataProviders: Collection<DataProvider>
            get() = // ...
    }
    ```

- 다음과 같이 사용할 수 있다.

    ```kotlin
    DataProviderManager.registerDataProvider(...)
    ```

---

출처
- https://kotlinlang.org/docs/object-declarations.html#object-expressions