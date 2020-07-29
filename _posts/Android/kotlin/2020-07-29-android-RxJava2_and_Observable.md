---
layout: post
title:  "[RxJava2] RxJava2와 Observable"
categories: Android RxJava2
tags: Kotlin
author: TaeHyungK
---

* content
{:toc}

### RxJava2

#### RxJava를 배우기에 앞서 Rx란 무엇인지에 알아보자.

### Rx(ReactiveX) 란 ?
비동기 이벤트 기반 프로그래밍 라이브러리로 Observer 패턴 + Iterator 패턴 (Reactive, 비동기 이벤트 방식)과 Functional 프로그래밍 (이벤트 처리 방식)이 합쳐진 것을 의미합니다.







> 여기서, Reactive (반응형) Programming이란 ?
>
> 데이터의 흐름과 변화의 전파에 중점을 둔 선언적 프로그래밍 패러다임. 정적 또는 동적인 **데이터 흐름**을 쉽게 표현할 수 있으며, 데이터 프름을 통해 연관된 실행 모델이 자동으로 변화를 전파할 수 있는 것을 의미한다.

반응형 프로그래밍(Reactive Programming)의 정의는 위와 같지만 중요한 것은 **로직이나 데이터를 하나의 흐름**으로 봄으로써 기존에 우리가 다루기 어려웠던 **비동기 작업을 쉽게 다루자**는 것입니다.

|기존의 비동기 방식|Reactive 비동기 방식|
|-----------------|--------------------|
|비동기 작업 A를 하면 B를 한다.<br>(B는 Callback 또는 Interface 함수)|비동기 작업 A가 이벤트를 **발행**하면 B가 **구독**을 수행한다.<br>(B는 Observer)|

##### 정리해보면, 명령형 프로그램은 작성된 코드가 정해진 순서대로 실행되는 것에 반해 반응형 프로그램은 데이터 흐름을 먼저 정의하고 데이터가 변경되었을 때 연관되는 함수나 수식이 업데이트 되는 방식입니다.

<br><br><br><br>
> 다음으로, Functional (함수형) Programming이란?
>
> 자료 처리를 함수의 연산으로 취급하고, 가변적인 상태나 데이터를 피하는 프로그래밍 패러다임. 명령형 프로그래밍에서는 상태를 바꾸는 것을 강조하는 것과 달리, 함수형 프로그래밍은 함수의 응용을 강조한다. 프로그래밍이 문이나 식이 아닌 선언으로 수행되는 선언형 프로그래밍 패러다임을 따르고 있다.
>

함수형 프로그래밍(Functional Programming)의 정의는 위와 같지만 함수형을 많이 접해보지 않은 사람에게는 약간 ~~의미 심장~~ 합니다. 자주 사용했던 명령형 프로그래밍 패러다임과 선언형 프로그래밍 패러다임을 비교해보면..

|명령형 프로그래밍(절차형, 객체지향형 등..)|함수형 프로그래밍|
|-----------------|--------------------|
|작업을 이렇게 저렇게 수행해라.<br>데이터 상태를 변경해서 결과를 얻어내자. 의 방식<br>즉, 알고리즘을 명시하는 방식|특정 데이터가 되어라.<br> 즉, 목적을 명시|

```kotlin
// 명령형 filter
fun size5Filter(list: List<Int>): List<Int> {
	val newList = arrayListOf<Int>()
	list.forEach {
		if (it < 5) newList.add(it)
	}
	return newList
}

// 함수형 filter
fun size5Filter(list: List<Int>): List<Int> = list.filter { it < 5 }
```

##### 즉, 함수형 프로그래밍은 주어진 데이터에서 새로운 데이터를 반환하는 함수의 연속이라고 볼 수 있습니다.

<br><br><br><Br>
#### 이제 다시 돌아와서 Rx에 대해서 살펴보겠습니다.

**Observable**이 이벤트를 발행하면 **Observer**가 구독한다.<br>
여기서 중요한 키워드는 **Observable**과 **Observer** 인데 각각 의미를 살펴보겠습니다.

##### Observable?
* 이벤트 발행 주체
* Observer를 구독시키면 (`subscribe()`)
* 이벤트 발생 시 구독중인 Observer의 `onNext()` 를 수행
* 타입
  * Observable: 최상위 기본 타입
  * Single: 1개의 데이터만 반환
  * Maybe: Null 가능성이 있는 1개의 데이터를 반환
  * Completable: 반환값 없이 수행 후 종료
  * Flowable: Backpressure

```kotlin
Observable											// Observable 객체
	.just(true) 									// 발행할 Item 등록
	.subscribeOn(Schedulers.io()) 					// Observable이 작업 수행될 thread를 지정, 중복될 경우 맨 마지막 것을 실행
	.observeOn(AndroidSchedulers.mainThread()) 		// Observable 이후의 operator, subscribe의 스케쥴러 설정
	.subscribe { res: Boolean ->
		print("res is : $res") 						// Observer의 onNext 를 수행
	}
```

Observable 각 타입에 대해 자세히 알아보겠습니다.

##### Observable 타입
Rx의 기본적인 단위입니다. Observable로 부터 발생되는 이벤트는 `onNext`, `onError`, `onComplete` 세가지로 각 이벤트들이 발생되는 케이스는 아래와 같습니다.<br>
* onNext: 데이터가 발생되었을 때 호출
* onError: 스트림 처리 또는 데이터 발생 과정 중 에러 발생했을 때
* onComplete: Observable에 대한 작업이 끝나고 스트림이 에러 없이 정상적으로 닫혔을 때 호출

