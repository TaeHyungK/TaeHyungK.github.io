---
layout: post
title:  "[RxJava2] 기본 연산자"
categories: [Android, RxJava2]
tags: [Kotlin]
---

### RxJava2의 기본 연산자

#### RxJava에서 사용되는 기본적인 연산자에 대해서 알아보자.






##### just()
인자로 넣은 데이터를 차례로 발행하려고 `Observable`을 생성합니다.

(단, 타입은 모두 같아야 하고 여러개의 최대 넣을 수 있는 데이터의 개수는 10개 입니다.)

```kotlin
Observable.just(1,2,3,4,5)
    .subscribe(System.out::println)

// 결과
1
2
3
4
5
```

##### subscribe()
subscribe는 실제 데이터를 발행하게끔 하는 함수로 Observable은 `just()` 같은 함수로 데이터 흐름을 정의만 하고, 실제 데이터가 발행이 되는건 **subscribe가 호출된 이후** 입니다.

인자의 갯수별로 subscribe은 달라집니다.
* 인자 0개: `onNext`와 `onComplete` 이벤트를 무시하고 `onError` 이벤트가 발생했을 때만 `onErrorNotImplementedException`을 던집니다.
* 인자 1개: `onNext`를 처리합니다.
* 인자 2개: `onNext`와 `onError` 이벤트를 처리합니다.
* 인자 3개: `onNext`와 `onError`, `onComplete` 이벤트를 처리합니다.

##### dispose()
dispose는 Observable에게 더 이상 데이터를 발행하지 말라고 구독을 해지하는 함수입니다. 일반적으로는 onComplete 이벤트가 정상적으로 발생하면 구독자가 별도의 `dispose()`를 호출하지 않아도 됩니다.

##### create()
`just()` 함수는 데이터를 넣으면 자동으로 이벤트가 발생하지만, `create()` 함수는 `onNext`, `onComplete`, `onError` 같은 이벤트를 직접 호출해야 합니다.

RxJava의 javadocs에 따르면 `create()` 는 RxJava에 익숙한 사용자만 활용하도록 권고합니다. 이유는 개발자가 하나하나 수동으로 잡아줘야 하는 옵션들이 많아지기 때문입니다.

```kotlin
Observable.create { emitter ->
    emitter.onNext(100)
    emitter.onNext(200)
    emitter.onNext(300)
    emitter.onComplete()
}.subscribe { println(it) }

// 결과
100
200
300
```

##### map()
입력값을 **어떤 함수**에 넣어서 원하는 값으로 변환하는 함수입니다.

```kotlin
val arrays = arrayOf("1", "2", "3", "4", "5")

Observable.fromArray(*arrays)
    .map { item -> "$item !!!"}
    .subscribe(System.out::println)

// 결과
1 !!!
2 !!!
3 !!!
4 !!!
5 !!!
```

##### flatMap()
`map()`과 비슷하지만 리턴타입이 Observable로 반환되기 때문에 여러개의 결과를 반환할 수 있습니다.

```kotlin
val arrays = arrayOf("1", "2", "3")

Observable.fromArray(*arrays)
    .flatMap { item -> Observable.just("$item !!!", "!!! $item")}
    .subscribe(System.out::println)

// 결과
1 !!!
!!! 1
2 !!!
!!! 2
3 !!!
!!! 3
```


###### 참고 사이트
* [Jungwoon Blog: RxJava2 정리 #1 - Observable과 기본 연산자](https://jungwoon.github.io/rxjava2/2019/07/05/RxJava-1/)
