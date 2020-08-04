---
layout: post
title:  "[RxJava2] Cold Observable과 Hot Observable, 그리고 Subject"
categories: Android RxJava2
tags: Kotlin
author: TaeHyungK
---

* content
{:toc}

### Hot Observable VS Cold Observable

간단하게 말하자면 Hot Observable은 뜨거워서 데이터를 바로 흘려 보내는 Observable 이고,

Cold Observable은 꽁꽁 얼어 붙어 있어 subscribe을 해야만 흘려 보내는 Observable 입니다.






#### Hot Observable

Observable을 생성하자마자 아이템들을 흘려보낸다는 의미입니다.

그래서, Observable을 생성하고 일정 시간이 지난 상태에서 subscribe하게 된다면
처음부터 아이템을 받아보지는 못하고, 중간부터 나온 아이템들 부터
subscribe 할 수 있게 됩니다.

Hot Observable에 대해 정리해보면,
* 구독자의 존재 여부와 상관없이 데이터를 배출하는 Observable 이다.
* 그래서 여러 구독자에 선택적으로 고려가 가능하다.
* **구독 시점**으로부터 발행하는 값을 받는 걸 기본으로 한다.
* 마우스 이벤트, 키보드 이벤트, 시스템 이벤트 등이 주로 사용된다.
* 멀티캐스팅도 포함된다.

<br><br><br>

#### Cold Observable
이름에서부터 Hot Observable과 반대로 바로 흘려보내지 않는다는 의미를 추측할 수 있습니다.

그렇다면 언제 흘려 보낸다는 걸까요?

바로 Subscribe 할 때, 아이템들을 흘려보내게 됩니다.

다음으로 Cold Observable에 대해 정리해보면,

* 일반적인 옵저버 형태를 말한다.
* 누가 구독해주지 않으면 데이터를 배출해주지 않는다.
* 일반적인 웹 요청, 데이터베이스 쿼리 등이 사용되며 내가 요청하면 결과를 받는 과정을 거친다.
* **처음부터 발행하는 것**을 기본으로 한다.

<br><br><br>

**Hot Observable VS Cold Observable**

**Cold Observable**은 스트림을 분기시키는 성질을 가지고 있지 않습니다.
따라서, Cold Observable을 여러번 Subscribe 하는 경우,
각각 별도의 스트림이 생성되고 할당되게 됩니다.

그러나, **Hot Observable**은 스트림을 분기시키는 성질을 가지고 있기 때문에
스트림의 분기가 필요한 경우 Hot Observable을 사용해야 합니다.


##### 즉, **하나의 스트림을 여러번 Subscribe 해야 하는 경우**에 Hot Observable을 사용하게 됩니다.


그렇다면, 일반적인 Observable(== Cold Observable)을 어떻게 Hot Observable로 바꾸는 것일까?


#### 정답은 바로 `ConnectableObservable` 과 `Subject` 클래스입니다.

### ConnectableObservable

ConnecatableObservable은 Cold Observable을 Hot Observable로 변환이 가능한 옵저버블입니다.

해당 옵저버블의 가장 큰 특징은 데이터 배출 시 `subscribe()` 메소드를 사용하기에 앞서 `connect()` 메소드를 호출하여 Hot Observable로 활성화 하여 사용이 가능하다는 점입니다.

다음 예시를 보겠습니다.

```kotlin
val connectableObservable = listOf("String 1","String 2","String 3","String 4","String 5").toObservable()
            .publish()//1

connectableObservable.
        subscribe { println("Subscription 1: $it") }//2

connectableObservable.map(String::reversed) // 3
        .subscribe { println("Subscription 2 $it")}//4

connectableObservable.connect()//5

connectableObservable.
        subscribe { println("Subscription 3: $it") } //6 구독을 받지못함

// 결과는
Subscription 1: String 1
Subscription 2 1 gnirtS
Subscription 1: String 2
Subscription 2 2 gnirtS
Subscription 1: String 3
Subscription 2 3 gnirtS
Subscription 1: String 4
Subscription 2 4 gnirtS
Subscription 1: String 5
Subscription 2 5 gnirtS
```
위 예시에서 1번 영역을 보면 `publish()` 라는 메소드가 있는데 해당 메소드를 이용하여 Cold Observable을 Hot Observable로 변환이 되게 됩니다.<br> 이후에 5번의 `connect()` 메소드를 호출하여 2,4번의 옵저버에서 모두 데이터가 처리되는 것을 볼 수 있습니다.

<br>
다음으로 Cold Observable을 Hot Observable로 변환할 수 있는 또다른 방법인 Subject 클래스에 대해서 알아보겠습니다.


#### Subject

앞서 말씀 드렸던 것 처럼, `Subject` 클래스는 Cold Observable을 Hot Observable로 변환시켜 주며 **Observable 속성** 과 **Subscriber 속성** 모두 가지고 있습니다.

