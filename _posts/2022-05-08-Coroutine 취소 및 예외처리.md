---
title: "Coroutine 취소 및 예외처리" 
date: 2022-05-08 16:37:28 +0900
categories: [Kotlin, Coroutine]
tags: [Kotlin, Coroutine]
---

### 일반적인 코루틴 예외처리 방법 (CoroutineExceptionHandler)
코루틴에서 예외를 처리하는 일반적인 방법은 `CoroutineScope`을 이용해 새 코루틴을 생성할 때 `CoroutineExceptionHandler`를 이용해 예외를 처리할 수 있다.

**코드**
```kotlin
import kotlinx.coroutines.*
import java.io.IOException
import kotlin.coroutines.cancellation.CancellationException

fun main() = runBlocking {
    val coroutineExceptionHandler = CoroutineExceptionHandler { _, throwable ->
        println("coroutineExceptionHandler -> $throwable")
    }

    val job = CoroutineScope(Dispatchers.Default + coroutineExceptionHandler).launch {
        val child1 = launch {
            try {
                throw IOException() // 예외 발생
                delay(3000)
                println("child1 coroutine")
            } catch (e: CancellationException) {
                println("child1 Exception -> ${e.message}")
            }
        }
    }.join()
    println("main")
}
```

**결과**
```text
coroutineExceptionHandler -> java.io.IOException
main
```

이렇게 함으로써 코루틴을 사용하는 도중에 예외가 발생했을 때 UI에 예외가 발생한 이유를 설명해주거나 다른 작업을 실행하도록 할 수 있다.

---

### 코루틴 취소 및 예외 발생

코루틴에서는 기본적으로 `구조적 동시성`때문에 부모 코루틴이 취소되거나 예외가 발생하면 부모 코루틴의 Scope에 있는 코루틴들은 모두 취소된다. (취소가 전파됨)

- 부모 코루틴 취소 시

    **코드**
    ```kotlin
    import kotlinx.coroutines.*
    import kotlin.coroutines.cancellation.CancellationException

    fun main() = runBlocking {
        val job = CoroutineScope(Dispatchers.Default).launch {
            val child1 = launch {
                try {
                    delay(3000)
                    println("child1 coroutine")
                } catch (e: CancellationException) {
                    println("child1 Exception -> ${e.message}")
                }
            }

            val child2 = launch {
                try {
                    val child3 = launch {
                        try {
                            delay(3000)
                            println("child 3 coroutine")
                        } catch (e: CancellationException) {
                            println("child3 Exception -> ${e.message}")
                        }
                    }

                    delay(3000)
                    println("child 2 coroutine")
                } catch (e: CancellationException) {
                    println("child2 Exception -> ${e.message}")
                }
            }

            println("parent coroutine")
        }
        delay(2000)
        job.cancel() // 부모 코루틴 취소

        println("main")
    }
    ```

    **결과**
    ```text
    parent coroutine
    child1 Exception -> StandaloneCoroutine was cancelled
    main
    child 2 Exception -> StandaloneCoroutine was cancelled
    ```

- 부모 코루틴 예외 발생 시 

    **코드**
    ```kotlin
    import kotlinx.coroutines.*
    import java.io.IOException
    import kotlin.coroutines.cancellation.CancellationException

    fun main() = runBlocking {
        val job = CoroutineScope(Dispatchers.Default).launch {
            val child1 = launch {
                try {
                    delay(3000)
                    println("child1 coroutine")
                } catch (e: CancellationException) {
                    println("child1 Exception -> ${e.message}")
                }
            }

            val child2 = launch {
                try {
                    val child3 = launch {
                        try {
                            delay(3000)
                            println("child 3 coroutine")
                        } catch (e: CancellationException) {
                            println("child3 Exception -> ${e.message}")
                        }
                    }

                    delay(3000)
                    println("child 2 coroutine")
                } catch (e: CancellationException) {
                    println("child2 Exception -> ${e.message}")
                }
            }

            throw IOException() // 예외 발생
            println("parent coroutine")
        }
        delay(2000)
        job.cancel()

        println("main")
    }
    ```

    **결과**
    ```text
    child1 Exception -> Parent job is Cancelling
    child3 Exception -> Parent job is Cancelling
    child2 Exception -> Parent job is Cancelling
    Exception in thread "DefaultDispatcher-worker-2" java.io.IOException
        at CehKt$main$1$job$1.invokeSuspend(ceh.kt:34)
        at kotlin.coroutines.jvm.internal.BaseContinuationImpl.resumeWith(ContinuationImpl.kt:33)
        at kotlinx.coroutines.DispatchedTask.run(DispatchedTask.kt:106)
        at kotlinx.coroutines.scheduling.CoroutineScheduler.runSafely(CoroutineScheduler.kt:570)
        at kotlinx.coroutines.scheduling.CoroutineScheduler$Worker.executeTask(CoroutineScheduler.kt:749)
        at kotlinx.coroutines.scheduling.CoroutineScheduler$Worker.runWorker(CoroutineScheduler.kt:677)
        at kotlinx.coroutines.scheduling.CoroutineScheduler$Worker.run(CoroutineScheduler.kt:664)
        Suppressed: kotlinx.coroutines.DiagnosticCoroutineContextException: [StandaloneCoroutine{Cancelling}@55321535, Dispatchers.Default]
    main
    ```

