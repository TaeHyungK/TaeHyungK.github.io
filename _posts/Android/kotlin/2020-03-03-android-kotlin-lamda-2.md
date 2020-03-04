---
layout: post
title:  "[Android] 람다로 프로그래밍 2탄"
categories: Android
tags: Kotlin
author: TaeHyungK
---

* content
{:toc}

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



###### 출처

- Kotlin IN Action / 출판사: 에이콘
  