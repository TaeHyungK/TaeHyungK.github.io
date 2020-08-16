---
layout: post
title:  "[Kotlin] 람다로 프로그래밍 1탄"
categories: [Android, Kotlin]
tags: [Kotlin]
---

> Kotlin IN ACTION(출판사: 에이콘) 책을 통해 Kotlin을 배워보자

### 람다 식 | 람다

기본적으로 다른 함수에 넘길 수 있는 작은 코드 조각을 뜻한다.

#### 람다 식과 멤버 참조

람다는 자바 8에 도입되어 자바에서도 비로소 람다를 사용할 수 있게 되었다.

##### 람다 소개: 코드 블록을 함수 인자로 넘기기

**[자바에서 익명 클래스를 통해 버튼 클릭 리스너 구현]**
```java
/* 자바 */
button.setOnClickListener(new OnClickListener() {
    @Override
    public void onClick(View view) {
        /* 클릭시 동작 */
    }
});
```

**[람다로 리스너 구현]**
```kotlin
button.setOnClickListener { /* 클릭시 동작 */}
```

- 두 구현은 동일한 동작이지만 람다를 이용한 경우 코드가 훨씬 간결하고 읽기 쉬워진다.







##### 람다와 컬렉션

**[컬렉션에서 가장 큰 값 찾기: 직접 구현]**
```kotlin
fun findTheOldest(people: List<Person>) {
    var maxAge = 0
    var theOldest: Person? = null
   
    for (person in people) {
        if (person.age > maxAge) {
            maxAge = person.age
            theOldest = person
        }
    }
    println(theOldest)
}

val people = listOf(Person("Alice", 29), Person("Bob", 31))
findTheOldest(people)
// 결과: Person(name=Bob, age=31)
```

**[컬렉션에서 가장 큰 값 찾기: 람다 이용]**
```kotlin
val people = listOf(Person("Alice", 29), Person("Bob", 31))
println(people.maxBy { it.age }) // 나이 프로퍼티를 비교해서 가장 큰 원소 찾기
// 결과: Person(name=Bob, age=31)

println(people.maxBy(Person::age)) // 나이 프로퍼티를 비교해서 가장 큰 원소 찾기
// 결과: Person(name=Bob, age=31
```

- 자바 컬렉션에 대해 (자바 8 이전, 람다 지원 전) 수행하던 대부분의 불편했던 작업들은 람다나 멤버 참조를 인자로 취하는 라이브러리 함수를 통해 개선이 가능하다. 이렇게 개선된 후에는 코드가 한결 짧아지고 더 이해하기 쉬워졌다.

##### 람다 식의 문법

```kotlin
{ x: Int, y: Int -> x + y }
//    파라미터     ->   본문
``` 

- 람다는 **값처럼** 여기저기 전달할 수 있는 동작의 모음이다.
  - 추가로 람다를 따로 선언해서 변수에 저장도 가능하다. 그렇다곤 해도 보통은 함수에 인자로 넘기면서 바로 람다를 정의하는 경우가 대부분이다.
  - ```kotlin
    val sum = { x: Int, y: Int -> x + y} // 람다식 변수에 저장
    println(sum(1, 2)) // 변수에 저장된 람다식 호출
    // 결과: 3
    ```
  - 심지어는 람다식을 직접 호출도 가능하다.
  - ```kotlin
    { println(42) }()
    //  결과: 42
    ```

    > 굳이 이렇게 쓰는 것보다는 가독성 때문에라도 람다 본문을 직접 실행하는 편이 낫다.
    >  ```kotlin
    >  // 인자로 받은 람다를 실행해주는 라이브러리 함수인 run을 사용하여 람다 본문 실행
    >  run { println(42) }
    >  // 결과: 42
    >  ``` 

- 코틀린 람다 식은 **항상 중괄호**로 둘러싸여있다.
- 인자 목록 주변에 괄호가 없다.
- 화살표(->)를 통해 인자 목록과 람다 본문을 구분한다.

**람다식 사용 예제**
```kotlin
val people = listOf(Person("Alice", 29), Person("Bob", 31))

// 인자가 한 개고 타입 추론 가능하면 디폴트 이름인 it 사용 가능
println(people.maxBy { it. age })


people.maxBy({ p: Person -> p.age})
// 람다가 마지막 인자면, 괄호 밖에 위치 가능
people.maxBy() { p: Person -> p.age}

// 람다가 마지막 인자고, 괄호 뒤에 람다를 썼을 경우 괄호 생략 가능
people.maxBy { p: Person -> p.age}
```

> 간단한 경우라면 괄호 없이 람다식만으로 명시해도 나쁘지 않을 것으로 생각되지만 웬만하면 괄호를 사용하여 해당 람다식이 메소드에 포함된 람다식이라는 것을 확실히 하는 것이 가독성에 더 좋을 것 같다.

**이름 붙인 인자를 사용해 람다 넘기기**
```kotlin
val people = listOf(Person("이몽룡", 29), Person("성춘향", 31))
val names = people.joinToString(seprator = " ", transform = { p: Person -> p.name })
println(names)
// 결과: 이몽룡 성춘향

// 괄호 밖으로 처리
val names = people.joinToString(" ") { p: Person -> p.name}
```

