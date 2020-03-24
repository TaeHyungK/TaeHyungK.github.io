---
layout: post
title:  "[Android] 코틀린 고차 함수"
categories: Android
tags: Kotlin
author: TaeHyungK
---

* content
{:toc}

> Kotlin IN ACTION(출판사: 에이콘) 책을 통해 Kotlin을 배워보자

## 고차 함수: 파라미터와 반환 값으로 람다 사용

- 함수 타입
- 고차 함수와 코드를 구조화할 때 고차 함수를 사용하는 방법
- 인라인 함수
- 비로컬 return과 레이블
- 무명 함수

### OverViews

람다를 인자로 받거나 반환하는 함수인 **고차 함수(high order function)**를 만드는 방법을 다루어 보자.

고차 함수로 코드를 더 간결하게 다듬고 코드 중복을 없애고 더 나은 추상화를 구축하는 방법을 살펴본다.

또한 람다를 사용함에 따라 발생할 수 있는 성능상 부가 비용을 없애고 람다 안에서 더 유연하게 흐름을 제어할 수 있는 코틀린 특성인 인라인(inline) 함수에 대해 배워보자.







#### 고차 함수 정의

- **고차 함수**는 다른 함수를 인자로 받거나 함수를 반환하는 함수다. 코틀린에서는 람다나 함수 참조를 사용해 함수를 **값**으로 표현할 수 있다. 따라서, 고차 함수는 람다나 함수 참조를 인자로 넘길 수 있거나 람다나 함수 참조를 반환하는 함수다.
- e.g.) 표준 라이브러리 함수 `filter`는 술어 함수를 인자로 받으므로 고차 함수다.
  - ```kotlin
    list.filter { x > 0 }
    ```

##### 함수 타입

```kotlin
// 타입 추론
val sum = {x: Int, y: Int -> x + y}
val action = { println(42) }

// 함수 타입 명시
val sum: (Int, Int) -> Int = {x, y -> x + y}
val action: () -> Unit = { println(42) }
```
- `->` 좌측: 함수의 파라미터을 괄호 안에 명시
- `->` 우측: 함수의 반환 타입을 명시
- 함수 타입을 선언할 때는 반환 타입을 반드시 명시해야 한다. (Unit도 마찬가지!)
- 함수 타입을 지정할 경우 람다의 파라미터 타입을 유추할 수 있으므로 람다 내에서 파라미터 타입을 생략할 수 있다.

```kotlin
// 반환 타입이 널이 될 수 있는 타입
var canReturnNull: (Int, Int) -> Int? = { x,y -> null }

// 함수 타입 변수 자체가 널이 될 수 있는 타입
var funOrNull: ((Int, Int) -> Int)? = null
```
- 함수 타입에서도 반환 타입이 널이 될 수 있는 타입으로 지정할 수 있다.
- 함수 타입 변수 자체를 널이 될 수 있는 타입으로도 사용이 가능하다.
- 코드 상으로는 단순히 괄호의 유무의 차이이지만 **반환 타입이 널이 될 수 있는 타입** 과 **함수 타입 변수 자체가 널이 될 수 있는 타입**은 큰 차이가 있다는 것을 유의해야 한다.

> **파라미터 이름과 함수 타입**
> ```kotlin
> fun performRequest (url: String, 
>                     callback: (code: Int, conetnt: String) -> Unit) {
>     /* TODO */
> }
> 
> val url = "http://kotl.in"
> performRequest(url) { code, content -> /* TODO */ } // API에서 제공하는 이름을 람다에서 사용
> performRequest(url) { code, page -> /* TODO */ } // 원하는 이름으로 붙여서 사용
> ```
> - 파라미터 이름은 타입 검사시 무시된다.
> - 람다에서 호출하는 파라미터 이름이 꼭 함수 타입 선언의 파라미터 이름과 일치하지 않아도 된다.
> - 함수 타입에 인자 이름을 추가하는 것이 가독성과 IDE를 사용함에 있어서 더 좋기 때문에 파라미터 이름을 사용하는 것을 권장한다. 


##### 인자로 받은 함수 호출

```kotlin
fun twoAndThree(operation: (Int, Int) -> Int) {
    val result = operation(2, 3)
    println("The result is $result")
}

twoAndThree { a, b -> a + b}
// 결과: The result is 5
twoAndThree() { a, b -> a * b }
// 결과: The result is 6
```
- **인자로 받은 함수를 호출하는 구문**은 일반 함수를 호출하는 구문과 같다.