```kotlin
// TODO 예제 추가 필요.
```

##### Flowable 타입
Rx2에서 새롭게 생긴 타입입니다. Observable을 사용하다보면 데이터를 생산하는 속도를 subscribe하여 소비하는 속도를 따라잡지 못하는 경우가 있습니다. 이런 경우에 발생한 데이터가 누락되거나 OOM이 발생될 수 있습니다. 그래서 기준에는 Observable에 Backpressure Buffer를 두었고 이 버퍼에 넘치는 데이터를 보관하고 버퍼가 가득찼을 경우 새로운 데이터를 publish를 하지 않도록 하는 방식으로 사용되었습니다.

하지만 이러한 Backpressure Buffer를 Rx를 잘 모르는 초보자들에게는 의도하지 않은 동작이 일어날 수 있다고 판단하게 되어 Observable에서 Backpressure를 없애고 Flowable을 대신 추가하였습니다.

Flowable에서는 아래의 5가지 BackpressureSTrategy를 통해 배압 문제를 다룰 수 있습니다.
* BUFFER: 처리할 수 없어서 넘치는 데이터를 별도의 버퍼에 저장
* DROP: 처리할 수 없어서 넘치는 데이터를 무시 (소비자에게 전달도 하지 않음)
* LATEST: 넘치는 데이터를 버퍼에 저장하지만 버퍼가 찰 경우 오래된 데이터를 무시하고 최신의 데이터만 유지
* ERROR: 넘치는 데이터가 버퍼 크기를 초과하면 MissingBackPressureException 을 통해 통지
* NONE: 특정 처리 수행 하지 않음

```kotlin
// RxJava1
Observable.from(...)
	.onBackpressureBuffer()
	.subscribe({...})

// RxJava2
Flowable.fromIterable(...)
	.onBackpressureBuffer()
	.subscribe({...})

// RxJava2 Observable.toFlowable
Observable.fromIterable(...)
	.toFlowable(BackpressureStrategy.BUFFER)
	.subscribe({...})
```
Backpressure Buffer를 관리하는 것을 더 편리하게 해주기 위해 Flowable 개념이 추가되었지만 여전히 배압관련 문제는 까다롭기 때문에 충분히 숙지 후에 사용해야 될 것 같습니다.

##### Single 타입
Observable은 0..N개의 데이터를 발생시킵니다. 하지만 대부분의 복잡하지 않은 비동기 작업들은 보통 1개의 데이터만 발생시키는 경우가 많습니다. 이러한 경우에 좀 더 편리하게 다루기 위해 RxJava2에서는 **Single**과 **Completable**이 등장하게 되었습니다.

Single은 Observable의 한 종류로 무한대의 값을 배출시킬 수 있는 Observable과 달리 작업을 수행한 뒤에 **하나의 데이터**만 발생시킬 수 있습니다. 즉, 작업이 성공했을 때 결과값을 배출시키는 `onSuccess` 와 작업을 실패 혹은 에러가 발생했을 때 에러를 배출시키는 `onError` 두가지 메소드를 사용할 수 있습니다.

```kotlin
// Firestore에서 특정 User에 대한 정보를 조회하는 예제
Single.create { emitter ->
            firestoreDb.collection(DataConst.USERS)
                .whereEqualTo(DataConst.USER_ID, userId)
                .get()
                .addOnSuccessListener { querySnapshot ->
                    emitter.onSuccess(querySnapshot)
                }
                .addOnFailureListener {exception ->
                    emitter.onError(exception)
                }
        }
```

##### Completable 타입
별도로 **발생시키는 데이터 없이** 작업의 성공, 실패 여부만 전파합니다. 따라서 작업이 성공했을 때 `onComplete`, 작업에 실패했을 때 `onError` 두가지만 가집니다.

```kotlin
// Firestore에 User 데이터를 추가하는 예제
Completable.create { emitter ->
            firestoreDb.collection(DataConst.USERS)
                .document(user.id)
                .set(user)
                .addOnSuccessListener {
                    emitter.onComplete()
                }
                .addOnFailureListener { e ->
                    emitter.onError(e)
                }
        }
```

##### Maybe 타입
이름 그대로 값이 배출될 수도 있고 배출되지 않을 수도 있는 경우에 사용되는 타입입니다. Single과 Completable 두가지가 합쳐졌다고 생각하면 될 것 같습니다. 따라서 성공하여 값이 발생했을 때 `onSuccess`, 성공했지만 값이 없을 때 `onComplete`, 실패했을 때 `onError` 세가지를 사용할 수 있습니다.

```kotlin
Maybe.just("Hello World")
            .delay(10, TimeUnit.SECONDS, Schedulers.io())
            .subscribe(
                { str ->
                    // onSuccess
                    Log.d(TAG, "onSuccess() | $str")
                },
                {e ->
                    // onError
                    Log.d(TAG, "onError()", e)
                },
                {
                    // onComplete
                    Log.d(TAG, "onComplete()")
                }
            )
```



###### 참고 사이트
* [WIKIPEDIA: Reactive Programming](https://en.wikipedia.org/wiki/Reactive_programming)
* [WIKIPEDIA: Functional Programming](https://en.wikipedia.org/wiki/Functional_programming)
* [박스여우: RxJava2의 5가지 Ovservable](https://boxfoxs.tistory.com/396)
* [꿀맛코딩: What is BackPressure? (BackPressure 란?)](https://sweetcoding.tistory.com/61)
