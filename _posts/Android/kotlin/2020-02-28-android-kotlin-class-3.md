---
layout: post
title:  "[Kotlin] 클래스, 객체, 인터페이스 3탄"
categories: [Android, Kotlin IN Action]
tags: [Kotlin]
---

> Kotlin IN ACTION(출판사: 에이콘) 책을 통해 Kotlin을 배워보자

### 클래스, 객체, 인터페이스

#### 모든 클래스가 정의해야 하는 메소드: toString(), equals(), hashCode()

```kotlin
class Client(val name: String, val postalCode: Int)
```

- **toString()**: 문자열을 표현하기 위한 메소드로 자바와 마찬가지로 코틀린에서도 클래스에 기본적으로 포함되어 있다.
  - 오버라이드 하기
    - `override fun toString() = "Client(name=$name, postalCode=$postalCode)"` 
- **equals()**: 객체의 동등성을 비교하기 위한 메소드이다.
  - ```kotlin
    val client1 = Client("txxbro", 1234)
    val client2 = Client("txxbro", 1234)
    println(client1 == client2) // 동등성 검사
    // 결과: false
    ```
      - 코틀린에서 `==` 연산자는 참조 동일성을 검사하는 것이 아닌 **객체의 동등성**을 검사한다. 따라서 `==` 연산은 `equals()` 를 호출하는 식으로 컴파일 된다.
      - 자바에서의 `==` 연산자는 참조 동일성을 검사하며 코틀린에서는 `===` 연산자를 사용한다.
  
- **hashCode()**: 객체의 해시코드를 반환하는 메소드이다.
  - JVM 언어에서는 hashCode가 지켜야하는 "equals()가 true를 반환하는 두 객체는 반드시 같은 hashCode()를 반환해야한다."라는 제약이 있다.
  - 내부적으로 `<객체모음>.contains(<객체>)` 을 하는 경우 먼저 해시 코드를 비교하고 해시 코드가 같은 경우에만 실제 값을 비교하게 된다.
  - 즉, 원소 객체들이 해시 코드에 대한 규칙을 지키지 않는 경우 HashSet은 제대로 동작하지 않게 된다. 이 문제를 고치기 위해서는 <객체> 클래스가 `hashCode()` 를 구현해야 한다.








> 다행히도 위에 적은 메소드들은 자동으로 생성해준다. 

#### 데이터 클래스: copy()

```kotlin
data class Client(val name: String, val postalCode: Int)
``` 

- `class` 앞에 `data` 만 붙여주면 데이터 클래스가 된다.
- 데이터 클래스는 데이터를 저장하는 역할만을 수행하는 클래스이며 필요한 메소드를 컴파일러가 자동으로 만들어 준다.
  - `toString()`, `equals()`, `hashCode()` 는 당연히 만들어준다.
  - `copy()` 도 만들어 준다.
    - `copy()`: 객체를 복사하면서 일부 프로퍼티를 바꿀 수 있게 해준다.
    - ```kotlin
      val temp = Client("temp", 1234)
      println(temp.copy(postalCode = 4000))
      // 결과: Client(name=temp, postalCode=4000)
      ```  
      
#### 데이터 클래스: 클래스 위임

`by` 키워드를 사용하여 손쉽게 위임을 구현할 수 있다.

```kotlin
class CountingSet<T>(
    val innerSet: MutableCollection<T> = HashSet<T>()
  ) : MutableCollection<T> by innerSet { // MutablleCollection의 구현을 innerSet 에게 위임한다. 
    var objectsAdded = 0
    override fun add(element: T): Boolean { // 위임하지 않고 새로운 구현을 제공
        objectAdded++
        return innerSet.add(element)
    }
    
    override fun addAll(c: Collection<T>): Boolean { // 위임하지 않고 새로운 구현을 제공
        objectAdded += c.size
        return innerSet.addAll(c)
    }
}
```

#### object 키워드: 싱글턴을 쉽게 만들기

`object` 키워드는 클래스를 정의하면서 동시에 인스턴스를 생성하게 한다.