> String에 대한 filter 구현 해보기
>
> ```kotlin
> fun String.filter(predicate: (Char) -> Boolean): String {
>     val sb = StringBuilder()
>     for (index in 0 until length) {
>         val element = get(index)
>         if (predicate(element)) sb.append(element)
>     }
>     return sb.toString()
> }
> 
> println("ab1c".filter {it in 'a'..'z'})
> // 결과: abc
> ```
> - `String`: 수신 객체 타입
> - `predicate`: 파라미터 이름
> - `(Char) -> Boolean`: 파라미터 함수 타입
>  - `(Char)`: 파라미터로 받는 함수의 파라미터 타입
>  - `Boolean`: 파라미터로 받는 함수의 반환 타입 

##### 자바에서 코틀린 함수 타입 사용

컨파일된 코드 안에서 함수 타입은 **일반 인터페이스**로 바뀐다.

함수 타입의 변수는 **FunctionN 인터페이스**를 구현하는 객체를 저장한다.
- 코틀린 표준 라이브러리는 함수 인자의 개수에 따라 Function0<R> (인자가 없는 함수), Function1<P1, R>(인자가 하나인 함수) 등의 인터페이스를 제공한다.
- 각 인터페이스에는 `invoke()` 메소드 정의가 하나 들어있으며 `invoke()`를 호출하면 함수를 실행할 수 있다.
  - 함수 타입인 변수는 FunctionN 인터페이스를 구현하는 클래스의 인스턴스를 저장
  - 그 클래스의 `invoke()` 메소드 본문에는 람다의 본문을 저장

말로 보면 어려우니 코드로 자바에서 코틀린의 함수 타입을 사용해보자. (자바8 에서!)
```kotlin
// 코틀린에서 선언
fun processTheAnswer(f: (Int) -> Int) {
    println(f(42))
}
```
```java
// 자바 8 에서 위 코틀린 코드 호출
processTheAnswer(number -> number + 1);
// 결과: 43

//자바 8 이전에서 위 코틀린 코드 호출
processTheAnswer(
    new Function1<Integer, Integer>() {
        @Override
        public Integer invoke(Integer number) {
            System.out.println(number);
            return number + 1;
        }
});
// 결과: 43
```
- 자바 8 이후의 버전은 람다를 그대로 넘기면 자동으로 함수 타입의 값으로 변환된다.
- 하지만 자바 8 이전의 버전인 경우 필요한 FunctionN 인터페이스의 `invoke()` 메소드를 구현하는 무명 클래스를 넘겨주어야 한다.

```java
// 코틀린 표준 라이브러리르 이용하여 코틀린의 람다를 인자로 받는 확장 함수 호출
List<String> strings = new ArrayList<>();
strings.add("42");
CollectionsKt.forEach(strings, s -> { // strings는 확장 함수의 수신 객체
    System.out.println(s)
    return Unit.INSTANCE; // Unit 타입의 값을 명시적으로 반환해야 함
});
```
- 코틀린 Unit 타입에는 값이 존재하므로 자바에서는 그 값을 명시적으로 return 해주어야 한다.
- 즉, `(String) -> Unit` 처럼 반환 타입이 Unit인 경우 void를 반환하는 자바 람다를 넘길 수 없다.

##### 디폴트 값을 지정한 함수 타입 파라미터나 널이 될 수 있는 함수 타입 파라미터

3장에서 살펴본 joinToString 함수를 함수 타입의 파라미터에 대한 디폴트 값을 지정하여 가공해보자.
```kotlin
fun <T> Collection<T>.joinToString(separator: String = ", ",
                                    prefix: String = "",
                                    postfix: String = "",
                                    transform: (T) -> String = { it.toString() } // 함수 타입 파라미터를 선언하면서 람다를 디폴트 값으로 지정
): String {
    val result = StringBuilder(prefix)
    for ((index, element) in this.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(transform(element))
    }
    result.append(postfix)
    return result.toString()
}

val letters = listOf("Alpha", "Beta")

// 디폴트 람다 사용
println(letters.joinToString())
// 결과: Alpha, Beta

// 직접 람다 전달
println(letters.joinToString { it.toLowerCase() })
// 결과: alpha, beta

// 이름이 붙은 인자 구문을 사용해 여러 인자를 전달
println(letters.joinToString(separator = "! ", postfix = "! ", transform = { it.toUpperCase() }))
//결과: ALPHA! BETA!
 
```

위와 다른 방법으로 널이 될 수 있는 함수 타입을 사용할 수 있다.

다만, 널이 될 수 있는 함수 타입을 사용하는 경우 그 함수를 직접 호출할 수 없다. 또한, NPE 발생 가능성이 있으므로 컴파일을 거부하게 된다