---

만약 자식 코루틴에서 취소가 발생하면 해당 코루틴만 취소되고 다른 코루틴에 취소가 전파되지 않는다.

- 자식 코루틴 취소 시

    **코드**
    ```kotlin
    import kotlinx.coroutines.*
    import kotlin.coroutines.cancellation.CancellationException

    fun main() = runBlocking {
        val job = CoroutineScope(Dispatchers.Default).launch {
            val child1 = launch {
                try {
                    delay(3000)
                    println("child1 coroutine")
                } catch (e: CancellationException) {
                    println("child1 Exception -> ${e.message}")
                }
            }
            child1.cancel() // 자식 코루틴 취소

            val child2 = launch {
                try {
                    val child3 = launch {
                        try {
                            delay(3000)
                            println("child 3 coroutine")
                        } catch (e: CancellationException) {
                            println("child3 Exception -> ${e.message}")
                        }
                    }

                    delay(3000)
                    println("child 2 coroutine")
                } catch (e: CancellationException) {
                    println("child2 Exception -> ${e.message}")
                }
            }

            delay(1000)
            println("parent coroutine")
        }.join()
        println("main")
    }
    ```

    **결과**
    ```text
    child1 Exception -> StandaloneCoroutine was cancelled
    parent coroutine
    child 2 coroutine
    child 3 coroutine
    main
    ```

---

하지만, 자식 코루틴에서 예외가 발생하면 그 예외는 최상위 부모 코루틴에 전달되어 모든 코루틴이 취소가 된다. 아래 예제는 `child3`에 예외를 발생시킨 것으로 먼저 부모 코루틴인 `child2`가 취소되고 다시 취소가 전파되어 부모 코루틴까지 취소가 됨으로써 `child1`도 취소가 되었다.

- 자식 코루틴 예외 발생 시

    **코드**
    ```kotlin
    import kotlinx.coroutines.*
    import java.io.IOException
    import kotlin.coroutines.cancellation.CancellationException

    fun main() = runBlocking {
        val job = CoroutineScope(Dispatchers.Default).launch {
            try {
                val child1 = launch {
                    try {
                        delay(3000)
                        println("child1 coroutine")
                    } catch (e: CancellationException) {
                        println("child1 Exception -> ${e.message}")
                    }
                }

                val child2 = launch {
                    try {
                        val child3 = launch {
                            try {
                                throw IOException() // 예외 발생
                                delay(3000)
                                println("child 3 coroutine")
                            } catch (e: CancellationException) {
                                println("child3 Exception -> ${e.message}")
                            }
                        }

                        delay(3000)
                        println("child 2 coroutine")
                    } catch (e: CancellationException) {
                        println("child2 Exception -> ${e.message}")
                    }
                }

                delay(1000)
                println("parent coroutine")
            } catch (e: CancellationException) {
                println("parent Exception -> ${e.message}")
            }
        }.join()
        println("main")
    }
    ```

    **결과**
    ```text
    child2 Exception -> StandaloneCoroutine is cancelling
    parent Exception -> StandaloneCoroutine is cancelling
    child1 Exception -> Parent job is Cancelling
    Exception in thread "DefaultDispatcher-worker-4" java.io.IOException
        at CehKt$main$1$job$1$child2$1$child3$1.invokeSuspend(ceh.kt:21)
        at kotlin.coroutines.jvm.internal.BaseContinuationImpl.resumeWith(ContinuationImpl.kt:33)
        at kotlinx.coroutines.DispatchedTask.run(DispatchedTask.kt:106)
        at kotlinx.coroutines.scheduling.CoroutineScheduler.runSafely(CoroutineScheduler.kt:570)
        at kotlinx.coroutines.scheduling.CoroutineScheduler$Worker.executeTask(CoroutineScheduler.kt:749)
        at kotlinx.coroutines.scheduling.CoroutineScheduler$Worker.runWorker(CoroutineScheduler.kt:677)
        at kotlinx.coroutines.scheduling.CoroutineScheduler$Worker.run(CoroutineScheduler.kt:664)
        Suppressed: kotlinx.coroutines.DiagnosticCoroutineContextException: [StandaloneCoroutine{Cancelling}@50074ae7, Dispatchers.Default]
    main
    ```