```kotlin
object Payroll {
    val allEmployees = arrayListOf<Person>()
    fun calculateSalary() {
        for (person in allEmployees) {
            /* TODO */
        }
    }
}

// 마침표(.) 를 통해 메소드나 프로퍼티에 접근이 가능하다.
// >>> Payroll.allEmployees.add(Person(...))
// >>> Payroll.calculateSalary()
```

- `객체 선언`은 클래스 선언과 그 클래스에 속한 단일 인스턴스의 선언을 합친 선언이다.
- `객체 선언`은 `object` 키워드로 시작한다.
- `객체 선언` 안에도 클래스처럼 프로퍼티, 메소드, 초기화 블록 등이 들어갈 수 있다. 단, 생성자는 객체 선언에 쓸 수 없다.
  - 일반 클래스 인스턴스와 달리 생성자 호출 없이 즉시 만들어지기 때문이다.
- `객체 선언` 도 클래스나 인터페이스를 상속할 수 있다.

```kotlin
data class Person(val name: String) {
    object NameComparator: Comparator<Person> {
        override fun compare(p1: Person, p2: Person): Int = 
            p1.name.compareTo(p2.name)
    }
}

// >>> val persons = listOf(Person("Bob"), Person("Alice"))
// >>> println(persons.sortedWith(Person.NameComparator))
// 결과: [Person(name=Alice), Person(name=Bob)]
```

#### 동반 객체 (companion object)

`companion` 키워드를 붙이면 동반 객체가 된다.

```kotlin
class User private constructor(val nickname: String){ // 주 생성자를 private으로 생성
    companion object { // 동반 객체 선언
        fun newSubscribingUser(email: String) = User(email.substringBefore('@'))
        fun newFacebookUser(accountId: Int) = User(getFacebookName(accountId))
    }
}

// >>> val subscribingUser = User.newSubscribingUser("bob@gmail.com")
// >>> val facebookUser = User.newFacebookUser(4)
// >>> println(subscribingUser.nickname)
// 결과: bob
```

- `동반 객체`로 선언한 메소드(팩토리 메소드)를 호출할 때에는 클래스 이름을 이용한다.
- `동반 객체`의 멤버를 사용하는 구문은 자바의 static 메소드나 static 필드와 유사하다.
- `동반 객체`는 자신을 둘러싼 클래스의 모든 private 멤버에 접근이 가능하다. 

#### 동반 객체: 인터페이스 구현

```kotlin
interface JSONFactory<T> {
    fun fromJSON(jsonText: String): T
}

class Person(val name: String) {
    companion object : JSONFactory<Person> {
        // 동반 객체가 인터페이스를 구현한다.
        override fun fromJSON(jsonText: String): Person = /* TODO */
    }
}
```

- 동반 객체도 인터페이스 구현이나 클래스 상속이 가능하다.

#### 동반 객체: 확장 함수 정의

```kotlin
// 비즈니스 로직 모듈
class Person(val firstName: String, val lastName: String) {
    companion object {
        // 비어있는 동반 객체 선언
    }
}

// 클라이언트/서버 통신 모듈
// 확장 함수 선언
fun Person.Companion.fromJSON(json: String): Person {
    /* TODO */
}
```

- 동반 객체에 대한 확장 함수도 가능하다.
  - 단, 확장 함수를 작성하려면 원래 클래스에 동반 객체를 **꼭** 선언해주어야 한다.
- 마치 동반 객체 안에서 `fromJSON()` 메소드를 정의한 것처럼 `fromJSON()` 메소드를 사용할 수 있다.

#### 객체 식

`object` 키워드를 이용해 익명 객체를 정의할 수 있다.

```kotlin
fun countClicks(window: Window) {
    var clickCount = 0 // 로컬 변수 선언

    window.addMouseListener(object: MoustAdapter() { // MouseAdapter를 확장하는 익명 객체 선언 
        override fun mouseClicked(e: MouseEvent) { // 오버라이드
            clickCount++ // 로컬 변수의 값을 변경
        }
    })
}
```
- 객체 식 안의 코드는 그 식이 포함된 함수의 변수에 접근할 수 있다.



###### 출처

- Kotlin IN Action / 출판사: 에이콘
  