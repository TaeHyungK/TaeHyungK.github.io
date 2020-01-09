---
layout: post
title:  "[Android] 코틀린 맛보기"
categories: Android
tags: Kotlin
author: TaeHyungK
---
* content
{:toc}

> Kotlin IN ACTION(출판사: 에이콘) 책을 통해 Kotlin을 배워보자

### 코틀린 맛보기

- Person이라는 클래스를 정의하고, 그 클래스를 사용해 사람을 모아둔 컬렉션을 만들어 가장 나이가 많은 사람을 찾아 출력하는 코드

```kotlin
data class Person (val name: String, val age: Int? = null)

fun main (args: Array<String>) {
  val persons = listOf(Person("영희"), Person("철수", age = 29))
	val oldest = persons.maxBy { it.age ?: 0}
	println("나이가 가장 많은 사람: $oldest")
}

// >>> 결과: 나이가 가장 많은 사람: Person(name=철수, age=29)
```

- Data class Person: name과 age라는 프로퍼티(property)가 들어간 데이터 클래스를 정의
- val age: Int? = null: 디폴트 값은 null
  - 영희는 age를 지정하지 않았기 때문에 null이 쓰이게 됨
- person.maxBy 구문
  - maxBy에 전달한 람다식은 파라미터를 하나 받음
  - it라는 이름을 사용하면 람다식의 유일한 인자를 사용할 수 있음
  - "?:" ( == 엘비스 연산자)는 age가 null인 경우 0을 반환하고, 그렇지 않은 경우 age값을 반환함

> **Think**
>
> 지금은 위 코드와 코드에 대한 설명에 이해가지 않는 부분이 꽤 있다.(~~전에 코틀린을 학습해봤던 경험이 있어 나름 코드는 꽤 읽힌다~~)  자바로 위와 같은 코드를 짜려면 파라미터에 default 값을 지정할 수 없으니 Person에 대한 생성자가 1개 더 생길 것이고 Person을 생성하여 ArrayList에 담아주는 부분과 maxBy함수 내부가 어떻게 되어있는지 모르겠으나 for문을 돌며 Person 객체의 age에 직접 접근하여 if문을 통해 비교하는 코드가 짜였을 것 같다. 불필요하게 긴 코드량을 직관적으로 잘 줄일 수 있는 Kotlin 언어에 관심이 생겼다.

### 코틀린 특징

- 타입 추론

  ```kotlin
  var x = 1
  ```
  위 코드와 같이 변수 타입을 지정하지 않아도 `타입 추론`을 통해 **컴파일러**가 문맥을 고려해 변수 타입을 결정한다.
  
- 클래스(class), 인터페이스(interface), 제네릭(generics)은 기존 자바와 비슷하게 작동한다.

- 널이 될 수 있는 타입

  널이 될 수 있는 타입(nullable type)을 지원함에 따라 **컴파일 시점**에 NPE(NullPointerException)가 발생할 수 있는지 여부를 검사할 수 있어 프로그램의 신뢰성을 더 높일 수 있다.
  
- 함수형 프로그래밍에 대한 지원을 한다.
  - 함수 타입을 지원함
    - 어떤 함수가 다른 함수를 파라미터로 받거나 함수가 새로운 함수를 반환할 수 있다.
  - 람다식을 지원함
    - 번거로운 준비 코드 없이 코드 블록을 쉽게 정의하고 여기저기 전달할 수 있다.
  - 데이터 클래스는 불변적인 Value Object를 간편하게 만들 수 있는 구문을 제공함
  - 코틀린 표준 라이브러리는 객체와 컬렉션을 함수형 스타일로 다룰 수 있는 API를 제공함 
  
  > 함수형 프로그래밍의 특징
  >
  > - **일급 시민인 함수**
  >   - 함수를 일반 값 처럼 변수에 저장할 수 있음
  >   - 함수를 인자로 다른 함수에 전달 할 수 있음
  >   - 함수에서 또 다른 함수를 생성하여 반환할 수 있음
  > - **불변성(immutable)**
  >   - 일단 만들어지고 나면 내부 상태가 절대로 바뀌지 않는 불변 객체를 사용해 프로그램을 작성함
  > - **side effect 없음**
  >   - 함수형 프로그래밍에서는 입력이 같으면 항상 같은 출력을 내놓음
  >   - 다른 객체의 상태를 변경하지 않음
  >   - 함수 외부나 다른 바깥 환경과 상호작용하지 않는 순수 함수를 사용함

> **Think**
>
> 함수형의 장점을 사용할 수 있도록 Kotlin에서는 함수형 프로그래밍을 할 수 있도록 1급 객체(일급 시민) 함수를 지원하고 컬렉션을 이용할 때 다소 귀찮은 코드들이 필요했었는데 함수형 스타일로 다룰 수 있는 API를 지원한다는 것이 좋은 것 같다. 또한 함수형 프로그래밍의 가장 큰 장점이라 생각하는 **side effect 없음**이 가능하다는 것에 대해 정말 Kotlin으로 코드를 짜면 없을까?라는 생각이 들긴하지만 어느정도 방어가 된다는 것으로 보여 학습해 볼 가치가 있는 언어인 것 같다는 생각이 들었다.

- ClassCastException을 방지해주려는 노력과 코드 간결화

  타입을 검사했을 경우 그 타입을 캐스트 없이 사용이 가능하다
  ```kotlin
  if (value is String) { // 타입 검사
      println(value.toUpperCase()) // 캐스팅 없이 해당 타입의 메소드 사용
  }
  ```

#### 더 알아보면 좋을 것 같은 내용

- [코틀린 개발 GitHub](https://github.com/jetbrains/kotlin)

- 코틀린 팀이 만든 [Anko 라이브러리](https://github.com/kotlin/anko)

###### 출처

- Kotlin in Action / 출판사: 에이콘