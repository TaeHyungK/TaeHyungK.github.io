---
layout: post
title:  "[Kotlin] 함수 정의와 호출 1탄"
categories: Android
tags: Kotlin
author: TaeHyungK
---

* content
{:toc}

> Kotlin IN ACTION(출판사: 에이콘) 책을 통해 Kotlin을 배워보자

### 함수 정의와 호출

#### 컬렉션 만들기
```kotlin
val set = hastSetOf(1, 7, 53) // class java.util.HashSet
val list = arrayListOf(1, 7, 53) // class java.util.ArrayList
val map = hashMapOf(1 to "one", 7 to "seven", 53 to "fifty-three") // class java.util.HashMap
```
  - `map`에 사용된 `to`는 키워드가 아닌 일반 함수이다. (~~추후에 추가로 다룰 예정이다.~~)
  - 코틀린에서 컬렉션 클래스들은 **표준 자바 컬렉션을 활용**하고 있다.
    - 장점: 자바 코드와 상호작용하기가 쉽다. 즉, 자바와 코틀린을 한 프로젝트에서 섞어서 사용할 떄에 별도의 컨버팅 작업이 필요 없다!
  - 그러면서 자바보다 더 많은 기능을 사용할 수 있다.
    - `list.last()`: 리스트의 마지막 원소를 가져온다.
    - `set.max()`: 수로 이뤄진 컬렉션에서 최댓값을 가져온다.
    
#### 함수를 호출하기 쉽게 만들기
```kotlin
fun <T> joinToString(collection: Collection<T>, separator: String, prefix: String, postfix: String): String {
    val result = StringBuilder(prefix)
    for ((index, element) in collection.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(element)
    }
    result.append(postfix)
    return result.toString();
}

// >>> val list = listOf(1, 2, 3)
// >>> println(joinToString(list, "; ", "(", ")"));
// 결과: (1; 2; 3)
```









#### 이름 붙인 인자
```kotlin
jointoString(collection, separator = " ", prefix = " ", postfix = ".")
```
  - 가독성 향상을 위해 파라미터에 이름을 명시할 수 있다.
  - 단, 인자 중 어느 하나라도 이름을 명시하고 나면 혼동을 막기 위해 그 뒤에 오는 모든 인자는 이름을 **반드시 명시**해야 한다.
  
#### 디폴트 파라미터 값
```kotlin
fun <T> joinToString(collection: Collection<T>, separator: String = ", ", prefix: String = "", postfix: String = ""): String

// >>> jointoString(list, ", ", "", "")
// 결과: 1, 2, 3
// >>> joinToString(list) // separator, prefix, postfix 생략
// 결과: 1, 2, 3
// >>> joinToString(list, "; ") // separator "; "로 지정 + prefix, postfix 생략
// 결과: 1; 2; 3
// >>> joinToString(list, postfix = ";", prefix = "# ") // separator 생략 + postfix, prefix 위치 변경하여 호출
// 결과: # 1, 2, 3; 
```
  - 이름 붙인 인자를 사용하는 경우 인자 목록의 중간에 있는 인자를 생략하고, 지정하고 싶은 인자를 이름을 붙여서 **순서와 관계없이 지정**할 수 있다.
  
    > 개인적으로 매우 특이하다고 생각했다. 함수를 호출하면서 파라미터의 순서에 상관없이 값을 넣을 수 있다는 것이 정말정말 신기했다.

 > @JvmOverloads 어노테이션
 >  - 해당 어노테이션을 추가하면 코틀린 컴파일러가 자동으로 맨 마지막 파라미터로부터 파라미터를 하나씩 생략한 오버로딩한 자바 메소드를 추가해준다.
 >
 >   예를 들어 위의 `joinToString()` 메소드에 `@JvmOverloads` 어노테이션을 붙이면 아래와 같은 오버로딩한 함수가 만들어진다.
 >    - String joinToString(Collection<T> collection, String separator, String prefix, String postfix);
 >    - String joinToString(Collection<T> collection, String separator, String prefix);
 >    - String joinToString(Collection<T> collection, String separator);
 >    - String joinToString(Collection<T> collection);

#### 최상위 함수

```kotlin
package strings

fun joinToString(param: Int): String {
    // TODO 동작
}
```
```kotlin
import strings;

fun main(args: Array<String>) {
    // 최상위 함수 사용
    joinToString(10)
}
```
  - 최상위 함수는 jvm에 의해 자바의 static 함수로 변환된다.
    - java에서 호출: `JoinKt.joinToString(10)` (사전에 import는 당연히 필요하다: `import strings.JoinKt;`) 
    - 클래스명을 바꾸고 싶다면(위에서 JoinKt 부분) **@JvmName** 어노테이션으로 클래스 이름을 지정할 수 있다.


