---
layout: post
title:  "[RxJava2] Observable 생성은 어떻게 하는걸까?"
categories: [Android, RxJava2]
tags: [Kotlin]
---

### Creating Observables - create, just, defer, fromCallable

#### RxJava2의 Observble 생성을 위한 Operator 중 create, defer, fromCallable에 대해 알아보자.

다른 포스팅 글에서 RxJava란 무엇인지, 기본 연산자는 무엇인지에 대해 공부해봤었는데요.

RxJava의 연산자는 Creating, Transforming, Filtering, Combining, Observable Utility Operators 등 여러 종류의 연산자가 있습니다.

오늘은 Creating Operator 에 대해 알아보겠습니다.







<br><br>
##### 먼저 보면 좋은 글
* [[RxJava2] RxJAva2 - Rx란 ?](https://taehyungk.github.io/2020/07/29/android-RxJava2-nd-Observable/)
* [[RxJava2] 기본 연산자](https://taehyungk.github.io/2020/08/04/android-RxJava2-Operator/)

<br><br>

#### Creating

RxJava에서는 옵저버블을 생성하는 다양한 방법을 제공하고 있습니다. 기존 함수의 결과 값을 옵저버블로 발행하거나 리스트의 값을 하나씩 발행할 수도 있으며 옵저버블 생성을 구독이 발생할 때로 지연시킬 수도 있습니다.<br>
심지어는 아무것도 발행하지 않고 종료하거나 에러를 발행하는 옵저버블을 생성할 수도 있습니다.


#### Observable.create()

`create()` 함수는 옵저버블 생성 시 가장 많이 사용하는 함수 중 하나입니다.
이 함수는 `OnSubscribe` 객체를 파라미터로 가지고 있어 구독이 발생하면 이 객체의 `call()` 함수가 실행되게 됩니다.

`create()` 함수는 `onNext`, `onComplete`, `onError` 같은 이벤트를 적절한 상황에 직접 호출해주어야 합니다.

주의해야 할 점은 `onComplete()`나 `onError()` 함수가 호출된 이후에는 옵저버의 어떠한 함수도 호출하지 않아야 합니다.

![width:500px center](http://reactivex.io/documentation/operators/images/create.c.png)

```kotlin
fun getBoardList(): Observable<QuerySnapshot> {
    return Observable.create { emitter ->
        firestoreDb.collection(DataConst.BOARDS)
            .orderBy(DataConst.TIMESTAMP, Query.Direction.DESCENDING)
            .addSnapshotListener{ querySnapshot, e ->
                if (e != null) {
                    Log.w(TAG, "getBoardList() | occur error", e)
                    emitter.onError(e)
                    return@addSnapshotListener
                }

                querySnapshot?.let {
                    emitter.onNext(it)
                    return@addSnapshotListener
                }
            }
    }
```

<br><br><br>

#### Observable.just()

`just()` 함수는 인자로 넣은 데이터를 바로 방출할 때 사용되는 함수입니다.<br>
내부적으로 ScalarDisposable을 사용하고있어 `onNext()`를 통해 데이터를 방출하고 `onComplete()`까지 호출하게 됩니다.

![width:500px](http://reactivex.io/documentation/operators/images/just.c.png)

`just()` 는 간단한 함수이지만 **주의 해야 할 사항**이 있습니다.<br>
##### 구독자가 구독할 때 옵저버블의 just가 호출이 끝난 시점까지 현재 쓰레드가 더 이상 진행되지 않고 펜딩된다는 점입니다.

그렇기 때문에 `just()` 호출 시에 무거운 로직을 넣게 되면 성능상 굉장히 좋지 않은 영향을 끼치게 될 수 있습니다.

```kotlin
// just 호출 시 오래 걸리는 로직을 넣은 케이스(1000ms)
fun justTemp() {
    Log.d(TAG, "justTemp() | start | ${System.currentTimeMillis()}")
    Observable.just(getDataWithSleep())
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        .subscribeWith(object: DisposableObserver<String>() {
            override fun onComplete() {
                Log.d(TAG, "justTemp | onComplete() called.")
            }

            override fun onNext(t: String) {
                Log.d(TAG, "justTemp | onNext() | $t")
            }

            override fun onError(e: Throwable) {
                Log.w(TAG, "justTemp | onError: $e", e)
            }

        })
    Log.d(TAG, "justTemp() | end | ${System.currentTimeMillis()}")
}

fun getDataWithSleep(): String {
    try {
        Thread.sleep(1000)
    } catch (e: InterruptedException) {
        e.printStackTrace()
    }
    return "Data With Sleep"
}
// 결과
2020-08-06 18:15:41.512 .../...: justTemp() | start | 1596705341512
2020-08-06 18:15:42.515 .../...: justTemp() | end | 1596705342515
2020-08-06 18:15:42.532 .../...: justTemp | onNext() | Data With Sleep
2020-08-06 18:15:42.532 .../...: justTemp | onComplete() called.
```
결과에서와 같이 end까지 1초가 걸리는 것을 볼 수 있습니다.

`just()` 연산자에서 방출하는 데이터는 `getDataWithSleep()` 메소드를 통해 전달받은 결과값입니다.<br>
해당 메소드에서는 약 1초간 딜레이를 주었습니다. 이렇게 되면 업스트림이 **io scheduler** 임에도 불구하고 구독자가 구독 후에 다음 메소드를 진행하는데 딜레이가 됨을 알 수 있습니다.<br>
RxJava를 주로 쓰는 이유 중 하나가 비동기 처리를 위함인데 위와 같은 코드는 전혀 비동기적이지 못한 결과를 보여주고 있는 것을 볼 수 있습니다.<br>
추가로, `getDataWithSleep()` 메소드 내에 스레드를 로깅해보면

```kotlin
2020-08-06 18:22:39.642 .../...: getDataWithSleep() | Thread[main,5,main]
```

Main Thread 에서 돌고있는 것을 확인 할 수 있습니다.<br>
그렇기 때문에 just를 사용한 옵저버 생성 시에는 유의해서 사용해야 합니다.

<br><br><br>

#### Observable.defer()

`defer()` 함수는 지연 초기화를 제공하는 함수입니다. 즉, **구독이 발생할 때 비로소 옵저버블을 생성**하게 하는 함수입니다.<br>
그렇기 때문에 다른 연산자들은 연산자를 선언하는 순간 메모리에 할당되지만 `defer()`는 subscribe가 호출될 때 할당되게 되며 각각의 옵저버에게 매번 새로운 옵저버블을 생성시키게 됩니다.

이로 인하여 `just()` 와는 다르게 스트림 생성을 지연하는 효과를 볼 수 있습니다. 앞서 설명드린 just의 예제를 defer 를 적용하여 어떤 결과가 나오는지 봐보도록 하겠습니다.

![width:500px](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fdys216%2FbtqygsEptw2%2FVLIdD8GK2vBwfNc9Rk2fz0%2Fimg.png)

```kotlin
fun deferTemp() {
    Log.d(TAG, "deferTemp() | start | ${System.currentTimeMillis()}")
    Observable.defer { Observable.just(getDataWithSleep()) }
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        .subscribeWith(object: DisposableObserver<String>() {
            override fun onComplete() {
                Log.d(TAG, "deferTemp | onComplete() called.")
            }

            override fun onNext(t: String) {
                Log.d(TAG, "deferTemp | onNext() | $t")
            }

            override fun onError(e: Throwable) {
                Log.w(TAG, "deferTemp | onError: $e", e)
            }

        })
    Log.d(TAG, "deferTemp() | end | ${System.currentTimeMillis()}")
}

fun getDataWithSleep(): String {
    try {
        Thread.sleep(1000)
    } catch (e: InterruptedException) {
        e.printStackTrace()
    }

    return "Data With Sleep"
}
// 결과
2020-08-06 18:28:52.983 .../...: deferTemp() | start | 1596706132983
2020-08-06 18:28:52.984 .../...: deferTemp() | end | 1596706132984
2020-08-06 18:28:53.985 .../...: deferTemp | onNext() | Data With Sleep
2020-08-06 18:28:53.985 .../...: deferTemp | onComplete() called.
```
결과에서 start에서 end까지 큰 차이 없이 종료되는 것을 볼 수 있습니다.

보시는 것 처럼 1초간 딜레이를 주는 `getDataWithSleep()` 메소드를 호출했음에도 불구하고 stream 생성의 지연 효과로 인해 비동기 작업을 하는데 문제가 없음을 확인할 수 있습니다.

추가로, `getDataWithSleep()` 메소드 내에 스레드를 로깅해보면

```kotlin
2020-08-06 18:28:53.985 .../...: getDataWithSleep() | Thread[RxCachedThreadScheduler-1,5,main]
```

해당 메소드가 **io scheduler** 에서 돌고있는 것을 확인 할 수 있습니다.

하지만, Observable을 위한 Observable을 생성해야 하는 단점이 있습니다.

<br><br><br>

#### Observable.fromCallable

`fromCallable()` 은 `defer()`와 마찬가지로 스트림 생성을 지연하는 효과를 가지고 있습니다. 하나의 차이점이 존재하는데 이는 옵저버블에서 데이터를 방출(emit) 할 때 추가로 Observable을 생성하지 않고 바로 데이터를 스트림으로 전달할 수 있다는 점입니다.<br>
`defer()`의 단점을 보완하여 `defer()`의 효과를 그대로 가져갈 수 있는 operator 입니다.


```kotlin
fun fromCallableTemp() {
    Log.d(TAG, "fromCallableTemp() | start | ${System.currentTimeMillis()}")
    Observable.fromCallable { getDataWithSleep() }
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        .subscribeWith(object: DisposableObserver<String>() {
            override fun onComplete() {
                Log.d(TAG, "fromCallableTemp | onComplete() called.")
            }

            override fun onNext(t: String) {
                Log.d(TAG, "fromCallableTemp | onNext() | $t")
            }

            override fun onError(e: Throwable) {
                Log.w(TAG, "fromCallableTemp | onError: $e", e)
            }

        })
    Log.d(TAG, "fromCallableTemp() | end | ${System.currentTimeMillis()}")
}

fun getDataWithSleep(): String {
    try {
        Thread.sleep(1000)
    } catch (e: InterruptedException) {
        e.printStackTrace()
    }

    Log.d(TAG, "getDataWithSleep() | ${Thread.currentThread().name} ")
    return "Data With Sleep"
}
// 결과
2020-08-06 18:47:11.743 .../...: fromCallableTemp() | start | 1596707231742
2020-08-06 18:47:11.744 .../...: fromCallableTemp() | end | 1596707231744
2020-08-06 18:47:12.746 .../...: fromCallableTemp | onNext() | Data With Sleep
2020-08-06 18:47:12.746 .../...: fromCallableTemp | onComplete() called.
```
start와 end 사이에 시간 차이가 거의 없음을 알 수 있습니다.

Observable.fromCallable 호출부를 보면 `defer()`와 다르게 `Observable.just()` 호출을 통한 스트림 생성 로직이 없음에도 Observable 생성 시간이 영향을 주지 않는 것을 확인하였습니다.

추가로, `getDataWithSleep()` 에 스레드를 로깅해보면

```kotlin
2020-08-06 18:47:12.745 .../...: getDataWithSleep() | Thread[RxCachedThreadScheduler-1,5,main]
```
해당 메소드가 **io scheduler** 에서 돌고있는 것을 확인 할 수 있습니다.

<br><br><br>

###### 참고 사이트
* [한로니: RxJava, Observable 생성하기](https://brunch.co.kr/@lonnie/18)
* [Owl Life: [RxJava] create, just, defer와 fromCallable 차이점 & 샘플코드](https://softwaree.tistory.com/36)
* [범석의 안드로이드 메모장: Observable -7 (fromCallable함수)](https://beomseok95.tistory.com/10)
