---
layout: post
title:  "[Coroutine] CoroutineScope와 Dispatchers, 코루틴의 상태 관리"
categories: [Android, Coroutine]
tags: [Kotlin]
---

### CoroutineScope

모든 코루틴은 Scope 내에서 실행이 되어야 한다. Scope는 [이전 글](https://taehyungk.github.io/posts/android-kotiln-coroutine/) 에서 
사용해보았던 것 처럼 `GlobalScope` 와 `CoroutineScope`이 존재한다.

`GlobalScope`는 앱의 생명주기와 함께 동작하기 때문에 별도 생명 주기 관리가 필요 없으며 앱의 시작부터 종료까지 긴~~ 시간 실행되는 코루틴에 적합하다.

`CoroutineScope`는 버튼을 눌러 다운로드하거나 서버와 통신한다거나 하는 등의 필요할 때만 시작, 완료되면 종료하는 용도로 사용된다.







번외로 `ViewModelScope` 라는 것도 존재하는데, 이는 Jetpack 아키텍처의 뷰모델 컴포넌트를 사용할 때 `ViewModel` 에서 사용하기 위해 제공되는 Scope이다.
해당 스코프로 실행되는 코루틴은 `ViewModel` 이 destory될 때 자동으로 취소가 된다.<br>
(아주 유용하게 생겼다 😲)

<br><br><br><br>

### Dispatchers

`CoroutineSCope`의 경우 `GlobalScope`와는 다르게 `Dispatcher` 라는 것을 지정할 수 있는데 이는 **코루틴이 실행될 스레드**를 지정하는 것이라고 생각하면 된다.

이러한 Dispatcher에는 `Default`, `IO`, `Main`, `Unconfined` 등이 있다.
* Dispatchers.Default: 안드로이드 기본 스레드풀 사용하는 Dispatcher. CPU를 많이 쓰는 작업에 최적화. (데이터 정렬, 복잡한 연산 등..)
* Dispatchers.IO: 이름처럼 IO 작업을 할 때에 최적화가 되어있는 Dispatcher. (이미지 다운로드, 파일 입출력, 네트워킹, DB 작업 등..)
* Dispatchers.Main: 안드로이드 Main 스레드를 사용하는 Dispatcher. (UI 작업)
* Dispatchers.Unconfined: 호출한 context를 기본으로 사용하는데 중단 후 다시 실행될 때 context가 바뀌면 바뀐 context를 따라가는 특이한 Dispatcher. 

```kotlin
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.util.Log
import kotlinx.coroutines.*

class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        CoroutineScope(Dispatchers.Default).launch {
            Log.d(TAG, "Dispatchers.Default: ${Thread.currentThread().name}")
        }

        CoroutineScope(Dispatchers.IO).launch {
            Log.d(TAG, "Dispatchers.IO: ${Thread.currentThread().name}")
        }

        CoroutineScope(Dispatchers.Main).launch {
            Log.d(TAG, "Dispatchers.Main: ${Thread.currentThread().name}")
        }

        CoroutineScope(Dispatchers.Unconfined).launch {
            Log.d(TAG, "Dispatchers.Unconfined before: ${Thread.currentThread().name}")
            delay(2000)
            Log.d(TAG, "Dispatchers.Unconfined after: ${Thread.currentThread().name}")
        }
    }
}
```

위 코드의 결과는 아래와 같다.<br>
(참고로 결과의 순서와 DefaultDispatcher-worker-1의 숫자는 매번 바뀔 수 있다.)

```log
2021-10-06 23:53:20.660 D/MainActivity: Dispatchers.Default: DefaultDispatcher-worker-2
2021-10-06 23:53:20.660 D/MainActivity: Dispatchers.IO: DefaultDispatcher-worker-1
2021-10-06 23:53:20.672 D/MainActivity: Dispatchers.Unconfined before: main
2021-10-06 23:53:20.838 D/MainActivity: Dispatchers.Main: main
2021-10-06 23:53:22.691 D/MainActivity: Dispatchers.Unconfined after: kotlinx.coroutines.DefaultExecutor
```

Default, IO는 Worker 스레드에서 실행되었고 Main은 Main 스레드에서,<br>
Unconfined는 caller 스레드인 Main 스레드에서 실행되었다가 `delay()` 이후 DefaultExecutor에서 호출되었다.

> delay() 함수가 DefaultExecutor 에 의해 호출되기 때문에 실행되는 스레드가 변경되는 것을 볼 수 있다.

<br><br><br><br>

#### 코루틴의 상태 관리

여태까지 예제에서는 늘 `launch` 를 사용해왔는데 사실 `async` 라는 것도 있다.

`launch` 는 코루틴 실행, 상태 관리를 할 때에 사용되고 `async`는 코루틴 실행, 상태 관리와 더불어서 결과도 반환 받을 수 있다.<br>

코루틴을 시작하는 2가지 블록에 대해 간단하게 알게 되었으니 코루틴의 상태를 관리하는 메소드들에 대해서 알아보도록 하자.

##### cancel()

코루틴의 동작을 취소하는 상태 관리 메소드로 호출되면 하나의 스코프 안에 여러 코루틴이 존재하는 경우 하위 코루틴까지 모두 멈추게 된다.

cancel() 메소드를 호출 할 대상이 어떤 것인지 학습한 내용만을 봤을 때에는 알 수가 없어서 당황스러웠을 수 있다.<br>
해답은 `launch` 의 내부 구현을 보면 알 수 있다. `launch`는  `Job` 이라는 반환 값이 존재한다는 것을 알 수 있다.

바로 이 `Job` 을 통해서 코루틴의 상태를 관리할 수 있다. 즉, Job으로 cancel() 을 할 수 있다.

```kotlin
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.util.Log
import androidx.databinding.DataBindingUtil
import com.study.taehyungk.databinding.ActivityMainBinding
import kotlinx.coroutines.*

class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = DataBindingUtil.setContentView(this, R.layout.activity_main)
        binding.run {
            lifecycleOwner = this@MainActivity
        }

        val parentJob = CoroutineScope(Dispatchers.Default).launch {
            val childJob = launch {
                for (i in 0..10) {
                    delay(500)
                    Log.d(TAG, "childJob i: $i")
                }
            }
        }

        binding.button.setOnClickListener {
            parentJob.cancel()
        }

    }


    companion object {
        private const val TAG = "MainActivity"
    }
}
```

앱이 실행되면 parentJob -> childJob을 생성하고 childJob에서 포문을 돌면서 로그를 찍기 시작한다.

중간에 버튼을 클릭해 parentJob 을 cancel() 해주면 childJob 까지도 멈추게 되어 로그가 더이상 찍히지 않는 것을 확인할 수 있다.

<br><br><br><br>

##### join()

코루틴 내부에 여러 launch 블록이 있는 경우 모두 새로운 코루틴으로 분기되어 동시에 실행되기 때문에 순서를 보장할 수가 없다.

이러한 상황에서 순차적으로 실행되도록 하는 것이 `join()` 이다.

```kotlin
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.util.Log
import androidx.databinding.DataBindingUtil
import com.study.taehyungk.databinding.ActivityMainBinding
import kotlinx.coroutines.*

class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = DataBindingUtil.setContentView(this, R.layout.activity_main)
        binding.run {
            lifecycleOwner = this@MainActivity
        }

        CoroutineScope(Dispatchers.Default).launch {
            launch {
                for (i in 0..3) {
                    delay(500)
                    Log.d(TAG, "first launch. i: $i")
                }
            }.join()

            launch {
                for (i in 0..3) {
                    delay(500)
                    Log.d(TAG, "second launch. i: $i")
                }
            }
        }

    }


    companion object {
        private const val TAG = "MainActivity"
    }
}
```

`join()` 을 만나게 되면 아래 결과처럼 해당 Job이 완료될 때까지 기다렸다가 이후 코드를 실행하게 된다.

```log
2021-10-07 00:26:33.108 D/MainActivity: first launch. i: 0
2021-10-07 00:26:33.612 D/MainActivity: first launch. i: 1
2021-10-07 00:26:34.114 D/MainActivity: first launch. i: 2
2021-10-07 00:26:34.623 D/MainActivity: first launch. i: 3
2021-10-07 00:26:35.132 D/MainActivity: second launch. i: 0
2021-10-07 00:26:35.635 D/MainActivity: second launch. i: 1
2021-10-07 00:26:36.139 D/MainActivity: second launch. i: 2
2021-10-07 00:26:36.641 D/MainActivity: second launch. i: 3

```

##### 이외의 Job 메소드

join, cancel 뿐만 아니라 Job 으로 할 수 있는 메소드들이 더 있는데 이를 소개하고자 한다.

* start(): 현재 coroutine의 동작 상태를 체크하며 동작중인 경우 true, 준비 또는 완료 상태이면 false를 반환한다.
* cancelAndJoin(): 현재 coroutine에 종료하라는 신호를 보내고 정상 종료할 때 까지 대기한다.
* cancelChildren(): CoroutineScope 내에 작성한 child coroutine들을 종료한다. `cancel()` 과는 다르게 child만 종료하고 부모는 종료하지 않는다.

##### 글을 마치며

오늘은 CoroutineScope, Dispatchers에 대해서만 알아보려고 했는데 어쩌다보니 코루틴의 상태 관리 까지도 알아보게 되었다.

개인적으로 아직은 코루틴이 어떤 느낌인지 감이 잘 오지 않는다.. 누군가 이 글을 본다면 꼭 예제들을 따라해보면서 감을 익히는 것을 추천한다. 💪


###### 참고 사이트
* [Coroutine 가이드](https://kotlinlang.org/docs/coroutines-guide.html)
* [[Android] 코틀린(Kotlin) 코루틴(Coroutine) 한번에 끝내기](https://whyprogrammer.tistory.com/596)
* [[Coroutine] Coroutine context and dispatchers](https://aroundck.tistory.com/6044)
* [Kotlin Coroutines의 Job 동작을 알아보자](https://thdev.tech/kotlin/2019/04/08/Init-Coroutines-Job/)