---

그렇다면 자식 코루틴에서 예외가 발생하더라도 부모에게 취소를 전파시키지 않는 방법은 무엇이 있을까?<br>
해답은 `SupervisorJob` 또는 `supervisorScope`을 이용하는 것이다. 이 두 개는 자식에서 예외가 발생하더라도 부모에게는 취소를 전파하지 않고 자식에게만 취소를 전파하게 된다. 먼저 `SupervisorJob`을 이용해 부모에게 취소를 전파시키지 않는 방법을 살펴보면 아래와 같다.

- SupervisorJob

    **코드**
    ```kotlin
    import kotlinx.coroutines.*
    import java.io.IOException
    import kotlin.coroutines.cancellation.CancellationException

    fun main() = runBlocking {
        val supervisorJob = SupervisorJob() // SupervisorJob 객체 생성
        val job = CoroutineScope(Dispatchers.Default).launch {
            try {
                val child1 = launch {
                    try {
                        delay(3000)
                        println("child1 coroutine")
                    } catch (e: CancellationException) {
                        println("child1 Exception -> ${e.message}")
                    }
                }

                val child2 = launch {
                    try {
                        val child3 = launch(supervisorJob) { // supervisorJob 객체 전달
                            try {
                                throw IOException() // 예외 발생
                                delay(3000)
                                println("child 3 coroutine")
                            } catch (e: CancellationException) {
                                println("child3 Exception -> ${e.message}")
                            }
                        }

                        delay(3000)
                        println("child 2 coroutine")
                    } catch (e: CancellationException) {
                        println("child2 Exception -> ${e.message}")
                    }
                }

                delay(1000)
                println("parent coroutine")
            } catch (e: CancellationException) {
                println("parent Exception -> ${e.message}")
            }
        }.join()
        println("main")
    }
    ```

    **결과**
    ```text
    Exception in thread "DefaultDispatcher-worker-1" java.io.IOException
        at CehKt$main$1$job$1$child2$1$child3$1.invokeSuspend(ceh.kt:22)
        at kotlin.coroutines.jvm.internal.BaseContinuationImpl.resumeWith(ContinuationImpl.kt:33)
        at kotlinx.coroutines.DispatchedTask.run(DispatchedTask.kt:106)
        at kotlinx.coroutines.scheduling.CoroutineScheduler.runSafely(CoroutineScheduler.kt:570)
        at kotlinx.coroutines.scheduling.CoroutineScheduler$Worker.executeTask(CoroutineScheduler.kt:749)
        at kotlinx.coroutines.scheduling.CoroutineScheduler$Worker.runWorker(CoroutineScheduler.kt:677)
        at kotlinx.coroutines.scheduling.CoroutineScheduler$Worker.run(CoroutineScheduler.kt:664)
        Suppressed: kotlinx.coroutines.DiagnosticCoroutineContextException: [StandaloneCoroutine{Cancelling}@3fb6433c, Dispatchers.Default]
    parent coroutine
    child1 coroutine
    child 2 coroutine
    main
    ```

    위 예제에서 본 것처럼, `child3`에서 예외가 발생하더라도 `launch`메소드에 `SupervisorJob`을 전달함으로써 부모에게 취소가 전파되지 않도록 한다. 또한, `child2`에서 예외가 발생할 경우 부모에게 예외를 전달하지 않지만 `child2`의 자식 코루틴인 `child3`에는 취소를 전파함으로써 같이 단방향으로 취소를 전파한다는 것을 알 수 있다.

    **코드**
    ```kotlin
    import kotlinx.coroutines.*
    import java.io.IOException
    import kotlin.coroutines.cancellation.CancellationException

    fun main() = runBlocking {
        val supervisorJob = SupervisorJob()
        [...]
                val child2 = launch(supervisorJob) {
                    try {
                        val child3 = launch {
                            try {
                                delay(3000)
                                println("child 3 coroutine")
                            } catch (e: CancellationException) {
                                println("child3 Exception -> ${e.message}")
                            }
                        }
                        
                        throw IOException() // 예외 발생

                        delay(3000)
                        println("child 2 coroutine")
                    } catch (e: CancellationException) {
                        println("child2 Exception -> ${e.message}")
                    }
                }

                delay(1000)
                println("parent coroutine")
            } catch (e: CancellationException) {
                println("parent Exception -> ${e.message}")
            }
        }.join()
        println("main")
    }
    ```

    **결과**
    ```text
    child3 Exception -> Parent job is Cancelling
    Exception in thread "DefaultDispatcher-worker-1" java.io.IOException
        at CehKt$main$1$job$1$child2$1.invokeSuspend(ceh.kt:29)
        at kotlin.coroutines.jvm.internal.BaseContinuationImpl.resumeWith(ContinuationImpl.kt:33)
        at kotlinx.coroutines.DispatchedTask.run(DispatchedTask.kt:106)
        at kotlinx.coroutines.scheduling.CoroutineScheduler.runSafely(CoroutineScheduler.kt:570)
        at kotlinx.coroutines.scheduling.CoroutineScheduler$Worker.executeTask(CoroutineScheduler.kt:749)
        at kotlinx.coroutines.scheduling.CoroutineScheduler$Worker.runWorker(CoroutineScheduler.kt:677)
        at kotlinx.coroutines.scheduling.CoroutineScheduler$Worker.run(CoroutineScheduler.kt:664)
        Suppressed: kotlinx.coroutines.DiagnosticCoroutineContextException: [StandaloneCoroutine{Cancelling}@1b633969, Dispatchers.Default]
    parent coroutine
    child1 coroutine
    main
    ```