그런 경우 null 여부를 명시적으로 검사하면 된다.
```kotlin
fun foo(callback: (() -> Unit)?) {
    // TODO
    if (callback != null){
        callback()
    }
}
```
- 전에 말했듯이 함수 타입은 `invoke()` 메소드를 구현하는 인터페이스다.
- 이 사실을 기억하면 일반 메소드처럼 `invoke()` 도 안전한 호출 구문(`callback?.invoke()`)으로 호출할 수 있다는 것을 알 수 있다.

```kotlin
fun <T> Collection<T>.joinToString(separator: String = ", ", 
                                    prefix: String = "",
                                    postfix: String = "",
                                    transform: ((T) -> String)? = null // 널이 될 수 있는 함수 타입의 선언
): String {
    val result = StringBuilder(prefix)
    for ((index, element) in this.withIndex()) {
        if (index > 0) result.append(separator)
        val str = transform?.invoke(element) ?: element.toString()
        result.append(str)
    }
    result.append(postfix)
    return result.toString()
}
```

##### 함수를 함수에서 반환

```kotlin
enum class Delivery { STANDARD, EXPEDITED }
class Order(val itemCount: Int)
fun getShippingCostCalculator(delivery: Delivery): (Order) -> Double {
    if (delivery == Delivery.EXPEDITED) {
        return { order -> 6 + 2.1 * order.itemCount }
    }

    return { order -> 1.2 * order.itemCount}
}

val calculator = getShippingCostCalculator(Delivery.EXPEDITED)
println("Shipping costs ${calculator(Order(3))}")
// 결과: Shipping costs 12.3
```
- 다른 함수를 반환하는 함수를 정의하려면 함수의 반환 타입으로 함수 타입을 지정해야 한다. 
  - 타입 추론을 믿고 생략할 경우 `error: type mismatch: inferred type is (???) -> ??? but Unit was expected` 컴파일 에러를 맞이하게 된다.

```kotlin
data class Person(val firstName: String, val lastName: String, val phoneNumber: String?)
class ContactListFilters {
    var prefix: String = ""
    var onlyWithPhoneNumber: Boolean = false
    fun getPredicate(): (Person) -> Boolean { // 함수를 반환하는 함수를 정의
        val startsWithPrefix = { p: Person -> 
            p.firstName.startsWith(prefix) || p.lastName.startsWith(prefix) 
        }
        if (!onlyWithPhoneNumber) {
            return startsWithPrefix
        }
        return { startsWithPrefix(it) && it.phoneNumber != null } // 람다를 반환
    }
}

val contacts = listOf(Person("Dmitry", "Jemerov", "123-4567"), Person("Svetlana", "Isakova", null))
val contactListFilters = ContactListFilters()
with (contactListFilters) {
    prefix = "Dm"
    onlyWithPhoneNumber = true
}
println(contacts.filter(contactListFilters.getPredicate()))
// 결과: [Person(firstName=Dmitry, lastName=Jemerov, phoneNumber=123-4567)]
```
- `getPredicate()` 메소드는 `filter` 함수에 넘길 수 있는 함수를 반환한다.
- 이처럼 함수 타입을 사용하면 함수에서 함수를 쉽게 반환할 수 있다.

> **다시 기억하자!**
> 
> with()
> - with 함수는 파라미터가 2개인 메소드로 첫 번째 인자는 객체를 두 번째 인자는 람다를 받는다.
> - 첫 번째 인자로 받은 객체를 두 번째 인자로 받은 람다의 수신 객체로 만든다.
>
> 태형 Think !
> - 위의 with() 함수가 어떻게 동작하는 것인지 이해가 가지 않는다.. 이야기 해보자..!

##### 람다를 활용한 중복 제거

