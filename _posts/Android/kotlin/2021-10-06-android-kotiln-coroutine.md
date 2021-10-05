---
layout: post
title:  "[Coroutine] 코루틴 막무가내로 사용해보기"
categories: [Android, Coroutine]
tags: [Kotlin]
---

### Coroutine

비동기 처리를 하는 데에는 `Rx`, (~~지금은 사라진..~~) `AsyncTask` 등 여러가지 방법이 있다. 오늘은 여기서 "등"에 해당하는 `Coroutine` 에 대해 학습해보고자 한다.

코루틴은 러닝 커브가 높기로 유명한 `Rx` 보다 배우기 쉽고 비동기스럽지 않은 코드로 비동기 처리를 할 수 있으며 메모리를 효율적으로 사용하므로 굉장히 이점이 많은 기술이다.

안드로이드에서는 주로 별도의 스레드를 사용해야 되는 케이스인 **네트워킹**, **DB 접근** 에서 주로 사용하게 될 것으로 예상된다.





<br><br><br><br>

#### 코루틴 사용하기

##### 디펜던시 추가

먼저 코루틴을 사용하려면 아래와 같이 gradle 파일에 디펜던시를 추가해주어야 한다.

```gradle
dependecies {
    ...

    def coroutine_version = "1.5.0"
    // Coroutine
    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:$coroutine_version"
    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:$coroutine_version"

    ...
}
```

<br><br><br><br>

##### GlobalScope

코루틴은 Coroutine Scope 을 통해서 코루틴이 사용되는 범위를 지정해주어야 하는데, 그 중에서도 전역 스코프인 `GlobalScope` 을 사용해보도록 한다.

> `GlobalScope` 은 말 그대로 전역 스코프라서 사용하는 즉시 새로운 스레드가 생성되고 그 안에서 비동기 작업이 가능해진다.
>
> 이 스코프는 **최상위 코루틴**이며 어플리케이션이 종료될 때까지 살아있는 코루틴이므로 조심해서(?) 사용해야 한다. 

```kotlin
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.util.Log
import kotlinx.coroutines.GlobalScope
import kotlinx.coroutines.delay
import kotlinx.coroutines.launch

class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        GlobalScope.launch {
            delay(3000)
            Log.d(TAG, "onCreate: GlobalScope.launch INNER | ${Thread.currentThread().name}")
        }
        Log.d(TAG, "onCreate: GlobalScope OUTER | ${Thread.currentThread().name}")
    }

    companion object {
        private const val TAG = "MainActivity"
    }
}
```

위에서 `GlobalScope` 를 사용하면서 `launch` 라는 메소드를 사용하여 코루틴을 열어주고 해당 코루틴을 3초간 delay 한 후 로그를 찍고
그 밖에서도 로그를 찍도록 해주었다.

그 결과는 아래와 같다.

```log
2021-10-06 00:30:02.371 D/MainActivity: onCreate: GlobalScope OUTER | main
2021-10-06 00:30:05.400 D/MainActivity: onCreate: GlobalScope.launch INNER | DefaultDispatcher-worker-1
```

Scope 외부는 MainThread 를 통해 순차적으로 호출이 되었고 Scope 내부는 WorkerThread 를 통해 실행되었으며 `delay()` 메소드를 통해 3초간 쉬었다가 찍히는 것을 확인할 수 있었다.

<br><br><br><br>

##### suspend 함수

위 코드에서 `delay()` 코드의 앞 쪽을 보면 화살표 + 물결 표시가 되어 있는 아이콘을 볼 수 있는데 이것은 `suspend` 함수를 의미한다.<br>
영어에서도 유추할 수 있듯이 코루틴 내에서 이 함수를 처리할 때까지 코루틴은 잠시 멈추고 `suspend` 함수가 완료되면 다음 코드가 실행되게 된다.

```kotlin
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.util.Log
import kotlinx.coroutines.GlobalScope
import kotlinx.coroutines.delay
import kotlinx.coroutines.launch

class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        Log.d(TAG, "onCreate() called.")

        GlobalScope.launch {
            delay(1000)
            val networkData = getNetworkData()
            val dbData = getDbData()

            Log.d(TAG, "networkData: $networkData")
            Log.d(TAG, "dbData: $dbData")
        }
    }

    private suspend fun getNetworkData(): String {
        delay(3000)
        return "get networkData Data.."
    }

    private suspend fun getDbData(): String {
        delay(3000)
        return "get Database Data.."
    }

    companion object {
        private const val TAG = "MainActivity"
    }
}
```

3초간 동작하게 되는 `getNetwokData()`, `getDbData()` 메소드를 만들고 `GloblScope` 에서 호출하도록 하고 그 이후에 로그를 찍도록 해보았다.

그 결과는 아래와 같다.

```log
2021-10-06 00:47:51.306 D/MainActivity: onCreate() called.
2021-10-06 00:47:58.458 D/MainActivity: networkData: get networkData Data..
2021-10-06 00:47:58.459 D/MainActivity: dbData: get Database Data..

```

예상했던 것 처럼 onCreate() 가 실행되고 deley 1초 + `suspend` 함수 `getNetwokData()`가 동작하는 시간 3초 + `suspend` 함수 `getDbData()` 가 동작하는 시간 3초가 걸려 
총 7초 후에 로그가 찍히는 것을 확인할 수 있다. 

<br><br><br><br>

##### 글을 마치며

작년에 Rx를 공부하면서 Coroutine을 꼭 공부하리라 마음먹었는데 드디어 코루틴을 공부하게 되었다.. 하하.. 😅 지금이라도 늦지 않았으니 열심히 공부해보자 !

이번 글에서는 매우매우 간단하게 Coroutine 이 무엇인지와 대강 어떤 느낌으로 사용되는지를 체험해보았다. 다음에는 `Dispatcher` 에 대해 알아보고 이를 사용해보는 시간을 가져보도록 하자.


###### 참고 사이트
* [Coroutine 가이드](https://kotlinlang.org/docs/coroutines-guide.html)
* [[Android][Kotlin] 코루틴(Coroutine) 사용해보기](https://blog.yena.io/studynote/2020/04/26/Android-Kotlin-Coroutine.html)
* [[Android] 코틀린(Kotlin) 코루틴(Coroutine) 한번에 끝내기](https://whyprogrammer.tistory.com/596)
* [Velog.peppermint100: 안드로이드 코루틴](https://velog.io/@peppermint100/%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C-%EC%BD%94%EB%A3%A8%ED%8B%B4-1)