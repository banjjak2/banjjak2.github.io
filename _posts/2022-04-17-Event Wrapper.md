---
title: "Event Wrapper" 
date: 2022-04-17 16:40:12 +0900
categories: [Android]
tags: [Android]
---

# Event Wrapper

MVVM 아키텍처에서 View가 ViewModel을 참조하기 위한 방법 중 하나는 LiveData를 이용하는 것이다. ViewModel은 UI 데이터를 가지고 있기 때문에 화면 회전과 같은 변경 사항이 발생하더라도 View의 UI 데이터가 사라지지 않는다.
<br>
그러나, Snackbar나 Toast, 화면 이동(navigation)과 같은 것들은 한 번만 사용되어야 한다. 
<br>
<details>
<summary>예제</summary>

<div markdown="1">

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var button: Button
    private lateinit var viewModel: TestViewModel
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        viewModel = ViewModelProvider(this).get(TestViewModel::class.java)
        button = findViewById(R.id.button)
        button.setOnClickListener {
            viewModel.error("ERROR MESSAGE")
        }

        viewModel.errorMessage.observe(this) {
            Toast.makeText(this, it, Toast.LENGTH_SHORT).show()
        }
    }
}

class TestViewModel : ViewModel() {
    private val _errorMessage = MutableLiveData<String>()
    val errorMessage: LiveData<String> = _errorMessage

    fun error(msg: String) {
        _errorMessage.value = msg
    }
}
```

</div>
</details>

![Apr-17-2022 16-52-48](https://user-images.githubusercontent.com/29175138/163705748-8eda8589-70f4-4b19-87a8-053c0873fef2.gif)

위 코드는 버튼 클릭 시 뷰모델에 메시지를 전달하고, 메인 엑티비티에서 메시지를 Observe 하고 있다가 상태가 변경되면 토스트 메시지를 띄우는 간단한 코드이다. 그러나 자세히 보면 화면 회전시에도 한 번 더 Toast 메시지가 발생하게 되는 것을 확인할 수 있는데, 이는 ViewModel과 Activity의 Lifecycle을 알면 이해하기 쉽다. 
<br>
간단히 말해서, 화면이 회전되면 Activity가 종료되었다가 다시 시작되고, LiveData를 Observe 하고있을 경우 최신 데이터를 가져오기 때문에 다시 Toast 메시지가 출력된다. 

<br>
그러나, Toast 메시지나 화면 이동같은 이벤트들은 화면이 회전되더라도 한 번만 발생해야하기 때문에 화면에 변경사항이 발생했을 경우 문제가 될 수 있다.

---

한 번만 발생해야하는 이벤트들은 Event Wrapper를 이용해 관리해야한다. 

<details>
<summary>예제</summary>

<div markdown="1">

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var button: Button
    private lateinit var viewModel: TestViewModel
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        viewModel = ViewModelProvider(this).get(TestViewModel::class.java)
        button = findViewById(R.id.button)
        button.setOnClickListener {
            viewModel.error("ERROR MESSAGE")
        }

        viewModel.errorMessage.observe(this, EventObserver {
            Toast.makeText(this, it, Toast.LENGTH_SHORT).show()
        })
    }
}

class TestViewModel : ViewModel() {
    private val _errorMessage = MutableLiveData<Event<String>>()
    val errorMessage: LiveData<Event<String>> = _errorMessage

    fun error(msg: String) {
        _errorMessage.value = Event(msg)
    }
}

open class Event<out T>(private val content: T) {
    var hasBeenHandled = false

    fun getContentIfNotHandled(): T? {
        return if (hasBeenHandled) {
            null
        } else {
            hasBeenHandled = true
            content
        }
    }

    fun peekContent(): T = content
}

class EventObserver<T>(private val onEventUnhandledContent: (T) -> Unit) : Observer<Event<T>> {
    override fun onChanged(event: Event<T>?) {
        event?.getContentIfNotHandled()?.let {
            onEventUnhandledContent(it)
        }
    }
}
```

</div>
</details>

![Apr-17-2022 17-18-24](https://user-images.githubusercontent.com/29175138/163706541-a3f164a6-3ee2-4568-b5ac-fb00aa96e481.gif)

자세한 설명이나 예제는 아래 사이트를 참고<br>
https://medium.com/androiddevelopers/livedata-with-snackbar-navigation-and-other-events-the-singleliveevent-case-ac2622673150

---