```kotlin
data class SiteVisit(val path: String, val duration: Double, val os: OS)

enum class OS { WINDOWS, LINUX, MAC, IOS, ANDROID }

val log = listOf(SiteVisit("/", 34.0, OS.WINDOWS), 
    SiteVisit("/", 22.0, OS.MAC),
    SiteVisit("/login", 12.0, OS.WINDOWS),
    SiteVisit("/signup", 8.0, OS.IOS),
    SiteVisit("/", 16.3, OS.ANDROID))

// 윈도우 사용자의 평균 방문 시간 가져오기
val averageWindowsDuration = log.filter { it.os == OS.WINDOWS }
                                .map(SiteVisit::duration)
                                .average()
println(averageWindowsDuration)
// 결과: 23.0

// OS별 사용자 평균 방문 시간 가져오는 확장 함수 정의
fun List<SiteVisit>.averageDurationFor(os: OS) = 
    filter { it.os }.map(SiteVisit::duration).average()

println(log.averageDurationFor(OS.WINDOWS))
// 결과: 23.0
println(log.averageDurationFor(OS.MAC))
// 결과: 22.0

// 하드코딩한 필터를 사용해 방문 데이터 분석하기
val averageMobileDuration = log.filter { it.os in setOf(OS.IOS, OS.ANDROID) }
                                .map(SiteVisit::duration)
                                .average()
println(averageMobileDuration)
// 결과: 12.15

// 고차 함수를 사용해 중복 제거하기
fun List<SiteVisit>.averageMobileDurationFor(predicate: (SiteVisit) -> Boolean) =
    filter(predicate).map(SiteVisit::duration).average()

```
- 확장 함수를 정의하며 가독성이 좋아진 것을 볼 수 있다.
- 함수 타입은 코드 중복을 줄일 때 상당한 도움이 된다.
- 함수 타입을 통해 [전략 패턴(Strategy Pattern)](https://tourspace.tistory.com/68)을 손쉽게 적용할 수 있다. 

#### 인라인 함수: 람다의 부가 비용 없애기

리마인드 할 내용
- 보통 람다를 무명 클래스로 컴파일하지만 그렇다고 람다 식을 사용할 때마다 새로운 클래스를 만들지는 않는다.
- 람다가 변수를 포획하면 람다가 생성되는 시점마다 새로운 무명 클래스 객체가 생긴다.

리마인드 할 내용에 따르면 무명 클래스 생성에 따른 부가 비용이 발생한다. 따라서 일반 함수를 사용한 구현보다 **덜** 효율적이다.

`inline` 변경자를 어떤 함수에 붙이면 컴파일러는 그 함수를 호출하는 모든 문장을 함수 본문에 해당하는 바이트코드로 바꿔치기 해주며 그로 인해 일반 명령문만큼 효율적인 코드를 생성하게 할 수 있다.

##### 인라이닝이 작동하는 방식

`inline`: 함수를 호출하는 코드를 **함수를 호출하는 바이트코드** 대신에 **함수 본문을 번역한 바이트 코드**로 컴파일한다는 뜻

```kotlin
// 인라인 함수 정의하기
inline fun <T> synchronized(lock: Lock, action: () -> T): T {
    lock.lock()
    try {
        return action()
    } finally {
        lock.unlock()
    }
}
val l = Lock()
synchronized(l) {
    // TODO
}
```

> 위 코드에 대한 의문
> - kotlin 에서는 `try catch` 구문에서 `catch`를 생략해도 되는 것인가?


```kotlin
fun foo(l: Lock) {
    println("Before sync")
    synchronized(l) {
        println("Action")
    }
    println("After sync")
}
```

을 컴파일 하면?

```kotlin
fun __foo__(l: Lock) {
    println("Before sync")
    l.lock() // synchronized 함수가 인라이닝된 코드 시작
    try {
        println("Action")  // 람다 코드의 본문이 인라이닝된 코드
    } finally {
        l.unlock()
    }        // synchronized 함수가 인라이닝된 코드 끝
    println("After sync")
}
```
- synchronized 함수의 본문뿐 아니라 synchronized에 전달된 **람다의 본문도 함께 인라이닝**된다.

```kotlin
class LockOwnser(val lock: Lock) {
    fun runUnderLock(body: () -> Unit) {
        synchronized(lock, body)
    }
}
```
- 위의 코드의 경우 인라인 함수를 호출하는 코드 위치에서는 변수에 저장된 람다의 코드를 알 수 없다.
- 따라서 람다 본문은 인라이닝되지 않고 synchronized 함수의 본문만 인라이닝된다.

위 코드를 컴파일 하면?

```kotlin
class LockOwner(val lock: Lock) {
    fun __runUnderLock__(body: () -> Unit) {
        lock.lock()
        try {
            body()
        } finally {
            lock.unlock()    
        }   
    }
}
```

정리 사항
- 한 인라인 함수를 두 곳에서 각각 다른 람다를 사용해 호출한다면 그 두 호출은 각각 따로 인라이닝된다.
- 인라인 함수의 본문 코드가 호출 지점에 **복사**되고 각 람다의 본문이 인라인 함수의 본문 코드에서 람다를 사용하는 위치에 **복사**된다.

// TBD.. 


###### 출처

- Kotlin IN Action / 출판사: 에이콘
  