> 개인적인 생각이지만 이 예제를 보니 개인적인 프로젝트에서 특정 메소드에 대한 완벽한 이해가 있고 람다식을 사용하는 곳도 완벽한 이해가 있는 경우에는 람다식을 괄호 밖으로 빼서 표현하는 것이 코드도 간결하고 보기도 더 좋지만 **협업을 하는 상황**이라면 누구라도 코드를 이해하기 쉽도록 괄호를 사용하여 일반적인 메소드 호출을 이용하여 람다식을 표현하는 것이 더 좋을 것으로 생각된다.  

**람다 파라미터 타입 제거하기**
```kotlin
people.maxBy { p: Person -> p.age } // 파라미터 타입 명시
people.maxBy { p -> p.age } // 파라미터 타입 생략 (컴파일러가 추론)

// 변수에 람다식 담는 경우 컴파일러가 타입 추론 불가
val getAge = { p: Person -> p.age }
people.maxBy(getAge)
```

- 컴파일러는 람다 파라미터의 타입도 **추론** 할 수 있다.
- people은 Person을 담은 컬렉션이므로 컴파일러는 Person 객체가 파라미터로 들어올 것을 추론할 수 있다.
  - ~~컴파일러가 람다 파라미터의 타입을 추론하지 못하는 경우도 있으나 이번 장에서는 다루지 않는다.~~
- 단, 람다식을 변수에 담는 경우 파라미터의 타입을 추론할 문맥이 존재하지 않으므로 파라미터를 명시해야만 한다.

**디폴트 파라미터 이름 it 사용하기**
```kotlin
people.maxBy { it.age }
```

- `it`는 자동 생성된 파라미터 이름이다.
- 람다의 파라미터가 **하나**뿐이고 그 타입을 컴파일러가 추론할 수 있는 경우 `it`를 사용할 수 있다.

> 람다 내에 람다가 중첩되는 경우나 문맥에서 람다 파라미터의 의미나 파라미터의 타입을 쉽게 알 수 없는 경우에는 파라미터를 명시적으로 선언하는 것이 가독성에 더 좋다.

**본문이 여러줄로 이뤄진 람다식**
```kotlin
val sum = { x: Int, y: Int -> 
    println("Computing the sum of $x and $y...")
    x + y
}
```

- 본문이 여러줄로 이루어진 경우 **맨 마지막에 있는 식**이 람다식의 결과값이 된다.

##### 현재 영역에 있는 변수에 접근

```kotlin
fun printMessageWithPrefix(messages: Collection<String>, prefix: String) {
    messages.forEach {
        println("$prefix $it") // 람다 내부에서 함수의 "prefix" 변수 사용
    }
}

// >>> val errors = listOf("403 Forbidden", "404 Not Found")
// >>> printMessagesWithPrefix(errors, "Error:")
// 결과: Error: 403 Forbidden
//      Error: 404 Not Found
```

- 람다를 함수 안에서 정의하면 함수의 파라미터에 접근이 가능하다.
- 뿐만 아니라 정의의 앞에 선언된 로컬 변수까지 람다에서 모두 사용할 수 있다.

```kotlin
fun printProblemCounts(response: Collection<String>) {
    var clientErrors = 0 // 람다 외부에 로컬 변수 선언
    var serverErrors = 0 // 람다 외부에 로컬 변수 선언
    
    response.forEach {
        if (it.startsWith("4")) {
            clientErrors++ // 람다 내부에서 외부의 로컬 변수 값 변경
        } else if (it.startsWith("5")) {
            serverErrors++ // 람다 내부에서 외부의 로컬 변수 값 변경
        }
    }
    println("$ClientErrors client errors, $serverErrors server errors")
}

// >>> val responses = listOf("200 OK", "418 I'm a teapot", "500 Internal Server Error"
// >>> printProblemCounts(responses)
// 결과: 1 client errors, 1 server errors
```

- 람다 내부에서는 `final` 변수가 아닌 변수에 접근이 가능하다.
- 람다 내부에서 람다 외부의 변수 변경도 가능하다.
  - 람다 내부에서 사용하는 람다 외부 변수를 `람다가 포획한 변수`라고 부른다. (위 예제들의 prefix, clientErrors, serverErrors)

> 람다를 실행 시점에 표현하는 데이터 구조는 람다에서 시작하는 모든 참조가 포함된 닫힌(closed) 객체 그래프를 람다 코드와 함께 저장해야 한다. 그런 데이터 구조를 **클로저(closure)** 라고 부른다. 함수를 쓸모 있는 1급 시민으로 만들려면 포획한 변수를 제대로 처리해야 하고, 포획한 변수를 제대로 처리하려면 클로저가 꼭 필요하다. 그래서 람다를 클로저라고 부르기도 한다.
>
> TODO 무슨 말인지 이해가 가지 않는다.. 코틀린 스터디 팀원들에게 물어보고 내용을 추가하도록 하자..!

