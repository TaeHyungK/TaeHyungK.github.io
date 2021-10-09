---
layout: post
title:  "[Coroutine] 결과를 반환받는 async & await"
categories: [Android, Coroutine]
tags: [Kotlin]
---

### Async

[이전글](https://taehyungk.github.io/posts/android-kotiln-coroutine-dispatchers/)에서 코루틴 상태 관리를 공부하며 `async`는 결과도 반환 받을 수 있도록 언급했던 적이 있다.

async는 결과를 반환하며 결과값은 `Deferred`로 감싸서 반환되게 된다.<br>
결과값은 `await()` 함수를 통해 접근할 수 있다.

> 기억하자.<br>
> launch는 `Job`을 반환하며 이를 통해 상태 관리를 할 수 있다.<br>
> async는 `Deferred`를 반환하며 이를 통해 상태 관리와 결과값에 접근할 수 있다.

<br><br><br><br>

#### 바로 사용해보자.

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

        Log.d(TAG, "onCreate: ")
        CoroutineScope(Dispatchers.Default).launch {
            val networkingDeferred = async(Dispatchers.IO) {
                delay(3000)
                "some networking.."
            }

            val dbDeferred = async(Dispatchers.IO) {
                delay(5000)
                "some db.."
            }

            Log.d(TAG, "${networkingDeferred.await()} | ${dbDeferred.await()}")
        }
    }
    
    companion object {
        private const val TAG = "MainActivity"
    }
}
```

아래 결과를 보면 `await()` 메소드를 통해 결과값(예제에선 문자열)을 반환 받아 로그로 노출하는 것을 볼 수 있다.<br>
또한, onCreate ~ 결과값을 노출하는 로그 까지 걸리는 시간을 보면 5초가 걸린 것을 볼 수 있다.

즉, 시간이 걸리는 작업들이 여러개 있을 때 모든 작업이 완료가 되어야 await()가 위치한 코드가 동작한다는 것을 알 수 있다.  

```log
2021-10-10 02:22:49.957 D/MainActivity: onCreate: 
2021-10-10 02:22:54.992 D/MainActivity: some networking.. | some db..
```

#### 주의할 점

`await()` 함수가 실행되면 코루틴은 결과가 반환될 때까지 기다리게 된다.<br>
이를 **일시 중단(suspend)** 되었다고 하며 이러한 특성으로 인해 `await()` 메소드는 일시 중단이 가능한 코루틴 내부에서만 사용이 가능하다.

그렇기 때문에 `await()` 는 코루틴 내부 혹은 suspend 함수 내에서만 사용이 가능하다.

![스크린샷 2021-10-10 오전 2.44.49.png](/img/스크린샷 2021-10-10 오전 2.44.49.png)

##### 글을 마치며

오늘은 async에 대해 알아보았다. async의 핵심은 **결과 값 반환**이다.

async는 내가 실제로 코루틴의 매력을 처음으로 느끼게 해준 메소드이다.(정확하게는 코루틴 자체의 코드 간결성(?) 때문이긴 하지만..🥸)<br>
서버 데이터와 안드로이드 내부 DB의 값 간에 싱크를 맞추는 작업에서 RxJava가 더 편해서 RxJava를 사용했었는데 너무 복잡해서 코루틴을 쓰니 굉장히 심플한 코드로 작업할 수 있었던 경험이 있다.<br>
async 를 공부하다가 갑자기 왜 나의 경험기까지 적는지는 모르겠지만.. 아무튼 코루틴은 RxJava에 비해 확실히 러닝커브가 낮고 코드도 더욱 더 간결하게 해주는 좋은 기술인 것 같다.

아직까지의 경험으로는 RxJava보다 더 못난점은 찾지 못한 것 같다..! (다만 아직은 RxJava가 더 손에 익은 것도 사실이다 🥲)<br>
앞으로 더 학습하고 사용해보면서 코루틴을 능수능란하게 사용할 수 있는 고수가 되고싶다. 그때까지 화이팅..! 💪


###### 참고 사이트
* [Coroutine 가이드](https://kotlinlang.org/docs/coroutines-guide.html)
* [[Android] 코틀린(Kotlin) 코루틴(Coroutine) 한번에 끝내기](https://whyprogrammer.tistory.com/596)
* [[Coroutine] 4. Dispatcher에 launch, async를 이용해 Coroutine 붙이기](https://kotlinworld.com/142)
* [[Kotlin] 코루틴 Coroutine - async와 await, LifecycleScope과 ViewModelScope 사용법 및 사용예제- 4편](https://underdog11.tistory.com/entry/Kotlin-%EC%BD%94%EB%A3%A8%ED%8B%B4-Coroutine-async%EA%B3%BC-await-LifecycleScope%EA%B3%BC-ViewModelScope-3%ED%8E%B8?category=867873)