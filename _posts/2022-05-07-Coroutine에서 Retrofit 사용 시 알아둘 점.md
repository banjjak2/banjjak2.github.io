---
title: "Coroutine에서 Retrofit 사용 시 알아둘 점" 
date: 2022-05-07 17:18:13 +0900
categories: [Android, TIP]
tags: [Android, Coroutine, Retrofit]
---

이전에 안드로이드 앱을 구현하면서 Coroutine에서 Retrofit을 사용한 적이 있었다. 그때 당시에는 인터넷 예제에서는 대부분 아래와 같이 구현되어 있었다.

```kotlin
interface XXXService {
    @GET("api/YYYY")
    suspend fun getData(): Call<ZZZZ> // 또는 Response<ZZZZ>
}
```

그런데, 레트로핏 2.6.0 이상부터는 [코틀린 전용 확장 함수](https://github.com/square/retrofit/blob/master/retrofit/src/main/java/retrofit2/KotlinExtensions.kt)를 지원함으로써 `Call` 또는 `Response`를 해줄 필요가 없다고 한다. 그 이유를 추측해보자면 코루틴을 사용하기 전에는 스레드에서 데이터를 UI로 보낼 때에는 콜백함수를 이용해서 메인 스레드에 전달 해주어야 하기 때문에 `Call`로 랩핑을 해주어야 하지만, 코루틴은 콜백함수 없이도 `suspend` 키워드가 있다면 `Dispatchers.Main`에서도 UI에서 실행할 수 있기 때문인 것 같다.<br>
만약 응답의 전체 데이터를 원한다면 `Result<ZZZZ>` 형식으로 사용 가능하다고 한다. <br>

또한, 여러 인터넷에 있는 예제들을 보면 아래와 같이 구현된 것들도 가끔가다 볼 수 있었다.

```kotlin
suspend fun getResult() {
    viewModelScope.launch {
        withContext(Dispatchers.IO) {
            val data = XXXService.getData()
            // ...
        }
    }
}
```
이또한 잘못 사용한 예로 볼 수 있다. 위에서 언급한 것과 비슷하지만, `viewModelScope`의 내부를 살펴보면 디스패처가 `Dispatchers.Main.immediate`로 되어있는 것을 알 수 있다.

```kotlin
public val ViewModel.viewModelScope: CoroutineScope
// ...
CloseableCoroutineScope(SupervisorJob() + Dispatchers.Main.immediate)
// ...
```

즉, 어차피 디스패처가 Main으로 설정되어 있고, 위에서 언급한 것처럼 레트로핏은 코루틴 관련 확장 함수들(suspend)을 지원하도록 구현되었으므로 굳이 코루틴에서 `withContext`를 실행해줄 필요가 없다. 

아래는 구글 코드랩에 작성된 글을 가져왔다.

![image](https://user-images.githubusercontent.com/29175138/167247754-d75cf76b-8a32-4801-9072-157b5370e76e.png)

---

### 내 생각
코루틴에서 `Dispatchers.IO`는 네트워크 통신 등에서 사용될 수 있다고 하는데 아마 레트로핏을 이용하지 않고 `HttpUrlConnection`이나 다른 동기적으로 네트워크 통신을 하는 라이브러리를 이용해 직접 구현할 때를 의미하는 것 같기도 하다. 이 부분에 대해서는 좀 더 생각을 해봐야할 것 같다.

---

### [출처]
- https://developer.android.com/codelabs/kotlin-coroutines#8