**로컬 변수의 생명주기와 함수의 생명주기가 다른 경우**

(예를 들면, 어떤 함수가 자신의 로컬 변수를 포획한 람다를 반환하거나 다른 변수에 저장하는 경우가 있다.)

- 포획한 변수가 있는 람다를 저장해서 함수가 끝난 뒤에 실행해도 람다의 본문 코드는 여전히 포획한 변수를 읽거나 쓸 수 있다.
  - `final` 변수인 경우: 람다 코드를 변수 값과 함께 저장하여 함수가 끝난 뒤에도 포획한 변수에 접근이 가능하다.
  - `final` 변수가 아닌 경우: 변수를 특별한 **래퍼**로 감싸서 나중에 변경하거나 읽을 수 있게 한 다음, 래퍼에 대한 참조를 람다 코드와 함께 저장한다.
  - 코틀린에서도 자바와 같이 약간의 꼼수(?)로 변경 가능한 변수를 포획하게 된다.
    - ```kotlin
      // 실제 코드
      var counter = 0
      val inc = { counter++ }
      
      // 내부 동작을 보여주는 코드
      class Ref<T>(var value: T)
      val counter = Ref(0)
      val inc = { counter.value++ }
      ```
    - `Ref` 라는 클래스로 래핑하여 해당 클래스를 `final` 하게 선언하고 그 내부에 멤버 변수에 counter 값을 저장한다.
    - 그 이후 람다식에서는 클래스의 변수값에 접근하여 변경 가능한 변수를 포획한다.

**람다를 이벤트 핸들러 등 비동기 실행 코드로 활용하는 경우**
```kotlin
fun tryToCountButtonClicks(button: Button): Int {
    var clicks = 0
    button.onClick { clicks++ }
    return clicks
}
```

- 람다를 비동기적으로 실행되는 코드로 활용하는 경우 함수 호출이 끝난 다음에 로컬 변수가 변경될 수도 있다는 점을 인지해 유의하여 사용해야 한다.
- 위 예시 코드에서 해당 함수는 항상 0을 반환한다.
  - onCiick 핸들러는 버튼이 클릭될 때마다 clicks 변수를 증가시키지만 그 때에는 함수 호출이 종료된 이후이기 때문이다.
  - 즉, 해당 clicks 변수를 확인할 수 있도록 **클래스의 프로퍼티**나 전역 프로퍼티 등의 위치로 빼서 나중에 해당 변수를 확인할 수 있도록 해야 한다.

##### 멤버 참조

**이미 선언된 함수를 값으로 사용해야 할 때 멤버 참조 `::` 를 사용하면 된다.**

```kotlin
// 모두 같은 동작
people.maxBy(Person::age) // 멤버 참조
people.maxBy { p -> p.age }
people.maxBy { it.age }

fun Person.isAdult() = age >= 21
val predicate = Person::isAdult // 확잠 함수도 동일하게 멤버 참조를 사용할 수 있음

fun salute() = println("Salute!")
run(::salute) // 최상위 함수 참조
// 결과: Salute!

// sendEmail 함수에게 작업 위임
val action = { person: Person, message: String -> sendEmail(person, message) }
// 람다 대신 멤버 참조 사용
val nextAction = ::sendEmail

// 생성자 참조
data class Person(val name: String, val age: Int)
val createPerson = ::Person // 생성자 참조 저장
val p = createPerson("Alice", 29) // 생성자 참조를 이용해 인스턴스 생성
```

- **멤버 참조**는 프로퍼티나 메소드를 단 하나만 호출하는 함수 값을 만들어준다.
- `::` 는 클래스 이름과 참조하려는 멤버(프로퍼티나 메소드) 이름 사이에 위치한다.
- **멤버 참조** 뒤에는 괄호를 넣으면 안된다. (메소드여도!!)
- **멤버 참조**는 그 멤버를 호출하는 람다와 같은 타입이다.
- 최상위 함수, 최상위 프로퍼티 참조도 가능하다.
  - 클래스 이름을 생략하고 `::` 로 참조를 바로 시작하면 된다.
- **생성자 참조**를 사용하면 클래스 생성 작업을 연기하거나 저장해둘 수 있다.
  - `::` 뒤에 클래스 이름을 넣으면 생성자 참조를 만들 수 있다.

**바운드 멤버 참조 (1.1부터 사용 가능)**
```kotlin
// 1.0 멤버 참조
val p = Person("Dmistry", 34)
val personAgeFunction = Person::age
println(personAgeFunction(p))
// 결과: 34

// 1.1 바운드 멤버 참조
val p = Person("Dmistry", 34)
val ageFunction = p::age // p에 엮인 멤버 참조
println(ageFunction())
// 결과: 34
```

> TODO 바운드 멤버 참조는 한 인스턴스에 대해서만 동작이 될 것으로 생각이 드는데 이걸 사용할만한 곳이 있을까? 하는 의문이 든다.

###### 출처

- Kotlin IN Action / 출판사: 에이콘
  