이렇게 되면 모든 자식 코루틴에 `SupervisorJob`을 전달해주어야 하는데, 자식 코루틴 개수가 많다면 일일히 다 전달해주기에는 조금 비효율적일 수 있다. 그럴 땐 `supervisorScope`을 사용할 수 있다. 단, `supervisorScope`안에 있는 자식 코루틴에만 적용되며 자식 코루틴의 자식 코루틴에서는 적용되지 않는다.

- supervisorScope

    **코드**
    ```kotlin
    import kotlinx.coroutines.*
    import java.io.IOException
    import kotlin.coroutines.cancellation.CancellationException

    fun main() = runBlocking {
        val job = CoroutineScope(Dispatchers.Default).launch {
            supervisorScope {
                try {
                    val child1 = launch {
                        try {
                            throw IOException() // 예외 발생
                            delay(3000)
                            println("child1 coroutine")
                        } catch (e: CancellationException) {
                            println("child1 Exception -> ${e.message}")
                        }
                    }

                    val child2 = launch {
                        try {
                            val child3 = launch {
                                try {
                                    delay(3000)
                                    println("child 3 coroutine")
                                } catch (e: CancellationException) {
                                    println("child3 Exception -> ${e.message}")
                                }
                            }

                            delay(3000)
                            println("child 2 coroutine")
                        } catch (e: CancellationException) {
                            println("child2 Exception -> ${e.message}")
                        }
                    }

                    delay(1000)
                    println("parent coroutine")
                } catch (e: CancellationException) {
                    println("parent Exception -> ${e.message}")
                }
            }
        }.join()
        println("main")
    }
    ```

    **결과**
    ```text
    Exception in thread "DefaultDispatcher-worker-2" java.io.IOException
        at CehKt$main$1$job$1$1$child1$1.invokeSuspend(ceh.kt:11)
        at kotlin.coroutines.jvm.internal.BaseContinuationImpl.resumeWith(ContinuationImpl.kt:33)
        at kotlinx.coroutines.DispatchedTask.run(DispatchedTask.kt:106)
        at kotlinx.coroutines.scheduling.CoroutineScheduler.runSafely(CoroutineScheduler.kt:570)
        at kotlinx.coroutines.scheduling.CoroutineScheduler$Worker.executeTask(CoroutineScheduler.kt:749)
        at kotlinx.coroutines.scheduling.CoroutineScheduler$Worker.runWorker(CoroutineScheduler.kt:677)
        at kotlinx.coroutines.scheduling.CoroutineScheduler$Worker.run(CoroutineScheduler.kt:664)
        Suppressed: kotlinx.coroutines.DiagnosticCoroutineContextException: [StandaloneCoroutine{Cancelling}@104cb185, Dispatchers.Default]
    parent coroutine
    child 2 coroutine
    child 3 coroutine
    main
    ```

    `supervisorScope`는 내부적으로 부모 코루틴 scope의 `coroutineContext`를 상속받으며 이 컨텍스트의 `Job`을 `SupervisorJob`로 오버라이드하기 때문에 가능한 일이다. 아래는 `supervisorScope`에 대한 주석 내용이다.
    
    ![image](https://user-images.githubusercontent.com/29175138/167288523-d749311d-71b3-42ff-8748-2c8d28667995.png)

---

### [참고]
- https://kotlinlang.org/docs/exception-handling.html