#### 최상위 프로퍼티
```kotlin
val UNIX_LINE_SEPARATOR = "\n" // getter() 를 통해 접근해야 접근이 가능
const val UNIX_LINE_SEPARATOR = "\n" // 바로 접근 가능
```
  - `const` 키워드를 통해 `public static final` 필드로 컴파일하게 할 수 있다.
  - 단, 원시 타입과 String 타입의 프로퍼티만 지정이 가능하다.

#### 확장 함수
어떤 클래스의 멤버 메소드인 것처럼 호출할 수 있지만 그 클래스의 밖에 선언된 함수
```kotlin
package strings
//    ↓ 수신 객체 타입            ↓ 수신 객체 ↓
fun String.lastChar(): Char = this.get(this.length - 1) // this 생략 가능
// >>> println("Kotlin".lastChar())
// 결과: n
```
  - 함수 이름 앞에 그 함수가 확장할 클래스의 이름을 붙여서 정의
    - 수신 객체 타입(receiver type): 확장을 정의할 클래스의 타입
    - 수신 객체(receiver object): 확장 함수가 호출되는 대상이 되는 값. 즉, 그 클래스에 속한 인스턴스
  - 수신 객체의 메소드나 프로퍼티 사용 가능
    - 단, private 멤버나 protected 멤버는 사용이 불가
  - 특정 클래스의 확장 함수와 해당 클래스의 멤버 함수의 이름과 시그니처가 같다면 멤버 함수가 호출된다. (**멤버 함수가 우선순위가 더 높다.**)

코틀린으로 선언한 위 함수를 자바에서 호출할 때 (StringUtil.kt 파일에 정의한 경우)
```java
char c = StringUtilKt.lastChar("Java");
```

#### 확장 함수와 임포트
```kotlin
import strings.lastChar
val c = "Kotlin".lastChar()
```
  - 확장 함수를 사용하려면 import를 해주어야 한다.
```kotlin
import strings.lastChar as last
val c = "Kotlin".last()
```
  - `as` 키워드를 통해 확장 함수를 다른 이름으로 부를 수 있다.
    - 여러 패키지에 있는 확장 함수를 사용할 때에 `as` 키워드를 통해 충돌을 피할 수 있다.

#### 확장 함수로 유틸리티 함수 정의
```kotlin
fun <T> Collection<T>.joinToString(separator: String = ", ", prefix: String = "", postfix: String = ""): String {
    val result = StringBuilder(prefix)
    for ((index, element) in this.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}

// >>> val list = arrayListOf(1, 2, 3)
// >>> println(list.joinToString(separator = "; ", prefix = "(", postfix = ")"))
// 결과: (1; 2; 3)
```
- 확잠 함수는 단지 **정적 메소드 호출에 대한 문법적인 편의**일 뿐이다.
- 즉, 확장 함수는 정적 메소드와 같은 특징을 가진다.
    - 이 말은, 확장 함수를 하위 클래스에서 **오버라이드** 할 수 없다는 말이다.

    ```kotlin
    fun View.showOff() = println("I'm a view!")
    fun Button.showOff() = println("I'm a button!")

    // >>> val view: View = Button()
    // >>> val.showOff()
    // 결과: I'm a view!
    ```
  - view가 가리키는 객체의 instance는 Button 타입이지만, view의 타입은 View이기 때문에 무조건 View의 확장 함수가 호출된다.
  - **확장 함수는 실제로 정적 메소드를 호출한다 ! + 호출될 확장 함수를 정적으로 결정한다 !**
      
#### 확장 프로퍼티
기존 클래스 객체에 대한 프로퍼티 형식의 구문으로 사용할 수 있는 API를 추가할 수 있다.
```kotlin
// 확장 프로퍼티 선언
val String.firstChar: Char
    get() = get(0)

// 변경 가능한 확장 프로퍼티 선언 
var StringBuilder.lastChar: Char
    get() = get(length - 1)
    set(value: Char) {
        this.setCharAt(length - 1, value)
    }

// >>> println("Kotlin".firstChar)
// 결과: K

// >>> val sb = StringBuilder("Kotlin?")
// >>> sb.lastChar = '!'
// >>> println(sb)
// 결과: Kotlin! 
```
  - 상태를 저장할 수는 없지만 프로퍼티 문법을 통해 코드를 더 짧게 작성할 수 있다.
  - BackingField가 없다.
    - getter 는 꼭 정의해야 한다. (기본 getter 를 제공할 수 없다.)
    - 초기화 코드도 쓸 수 없다. (계산한 값을 담을 장소가 없기 때문이다.)


###### 출처

- Kotlin IN Action / 출판사: 에이콘
  