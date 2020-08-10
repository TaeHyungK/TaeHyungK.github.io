---
layout: post
title:  "[RxJava2] Observable Operator - Transforming, Filtering, Combining"
categories: Android RxJava2
tags: Kotlin
author: TaeHyungK
---

* content
{:toc}

### Observable Operator - Transforming, Filtering, Combining

#### RxJava2의 Observable 중 Transforming, Filtering, Combining 에 대해 알아보자.

이전 글에서 Observable Operator 중 Creating Operator 에 대해 알아보았는데요.

이번에는 Transforming, Filtering, Combining Operator 에 대해 알아보겠습니다.







<br><br>
##### 먼저 보면 좋은 글
* [[RxJava2] RxJAva2 - Rx란 ?](https://taehyungk.github.io/2020/07/29/android-RxJava2-and-Observable/)
* [[RxJava2] 기본 연산자](https://taehyungk.github.io/2020/08/04/android-RxJava2-Operator/)
* [[RxJava2] Creating Observables - create, just, defer, fromCallable](https://taehyungk.github.io/2020/08/06/android-RxJava2-Creating-Observable-Operator/)

<br><br>


#### Transforming

옵저버블을 사용하다보면 중간에 데이터 흐름(옵저버블)을 변형시키고 싶은 순간이 찾아오게 되는데요.
물론, 데이터를 그대로 보낸 후 Subscriber에서 데이터를 가공해서 사용할 수도 있습니다.
하지만 Subscriber는 메인 스레드에서 동작해야 될 수도 있으므로 최대한 가볍게 두는 것이 좋고, Subscriber는 데이터를 받았을 때에 **반응**을 하기 위한 곳이지 변화를 시키는 곳이 아닙니다.

해서, RxJava에서는 이러한 변화를 가능하게 해주는 다양한 operator 가 있습니다.
이번에는 그러한 operator 들 중 자주 사용하는 operator에 대해 알아보겠습니다.

<br>

##### Map

`map` 함수는 옵저버블에서 받은 데이터를 옵저버로 가기 전에 변형해주며 값 자체를 반환하는 함수 입니다.

![width:500px](https://t1.daumcdn.net/cfile/tistory/99D9D73E5B9471BF03)

```kotlin
// 특정 user 조회 후 map 함수를 통해 데이터 리 예제
fun getUser(userId: String): Single<List<User>> {
    Log.d(TAG, "getUser() called. ")
    return rxRepository.getUser(userId)
        .map { querySnapshot ->
            querySnapshot.map { document ->
                document.toObject(User::class.java)
            }
        }
}
```

<br><br><br>

##### flatMap

`flatMap` 은 옵저버블 작업을 여러번 연계해서 사용할 때 사용하는 함수입니다.<br>
그러나 이렇게 추상적으로 설명하면 쉽게 와닿지가 않습니다.

구체적인 예시를 들어보자면..
> 특정 뉴스 페이지에서 특정 기사를 클릭하면 안드로이드 앱에서 기사의 텍스트, 사진 또는 동영상, 좋아요 갯수, 댓글 정보, 기사의 광고 등을 서버로부터 받아올 것입니다. 이때 이 정보들이 하나의 api에 속하지 않고 `텍스트 요청 api`, `사진/동영상 요청 api`, `좋아요, 댓글 정보 api`가 모두 따로있다면 각각의 api로 호출을 해야합니다.
>
> 각각 api를 호출한 후 각자의 응답에 대해 UI에 표시한다면 `flatMap`을 사용할 일이 없습니다. 하지만 만약에 `텍스트 요청 api`가 실패한 경우, 사진/동영상을 노출하지 않는다는 정책이 추가됐다거나 서버에 동시에 여러 요청을 한번에 하는 것이 서버측에서 부담이 된다는 문제가 발생했다고 가정합시다.
>
> 위와 같은 상황에서 RxJava의 `flatMap`을 사용하면 한번에 해결이 가능합니다.


![width:500px](http://reactivex.io/documentation/operators/images/flatMap.c.png)


```kotlin
// 특정 유저 정보를 조회 후 값이 존재할 경우 회원가입 api 호출하는 예제
repository.getUser(user.id)
    .observeOn(Schedulers.io())
    .flatMap {
        if (!Utils.isListEmpty(it)) {
            Single.fromCallable { true }
        } else {
            repository.addUser(user)
                .toSingleDefault(false)
        }
    }
    .observeOn(AndroidSchedulers.mainThread())
    .subscribeWith(object : DisposableSingleObserver<Boolean>() {
        override fun onSuccess(isDuplicated: Boolean) {
            if (isDuplicated) {
                singleToastResId.value = R.string.toast_sign_up_duplicate_failed
            } else {
                singleToastResId.value = R.string.toast_sign_up_success
                singleIsSuccess.postValue(true)
            }
        }

        override fun onError(e: Throwable) {
            Log.w(SignInViewModel.TAG, "requestLogin() | onFailure ", e)
            singleToastResId.value = R.string.toast_common_error
        }
    })
```

<br><br><br>

#### Filtering

Filtering Operator는 업스트림의 데이터들 중 특정 값들을 수신하지 않기 위해서 존재하는 operator 입니다. 이 operator를 사용하면 사용자가 이벤트를 여러번 발생시켰을 때, 특정 기준에 의해 이벤트를 최소의 이벤트만 송신하는 등의 기능을 구현할 수 있습니다.


##### Debounce

`debounce()`는 기본적으로 지연 방출을 하는 함수입니다. item의 방출 시간을 지정하고 방출 시간이 끝나기 전에 새로운 item이 방출되었을 경우 이전 item의 방출을 중지합니다.

![width:500px](http://reactivex.io/documentation/operators/images/debounce.png)

```kotlin
fun debounceTemp() {
    Log.d(TAG, "debounceTemp() | start | ${System.currentTimeMillis()}")

    val debounceStream: Observable<String> = Observable.create { emitter ->
        emitter.onNext("A") // A, 1초 지연방출
        Thread.sleep(1500) // 1.5초 sleep

        emitter.onNext("B") // B, 1초 지연방출
        Thread.sleep(500) // 0.5초 sleep

        emitter.onNext("C") // C, 1초 지연방출 (B드랍)
        Thread.sleep(250) // 0.25초 sleep

        emitter.onNext("D") // D, 1초 지연방출 (C드랍)
        Thread.sleep(2000) // 2초 sleep

        emitter.onNext("E") // E, 1초 지연방출
        emitter.onComplete()
    }

    debounceStream.subscribeOn(Schedulers.io())
        .debounce(1, TimeUnit.SECONDS)
        .blockingSubscribe {
            Log.d(TAG, "debounceTemp() | subscribe | $it | ${Thread.currentThread()}")
        }

    Log.d(TAG, "debounceTemp() | end | ${System.currentTimeMillis()}")
}

// 결과
2020-08-07 12:09:54.878 .../...: debounceTemp() | start | 1596769794878
2020-08-07 12:09:55.880 .../...: debounceTemp() | A | Thread[main,5,main]
2020-08-07 12:09:58.136 .../...: debounceTemp() | D | Thread[main,5,main]
2020-08-07 12:09:59.137 .../...: debounceTemp() | E | Thread[main,5,main]
2020-08-07 12:09:59.137 .../...: debounceTemp() | end | 1596769799137

// 스트림에 지연 로직이 있을 때 `subscribe()` 을 사용하면 subscribe block은 생산된 결과가 모두 수신될 때 까지 block 되지 않기 때문에 `blockingSubscribe()`을 사용했습니다.
```

<br><br><br>

##### Distinct

`distinct()`는 이전에 방출된 item과 같은 값을 가진 item이 방출될 경우 생략하도록 하는 함수입니다.

![width:500px](http://reactivex.io/documentation/operators/images/distinct.png)

```kotlin
fun distinctTemp() {
        Log.d(TAG, "distinctTemp() | start | ${System.currentTimeMillis()}")

        val debounceStream: Observable<String> = Observable.create { emitter ->
            emitter.onNext("A")
            emitter.onNext("B")
            emitter.onNext("A")
            emitter.onNext("B")
            emitter.onNext("C")
        }

        debounceStream.subscribeOn(Schedulers.io())
            .distinct()
            .subscribe {
                Log.d(TAG, "distinctTemp() | $it")
            }

        Log.d(TAG, "distinctTemp() | end | ${System.currentTimeMillis()}")
    }

// 결과
2020-08-07 12:16:55.815 .../...: distinctTemp() | start | 1596770215815
2020-08-07 12:16:55.820 .../...: distinctTemp() | end | 1596770215820
2020-08-07 12:16:55.823 .../...: distinctTemp() | A
2020-08-07 12:16:55.823 .../...: distinctTemp() | B
2020-08-07 12:16:55.823 .../...: distinctTemp() | C
```

<br><br><br>

#### Combining

Combining Operator는 다수의 옵저버블을 하나로 합치는 방법을 제공하는 operator 입니다.<br>
앞서 설명드렸던 Transforming Operator인 `flatMap()` 등은 1개의 옵저버블을 여러개로 확장해주는 반면, Combining Operator는 여러개의 옵저버블을 내가 원하는 옵저버블로 **결합**시켜 줍니다.

##### concat

`concat()` 은 2개 이상의 옵저버블을 이어 붙여주는 함수입니다. 첫번째 옵저버블에 `onComplete()` 이벤트가 발생해야 두 번째 옵저버블을 구독합니다.
스레드를 활용해 일반적인 코드로 이와 같은 내용을 구현하려면 굉장히 복잡해질 것 같은데 RxJava에서는 이를 제공해주고 있습니다.

![width:500px](http://reactivex.io/documentation/operators/images/concat.png)

```kotlin
fun concatTemp() {
    val firstData = arrayOf("A", "C")
    val secondData = arrayOf("B", "D", "E")
    val firstSource =
        Observable.interval(0, 100, TimeUnit.MILLISECONDS)
            .map {
                firstData[it.toInt()]
            }
            .take(firstData.size.toLong())
            .doOnComplete {
                Log.d(TAG, "concatTemp() | firstSource | onComplete !")
            }
    val secondSource =
        Observable.interval(50, TimeUnit.MILLISECONDS)
            .map {
                secondData[it.toInt()]
            }
            .take(secondData.size.toLong())
            .doOnComplete {
                Log.d(TAG, "concatTemp() | secondSource | onComplete !")
            }

    val originSource =
        Observable.concat(firstSource, secondSource)
            .doOnComplete {
                Log.d(TAG, "concatTemp() | originSource | onComplete !")
            }

    originSource.subscribe {
        Log.d(TAG, "concatTemp: | origin subscribe | ${it.toString()}")
    }
}
// 결과
2020-08-07 12:45:29.399 .../...: concatTemp: | origin subscribe | A
2020-08-07 12:45:29.499 .../...: concatTemp: | origin subscribe | C
2020-08-07 12:45:29.499 .../...: concatTemp() | firstSource | onComplete !
2020-08-07 12:45:29.552 .../...: concatTemp: | origin subscribe | B
2020-08-07 12:45:29.601 .../...: concatTemp: | origin subscribe | D
2020-08-07 12:45:29.651 .../...: concatTemp: | origin subscribe | E
2020-08-07 12:45:29.651 .../...: concatTemp() | secondSource | onComplete !
2020-08-07 12:45:29.652 .../...: concatTemp() | originSource | onComplete !

```

`concat()` 에서 주의해야 할 사항은 **첫번째 옵저버블이 `onComplete` 이벤트가 발생하지 않게 되면 두번째 옵저버블은 영원히 대기한다는 점** 입니다.
즉, 잠재적인 메모리 누수에 대한 위험을 의미합니다. 따라서 앞에 존재하는 옵저버블이 반드시 `onComplete`가 될 수 있도록 작업하여야 합니다.

추가로, `concat()`을 이용해 연결할 수 있는 옵저버블의 최대 갯수는 **4개** 입니다.

<br><br><br>

##### merge

`merge()` 함수는 `zip()` 함수나 `combineLatest()` 함수와 비교하면 가장 단순한 Combining 함수입니다.
입력 옵저버블의 순서와 모드 옵저버블이 데이터를 발행하는지에 대해 관여하지 않고 어느 것이든 업스트림에 먼저 입력되는 데이터를 그대로 발행합니다.

![width:500px](http://reactivex.io/documentation/operators/images/merge.png)

```kotlin
fun mergeTemp() {
    val firstData = arrayOf("A", "C")
    val secondData = arrayOf("B", "D", "E")
    val firstSource =
        Observable.interval(0, 100, TimeUnit.MILLISECONDS)
            .map {
                firstData[it.toInt()]
            }
            .take(firstData.size.toLong())
    val secondSource =
        Observable.interval(50, TimeUnit.MILLISECONDS)
            .map {
                secondData[it.toInt()]
            }
            .take(secondData.size.toLong())

    val originSource =
        Observable.merge(firstSource, secondSource)
            .doOnComplete {
                Log.d(TAG, "mergeTemp() | originSource | onComplete !")
            }

    originSource.subscribe {
        Log.d(TAG, "mergeTemp: | origin subscribe | ${it.toString()}")
    }
}

// 결과
2020-08-07 12:55:08.308 .../...: mergeTemp: | origin subscribe | A
2020-08-07 12:55:08.356 .../...: mergeTemp: | origin subscribe | B
2020-08-07 12:55:08.406 .../...: mergeTemp: | origin subscribe | C
2020-08-07 12:55:08.407 .../...: mergeTemp: | origin subscribe | D
2020-08-07 12:55:08.457 .../...: mergeTemp: | origin subscribe | E
2020-08-07 12:55:08.459 .../...: mergeTemp() | originSource | onComplete !

```

예시를 보면 첫번째 Observable은 대기시간 없이 100ms 간격으로 데이터를 발행하고
두번째 Observable은 50ms 간격으로 데이터를 발행하고 있습니다.

결과적으로 `originSource` 라는 Observable을 통해 두 옵저버블을 `merge()` 하여 2개의 값이 `originSource` Observable에 섞이게 됩니다.

이때, 첫번째 Observable과 두번째 Observable은 서로 다른 스레드에서 발행되고 있다는 점을 알 수 있습니다.

```kotlin
2020-08-07 14:32:41.455 .../...: mergeTemp: | origin subscribe | A | Thread[RxComputationThreadPool-1,5,main]
2020-08-07 14:32:41.505 .../...: mergeTemp: | origin subscribe | B | Thread[RxComputationThreadPool-2,5,main]
2020-08-07 14:32:41.555 .../...: mergeTemp: | origin subscribe | C | Thread[RxComputationThreadPool-1,5,main]
2020-08-07 14:32:41.555 .../...: mergeTemp: | origin subscribe | D | Thread[RxComputationThreadPool-2,5,main]
2020-08-07 14:32:41.605 .../...: mergeTemp: | origin subscribe | E | Thread[RxComputationThreadPool-2,5,main]
```

<br><br><br>

##### zip

`zip()` 함수는 여러개의 옵저버블을 합쳐서 전송하게 하는 operator 입니다. 특정 item 끼리 합쳐서 두개의 발행을 합쳐서 내려주게 됩니다.

![width:500px](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=http%3A%2F%2Fcfile4.uf.tistory.com%2Fimage%2F990BAD505C75F230158455)

```kotlin
fun zipTemp() {
    val firstSource = Observable.just("A", "B")
    val secondSource = Observable.create<Long> { emitter ->
        emitter.onNext(1L)
        Thread.sleep(2000)

        emitter.onNext(2L)
        Thread.sleep(2000)

        emitter.onNext(3L)
        emitter.onComplete()
    }

    Observable.zip(firstSource, secondSource,
        BiFunction<String, Long, String> { t1, t2 -> t1 + t2})
        .subscribe {
            Log.d(TAG, "zipTemp() | onNext | $it | ${Thread.currentThread()}")
        }
    }

// 결과
2020-08-07 15:02:43.186 .../...: zipTemp() | onNext | A1 | Thread[main,5,main]
2020-08-07 15:02:45.189 .../...: zipTemp() | onNext | B2 | Thread[main,5,main]
```

위 예제에서와 같이 2초 간격으로 발행되며 두 옵저버블의 데이터가 합쳐진 것을 확인할 수 있습니다.


<br><br><br>

###### 참고 사이트
* [한로니: RxJava, Observable 생성하기](https://brunch.co.kr/@lonnie/18)
* [Owl Life: [RxJava] create, just, defer와 fromCallable 차이점 & 샘플코드](https://softwaree.tistory.com/36)
* [범석의 안드로이드 메모장: Observable -7 (fromCallable함수)](https://beomseok95.tistory.com/10)
* [태임쓰의 개발블로그: [RxJava2] #3. 리액티브 연산자 - 기초 (map, filter, reduce)](https://taeiim.tistory.com/entry/RxJava2-3-%EB%A6%AC%EC%95%A1%ED%8B%B0%EB%B8%8C-%EC%97%B0%EC%82%B0%EC%9E%90-%EA%B8%B0%EC%B4%88-map-filter-reduce)
* [Medium.CCanary Park: RxJAVA, Part 1 : 기초](https://medium.com/@LIP/rxjava-29cfb3ceb4ca)
* [아는 개발자: RxJava - flatMap](https://selfish-developer.com/entry/RxJava-flatMap)
* [Zero to Hero: RxJava - Filter Operator편](https://myungjunchae.github.io/android/rxjava/rxjava-filter/)
* [범석의 안드로이드 메모장: 리액티브연산자[결합 연산자]- 16(merge함수,concat함수)](https://beomseok95.tistory.com/36)
