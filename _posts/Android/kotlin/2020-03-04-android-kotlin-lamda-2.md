---
layout: post
title:  "[Kotlin] 람다로 프로그래밍 2탄"
categories: [Android, Kotlin]
tags: [Kotlin]
---

> Kotlin IN ACTION(출판사: 에이콘) 책을 통해 Kotlin을 배워보자

### 람다 식 | 람다

[람다로 프로그래밍 1탄](https://taehyungk.github.io/2020/03/02/android-kotlin-lamda-1/)에 이어 2탄을 공부해보자

#### 컬렉션 함수형 API

##### 필수적인 함수: filter와 map

**filter 함수**
```kotlin
val list = listOf(1, 2, 3, 4)
println(list.filter { it % 2 == 0 }) // 짝수만 filtering
// 결과: [2, 4]

val personList = listOf(Person("Bob", 31), Person("Alice", 29))
val filterList = personList.filter { it.age > 30 }
println(filterList)
// 결과: [Person(name=Bob, age=31)]
```
- 컬렉션을 이터레이션하면서 주어진 람다에 각 원소를 넘겨 람다가 **true**인 원소를 모은다.
- 만족하는 원소들을 모아 **새로운 컬렉션**으로 반환한다.








**map 함수**
```kotlin
val list = listOf(1, 2, 3, 4)
println(list.map { it * it }) // 자기자신을 곱함
// 결과: [1, 4, 9, 16]

val personList = listOf(Person("Bob", 31), Person("Alice", 29))
val mapList = personList.map { it.age } // 나이만으로 컬렉션을 만듦
println(mapList)
// 결과: [31, 29]

// 멤버 참조 사용
val memberRefMapList = personList.map(Person::name)
println(memberRefMapList)
// 결과: [Bob, Alice]
```
- 주어진 람다를 컬렉션의 각 원소에 **적용한 결과**를 모아서 **새 컬렉션**을 만든다.

**filter + map 조합**
```kotlin
val list = listOf(Person("Bob", 31), Person("Alice", 29))
val filterAndMap = list.filter { it.age > 30 }.map { it.name }
// 결과: [Bob]
```
- 연쇄로 호출하여 사용도 가능하다.

**maxBy + filter 조합**
```kotlin
val list = listOf(Person("Bob", 31), Person("Alice", 29))
val filterAndMaxBy = list.filter { it.age == list.maxBy(Person::age)!!.age}
// 결과: [Person(name=Bob, age=31)]
```
- 위 코드의 단점은 filter가 이터레이션하기 때문에 maxby 함수가 컬렉션 수 만큼 호출되며 처리된다는 것이다.
- 개선해보면...
    ```kotlin
    val list = listOf(Person("Bob", 31), Person("Alice", 29))
    val maxAge = list.maxBy(Person::age)!!.age
    val filterAndMaxBy = list.filter { it.age == maxAge }
    ```
- 이터레이션 된다는 것을 항상 기억하고 불필요한 작업을 반복하지 않도록 유의해야 한다.

**컬렉션 맵에서의 filter, map**
```kotlin
val numbers = mapOf(0 to "zero", 1 to "one", 2 to "two", 3 to "three", 4 to "four")
val filterValuesMap = numbers.filterValues { it == "zero"}
val mapValuesMap = numbers.mapValues { it.value.toUpperCase() }
println(filterValuesMap)
println(mapValuesMap)

val filterKeysMap = numbers.filterKeys { it == 1 }
val mapKeysMap = numbers.mapKeys { it.key % 2 }
println(filterKeysMap)
println(mapKeysMap)

// 결과: filterValuesMap = {0=zero}
//      mapValuesMap = {0=ZERO, 1=ONE, 2=TWO, 3=THREE, 4=FOUR}
//      filterKeysMap = {1=one}
//      mapKeysMap = {0=four, 1=three}
```
- 맵에서의 filter와 map은 별도의 API가 존재한다.
- 맵의 `filterValues`, `filterKeys` 의 `it` 는 각각 value와 key를 가르킨다.

##### 컬렉션에 술어 사용: all, any, count, find

```kotlin
val list = listOf(Person("Alice", 27), Person("Bob", 31), Person("hzoou", 25), Person("txxbro", 28), Person("iyj", 28), Person("WooVictory", 27))

// 술어 선언
val canBeInClub27 = { p: Person -> p.age <= 27 }
println("all: ${list.all(canBeInClub27)}")
println("any: ${list.any(canBeInClub27)}")
println("count: ${list.count(canBeInClub27)}")
println("find: ${list.find(canBeInClub27)}")

// 결과: all: false
//      any: true
//      count: 3
//      find: Person(name=Alice, age=27)
```

- `all`: 컬렉션의 모든 원소가 조건을 만족하는지 판단
- `any`: 컬렉션의 모든 원소 중 하나라도 조건을 만족하는지 판단
- `count`: 조건을 만족하는 원소의 갯수를 반환
- `find`: 조건을 만족하는 첫 번째 원소를 반환, 만족하는 원소가 없을 경우 **null**을 반환

> 함수형 API 사용시 고려할 점
> - 함수형 API `count` 와 컬렉션에 포함된 함수 `size()` 의 차이?
>   - `count`의 경우 조건을 만족하는 원소의 개수만 추적할 뿐 원소를 따로 저장하지 않는다.
>   - `size()`의 경우 만족하는 원소를 가진 객체를 생성 시키게 된다.
> - 위 예제 코드의 결과에서 보듯이 all과 any는 서로 부정으로 대응한다. 하지만 가독성을 이유로 `any` 대신 `!all` 이나 `all` 대신 `!any`는 사용하지 않는 것이 좋다.

##### groupBy

```kotlin
val list = listOf(Person("Alice", 27), Person("hzoou", 25), Person("txxbro", 28), Person("iyj", 28), Person("WooVictory", 27))

println("groupBy: ${list.groupBy { it.age }}")
// 결과: groupBy: {27=[Person(name=Alice, age=27), Person(name=WooVictory, age=27)], 
//              25=[Person(name=hzoou, age=25)], 
//              28=[Person(name=txxbro, age=28), Person(name=iyj, age=28)]}

val strs = listOf("12", "345", "11", "456")
println(strs.groupBy { it.length })
// 결과: {2=[12, 11], 3=[345, 456]}
```

- groupBy: 리스트를 특정 기준에 맞춰 맵으로 변경하여 반환


##### flatMap과 flatten: 중첩된 컬렉션 안의 원소 처리

```kotlin
val strings = listOf("abc", "def")
println(strings.flatMap { it.toList() })
//결과: [a, b, c, d, e, f]

data class Book(val title: String, val authors: List<String>)

val books = listOf(Book("책1", listOf("작가1")),
                 Book("책2", listOf("작가2", "작가3")), 
                 Book("책3", listOf("작가4", "작가1")))

println("toSet(): ${books.flatMap { it.authors }.toSet()}")
println("기본: ${books.flatMap { it.authors }}")
// 결과: toSet(): [작가1, 작가2, 작가3, 작가4]
//      기본: [작가1, 작가2, 작가3, 작가4, 작가1]

println("flatten(): ${books.map { it.authors }.flatten()}")
// 결과: flatten(): [작가1, 작가2, 작가3, 작가4, 작가1]
```

- `flatMap`: 인자로 주어진 람다를 컬렉션의 모든 객체에 적용(매핑)하고 람다를 적용한 결과 얻어지는 여러 리스트를 한 리스트로 모은다(flatten). **즉, 리스트의 리스트가 있을 때 중첩된 리스트의 원소를 한 리스트로 모을 때 사용한다.**
- `toSet()`: 컬렉션의 중복을 제거리
- `flatten()`: 변환할 내용 없이 펼치기만 하는 경우 사용

> 책에서 다루지 않은 이외에도 많은 컬렉션 API가 존재한다.
> 이외의 API 는 [Kotlin Collection Reference](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/index.html) 를 참고하자.

##### 지연 계산(lazy) 컬렉션 연산

콜렉션의 연산자(e.g. map, filter)는 결과 컬렉션을 **즉시** 생성한다. 이는 컬렉션 함수를 연쇄하면 매 단계마다 계산 중간 결과를 새로운 컬렉션에 임시로 담는다는 말이다. 

`시퀀스(sequence)`를 사용하면 중간 임시 컬렉션을 사용하지 않고도 컬렉션 연산을 연쇄할 수 있다. 

```kotlin
poeple.map(Person::name).filter { it.startsWith("A") }
```
- `map`과 `filter`는 둘 다 리스트를 반환한다. 즉 위 코드에서 연쇄 호출로 인해 리스트를 2개 만들어졌다.

```kotlin
people.asSequence() // 원본 컬렉션을 시퀀스로 변환
    .map(Person::name).filter { it.startsWith("A")}
    .toList() // 결과 시퀀스를 다시 리스트로 변환
```  
- 시퀀스의 원소는 필요할 때 비로소 계산되기 때문에 중간 처리 결과를 저장하지 않고도 연산을 연쇄적으로 적용해 효율적으로 계산을 수행할 수 있다.
- `asSequence()`: 어떤 컬렉션이든 시퀀스로 바꿀 수 있다.
- `toList()`: 시퀀스를 리슷트로 바꿀 때 사용한다.

> 리스트 대신에 시퀀스를 쓰는 것이 더 낫지 않은가?
> - "항상 그렇지는 않다."
> - 인덱스를 사용해 접근하는 등 다른 API 메소드를 사용하기 위해서는 리스트로 변환해야 한다.

##### 시퀀스 연산 실행: 중간 연산과 최종 연산

중간 연산과 최종 연산은 p225의 그림 5.7을 참고하자.
- 중간 연산: 다른 시퀀스를 반환하며 최초 시퀀스의 원소를 변환하는 방법을 알고 있다.
  - **항상 지연 계산된다.** 즉, 최종 연산을 하지 않으면 계속 지연이 되어 결과를 반환하지 않는다.
- 최종 연산: 최초 컬렉션에 대해 변환을 적용한 시퀀스로부터 일련의 계산을 수행해 얻을 수 있는 컬렉션이나 원소, 숫자, 객체이다.
  
**즉시 계산의 수행 순서와 지연 계산의 수행 순서**
```kotlin
// eagerly
listOf(1, 2, 3, 4).map { println("eagerly map($it)"); it * it }
                .filter { println("eagerly filter($it)"); it % 2 == 0 }
// 결과: eagerly map(1)
//      eagerly map(2)
//      eagerly map(3)
//      eagerly map(4)
//      eagerly filter(1)
//      eagerly filter(4)
//      eagerly filter(9)
//      eagerly filter(16)

// lazy
listOf(1, 2, 3, 4).asSequence()
                .map { println("lazy map($it)"); it * it}
                .filter { println("lazy filter($it)"); it % 2 == 0 }
                .toList()
//결과: lazy map(1)
//     lazy filter(1)
//     lazy map(2)
//     lazy filter(4)
//     lazy map(3)
//     lazy filter(9)
//     lazy map(4)
//     lazy filter(16)
```
- 즉시 계산의 경우 **모든 원소**에 대해 먼저 map을 끝낸 후 이후 filter를 수행하게 된다.
- 시퀀스(지연 계산)의 경우 **각 원소**에 대해 순차적으로 적용이 된다.
- p226의 그림 5.8을 참고하자.

**map과 filter 호출 순서에 따른 성능 차이의 발생**
```kotlin
val list = listOf(Person("Alice", 27), Person("hzoou", 25), Person("txxbro", 28), Person("iyj", 28), Person("WooVictory", 27))

list.asSequence().map(Person::name) // map 먼저 실행
    .filter { it.length < 4 }.toList()

list.asSequence().filter { it.length < 4 } // filter 먼저 실행
    .map(Person::name).toList()
```

- p227의 그림 5.9를 참고하자.
  - filter 보다 map을 호출할 경우 map은 모든 원소를 변환하므로 더 많은 이터레이션이 발생하게 된다.

> **자바 스트림과 코틀린 시퀀스 비교**
> 
> 자바 8의 스트림과 코틀린의 시퀀스는 개념적으로 같다. 다만, 자바 8일 경우 코틀린 컬렉션과 시퀀스에서 제공하지 않는 **스트림 연산(map과 filter)을 여러 CPU에서 병렬적으로 실행하는 기능**이 존재한다. 그렇기 때문에 자바 버전에 따라서 시퀀스와 스트림 중에 적절한 것을 사용하면 된다.
>
> 자바 8에 대해서는 다른 개발자의 블로그의 글인 [자바 8 스트림 이란?](https://12bme.tistory.com/461) 을 참고하자.

##### 시퀀스 만들기

```kotlin
//                  첫번째 인자: 초기값 / 두번째 인자: 다음 값 생성 로직
val numbers = generateSequence(0) { it + 1 } // 시퀀스 생성
val numbersTo100 = numbers.takeWhile { it <= 100 } // while loop 시퀀스 생성
println(numbersTo100.sum()) // 위의 모든 시퀀스는 sum의 결과를 계산할 때 수행된다.
// 결과: 5050

// File의 확장함수 선언
fun File.isInsideHiddenDirectory() = generateSequence(this) { it.parentFile }.any { it.isHidden }
val file = File("/Users/svtk/.HiddenDir/a.txt")
println(file.isInsideHiddenDirectory())
// 결과: true
```
- `generateSequence`: 이전의 원소를 인자로 받아 다음 원소를 계산하는 시퀀스를 만드는 함수
- 최종 연산인 `sum()` 을 호출 하기 전에는 계산되지 않다가 최종 연산이 호출될 때에 계산이 수행된다.

#### 자바 함수형 인터페이스 활용

##### 함수형 인터페이스

추상 메소드가 단 하나 있는 인터페이스를 **함수형 인터페이스** 또는 **SAM(단일 추상 메소드, Single Abstract method) 인터페이스**라고 한다.

```java
// java 8 이전 익명클래스로 표현
button.setOnClickListener(new OnClickListener() {
    @Override
    public void onClick(View v) {
        /* TODO */
    }
})

// java 8 이후 함수형 인터페이스를 람다로 표현
button.setOnClickListener {view -> /* TODO */}
```
- 자바에서는 함수형 인터페이스 즉, SAM 인터페이스인 경우 자바 8버전 이후 람다를 이용하여 더 간결하게 표현할 수 있다. (코틀린도 너무나 당연하게 사용 가능하다.)

##### 자바 메소드에 람다를 인자로 전달

```java
// java 함수형 인터페이스를 인자로 전달
void postponeComputation(int delay, Runnable computation);
```

```kotlin
// 위의 자바 코드에 코틀린에서 람다를 전달하여 호출
postponeComputation(1000) { println(42) } // 함수형 인터페이스에 람다를 전달

// 객체 식을 전달
postponeComputation(1000, object: Runnable {
    override fun run() {
        println(42)
    }
})
```
- 컴파일러는 자동으로 람다를 Runnable 인스턴스(Runnable을 구현한 익명 클래스 인스턴스)로 변환하여 전달한다.
- Runnable을 구현하는 무명 객체를 명시적으로 만들어서 사용하는 것도 가능하다.

> 람다를 넘길 때와 무명 객체를 생성하여 넘길 때의 차이점?
>   - 무명 객체를 생성하여 넘기는 경우, 메소드를 호출할 때마다 새로운 인스턴스가 생성된다.
>   - 생성된 Runnable 인스턴스는 단 하나만 생성되며 메소드 호출 시 반복 사용된다.
>     - 단, 람다 내에서 람다 외부의 변수를 포획하는 경우에는 무명 객체처럼 새로운 인스턴스가 생성된다.

> Java 8 언어 기능과 Jack을 활성화 방법
> - app 단 build.gradle 내에 **compileOptions** 를 통해 지정해준다.
> - ![스크린샷 2020-03-05 오후 9.31.17](/img/스크린샷 2020-03-05 오후 9.31.17.png)
>
> 코틀린 컴파일 시 자바 8 바이트 코드생성 방법
> - jvm-target 1.8 이라고 kotlinc 호출할 때 커맨드라인에서 옵션 설정을 지정
> - 메이븐이나 그래들 프로젝트 설정에 명시

##### SAM 생성자: 람다를 함수형 인터페이스로 명시적으로 변경

컴파일러가 자동으로 람다를 함수형 인터페이스 익명 클래스로 바꾸지 못하는 경우 `SAM 생성자`를 사용한다.

```kotlin
val listener = OnClickListener { view -> 
    val text = when (view.id) {
        R.id.button1 -> "First button"
        R.id.button2 -> "Second button"
        else -> "Unknown button"
    }
    toast(text)
}

button1.setOnClickListener(listener)
button2.setOnClickListener(listener)
```

> 람다와 리스너 등록/해제
> - 람다는 코드 블럭이기 때문에 `this` 가 없다. 즉, 객체처럼 익명 클래스의 인스턴스를 참조할 수 없다.
> - 람다 내에서 `this`는 그 람다를 둘러싼 클래스의 인스턴스를 가르킨다. 주의하자.
> - 리스너를 가르키고 싶다면 람다가 아닌 **무명 객체**를 사용해야 한다.
> - 무명 객체 내에서 `this`는 객체 인스턴스 자신을 가르킨다.

#### 수신 객체 지정 람다: with와 apply

자바의 람다에는 없는, 코틀린 람다만의 독특한 기능인 `수신 객체 지정 람다`는 수신 객체를 명시하지 않고 람다의 본문 안에서 다른 객체의 메소드를 호출할 수 있게 하는 것이다.

##### with 함수

`with` 함수는 파라미터가 2개인 메소드로 첫 번째 인자는 객체를 두 번째 인자는 람다를 받는다.
  - 첫 번째 인자로 받은 객체를 두 번째 인자로 받은 람다의 수신 객체로 만든다.

```kotlin
fun alphabet(): String {
    val sb = StringBuilder()
    return with(sb) {
        for (letter in 'A'..'Z') {
            this.append(letter) // this를 통해 수신 객체에 접근
        }
        append("\nNow I know alphabet!") // this 없이 수신 객체의 메소드 호출
        this.toString() // 람다에서 값 반환
    }
}
```

- 수신 객체 지정 람다는 확장 함수와 비슷한 동작을 정의하는 한 방법이다.
- `<T, R> with(receiver: T, block: T.() ‐> R)`: block 함수의 수신 객체는 T

##### apply 함수

`apply` 함수는 `with` 함수와 동일한 동작이지만 항상 자신에게 전달된 객체(수신 객체)를 반환한다.

```kotlin
fun alphabet() = StringBuilder().apply {
    for (letter in 'A'..'Z') {
        append(letter)
    }
    append("\nNow I know the alphabet!")
}.toString()
```

- ` fun <T> T.apply(block: T.() ‐> Unit): T`: apply 함수는 확장 함수로 정의되어 있다.
- 객체의 인스턴스를 만들면서 즉시 프로퍼티 중 일부를 초기화 해야 하는 경우 유용하다.

**apply를 이용해 TextView 만들면서 초기화 하기**

```kotlin
fun createViewWithCustomAttributes(context: Context) =
    TextView(context).apply {
        text = "Sample Text" // this 생략하여 TextView의 프로퍼티 사용
        textSize = 20.0 
        setPadding(10, 0, 0, 0) // this 생략하여 TextView의 멤버 함수 사용
    }
```

##### buildString 함수

`buildString` 함수는 StringBuilder 객체를 만들어 toString()을 호출해주는 작업을 해준다.

```kotlin
fun alphabet() = buidlString {
    for (letter in 'A'..'Z') {
        append(letter)
    }
    append("\nNow I know the alphabet!")
}
```


###### 출처

- Kotlin IN Action / 출판사: 에이콘
  