Subject에는 여러 종류가 있는데, 다음으로 Subject의 종류에 대해 알아보겠습니다.

<br><br><br>

##### AsyncSubject 클래스

AsyncSubject 클래스는 Observable 에서 발행한 마지막 데이터를 얻어올 수 있는 Subject 클래스 입니다.

완료되기 전 마지막 데이터에만 관심이 있으며 이전 데이터는 무시합니다.
즉, `onComplete()` 되기 전 마지막 `onNext()`의 값만 처리합니다.

```kotlin
val subject: AsyncSubject<String> = AsyncSubject.create()
subject.subscribe { data ->
    Log.d("Subscriber #1", "data: $data")
}
subject.onNext("1")
subject.onNext("3")
subject.subscribe { data ->
    Log.d("Subscriber #2", "data: $data")
}
subject.onNext("5")
subject.onComplete()

// data는 "5"만 전달됨
```

```kotlin
val tempArray = arrayOf(10, 18, 27)
val source = Observable.fromArray(tempArray)

val subject = AsyncSubject.create<Array<Int>>()
subject.subscribe { data ->
    Log.d("Subscriber #1", "data: $data")
}
source.subscribeWith(subject)

// data는 27 만 찍힘
```

<br><br><br>

##### BehaviorSubject 클래스

BehaviorSubject 클래스는 구독자가 구독을 하면 가장 최근 값 혹은 기본값을 넘겨주는 클래스입니다.

```kotlin
val subject = BehaviorSubject.createDefault("6")
subject.subscribe { data ->
    Log.d("Subscriber #1", "data: $data")
}
subject.onNext("1")
subject.onNext("3")
subject.subscribe { data ->
    Log.d("Subscriber #2", "data: $data")
}
subject.onNext("5")
subject.onComplete()

// 결과는
Subscriber #1: data: 6 // "1" 발행 전 기본값 전달
Subscriber #1: data: 1
Subscriber #1: data: 3
Subscriber #2: data: 3 // subscribe 하기 직전 (가장 최근) 값 전달
Subscriber #1: data: 5
Subscriber #2: data: 5
```

<br><br><br>

##### PublishSubject 클래스

PublishSubject 클래스는 가장 평범한 Subject 클래스입니다.
구독자가 `subscribe()` 함수를 호출하면 값을 발행하기 시작합니다.
해당 시간에 발생한 데이터를 그대로 구독자에게 전달하게 됩니다.

<br><br><br>

##### ReplaySubject 클래스

ReplaySubject 클래스는 가장 특이하고 사용할 때 주의가 많이 필요한 클래스입니다. <br>
앞서 학습한 바에 의하면 Subject 는 Hot Observable로 변환하여 사용하기 위한 것인데 이 클래스는 Cold Observable 처럼 동작하기 때문입니다.

Replay Subject는 구독자가 새로 생기면 항상 데이터의 처음부터 끝까지 발행해주는 클래스입니다. <br>
그러므로 모든 데이터 내용을 저장해두는 과정에서 **메모리 누수가 발생할 수 있다는 가능성**을 염두에 두고 사용해야 합니다.

```kotlin
val subject = ReplaySubject.create<String>()
subject.subscribe { data ->
    Log.d("Subscriber #1", "data: $data")
}
subject.onNext("1")
subject.onNext("3")
subject.subscribe { data ->
    Log.d("Subscriber #2", "data: $data")
}
subject.onNext("5")
subject.onComplete()

// 결과는
Subscriber #1: data: 1
Subscriber #1: data: 3
Subscriber #2: data: 1
Subscriber #2: data: 3
Subscriber #1: data: 5
Subscriber #2: data: 5
```


###### 참조
* [올해는 머신러닝이다.: [RxJava2] cold Observable? Hot Observable? 개념 설명](https://javaexpert.tistory.com/794)
* [Developer88: Hot Observable 과 Cold Observable은 무엇인가요?](https://developer88.tistory.com/86)
* [Progmming Summary Notes: Rx의 Hot과 Cold에 대해](http://lonpeach.com/2019/09/29/UniRx-Hot-Cold/#hot%EA%B3%BC-cold-%EA%B5%AC%EB%B6%84%EB%B2%95)
* [Progmming Summary Notes: [Reactive Extensions] Hot 변환은 어떤 때에 필요한가?](http://lonpeach.com/2019/10/13/UniRx-When-is-a-Hot-Conversion/)
* [범석의 안드로이드 메모장: Observable -12 ( 뜨거운 Observable)](https://beomseok95.tistory.com/15)
* [돼지왕 왕돼지 놀이터: [RxJava] #2 Observable 처음 만들기](https://aroundck.tistory.com/6223)
* [3-02. 핫 옵저버블 & 콜드 옵저버블 - gists](https://gist.github.com/SODA1127/7ce11958f0914a7d09fb5ba68720c364)
