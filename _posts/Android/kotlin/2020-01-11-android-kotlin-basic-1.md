---
layout: post
title:  "[Kotlin] 코틀린 기초 - 함수와 변수"
categories: Android
tags: Kotlin
author: TaeHyungK
---

* content
{:toc}

> Kotlin IN ACTION(출판사: 에이콘) 책을 통해 Kotlin을 배워보자

### 코틀린 기초

> 새로운 언어의 시작 Hello world를 찍어보자

```kotlin
fun main(args: Array<String>) {
    println("Hello world!")
}
```
- fun 키워드 : 함수 선언 키워드
- args: Array<String> : 파라미터 이름 뒤에 파라미터의 타입을 작성
- println : System.out.println 대신 사용하는 출력문
- 세미콜론을 붙이지 않아도 정상 작동함






#### 함수
- REPL(입력을 받아 값을 계산한 다음 결과 값을 출력하는 루프라는 뜻의 read-eval-print loop)을 통해 실행해보자(IntelliJ > Tools > Kotlin > Kotlin REPL)

```bash
Welcome to Kotlin version 1.3.41 (JRE 11.0.4+10-b304.69)
Type :help for help, :quit for quit

fun max(a: Int, b: Int): Int {
    return if (a > b) a else b
}
println(max(1, 2))
2 // 결과
```

> **문(statement)과 식(expression)의 구분**
>
> 식은 <u>값을 만들어</u> 내며 다른 식의 하위 요소로 계산에 참여할 수 있나 문은 자신을 둘러싸고 있는 가장 안쪽 블록의 최상위 요소로 존재하며 <u>아무런 값을 만들어내지 않는다</u>.
>
> 위의 식에서 if식은 3항 연산자로 작성한 (a > b) ? a : b 식과 비슷하다.

- 위 예제 코드를 더 줄여 더 간결한 함수로 표현할 수 있다.
    ```kotlin
    fun max(a: Int, b: Int): Int = if (a > b) a else b
    ```
    - 본문이 중괄호로 둘러싸인 함수를 **블록이 본문인 함수**라 부르고, 등호와 식으로 이뤄진 함수를 **식이 본문인 함수**라고 부른다.

- 위 예제 코드를 더더 줄여 반환 타입을 생략하면 더 간략한 함수로 표현할 수 있다.
    ```kotlin
    fun max(a: Int, b: Int) = if (a > b) a else b
    ```
    - 반환 타입을 생략할 수 있는 이유?
      - 식이 본문인 함수의 경우 반환 타입을 적지 않아도 컴파일러가 함수 본문 식을 분석해 식의 결과 타입을 함수 반환 타입으로 정해준다. 즉, 컴파일러가 `타입 추론`을 해준다.
      - 단, **식이 본문인 함수의 반환 타입만** 생략이 가능하다

> **Think**
>
> 실무에서 작성 코드가 긴 함수가 많고 조건에 따라 return이 여러개 있는 경우가 많다. 
> 또한, 여러 개발자가 손을 대기 때문에 가독성이 중요한 경우가 많아 긴 함수인 경우 반환 타입을 명시하는 것이 좋을 것이라고 생각이 든다.
> 그렇기 때문에 개인적으로는 정말 간단한 함수가 아니라면 **식이 본문인 함수**를 사용하는 것 보다는 **블록이 본문인 함수**를 사용을 지양하며 return 값을 명시적으로 하는 것이 좋을 것 같다.

#### 변수
- 코틀린은 변수를 선언할 때에 변수명 뒤에 명시해준다.
  > 뒤에 명시하는 이유?
  > <br> 타입을 생략할 경우 식과 변수 선언을 구별할 수 없기 때문!

```kotlin
// 타입 표기 생략
val question = "삶, 우주, 그리고 모든 것에 대한 궁극적인 질문"
val answer = 42
// 타입 표기 명시
val answer: Int = 42
```
- 타입을 지정하지 않을 경우 역시나 `타입 추론`을 통해 컴파일러가 초기화에 사용한 타입을 해당 변수 타입으로 지정한다. (부동소수점을 사용한다면 변수 타입은 Double이 된다.)
- **단, 초기화를 하지 않고 변수를 선언하려면 변수 타입을 반드시 명시해야 한다.**
> 아직 정응이 되지 않아 타입을 생략하는 것이 편하지는 않지만 추후 편해진다면 컴파일러가 해주는 `타입 추론`을 ~~사랑~~하게 될 것 같다.
  
- 변경 가능한 변수와 변경 불가능한 변수
  - val: **변경이 불가능한 값**을 저장하는 변수로 자바의 final 변수에 해당한다.
    - val 참조 자체는 불변일지라도 그 참조가 가리키는 객체의 내부 값은 변경될 수 있다.
      <br> -*하기 코드는 올바른 코틀린 코드이다.*-
        ```kotlin
        val languages = arrayListOf("Java") // 불변 참조 선언
        languages.add("Kotlin") // 참조가 가리키는 객체 내부를 변경
        ```
  - var: **변경이 가능한 값**을 저장하는 변수로 자바의 일반 변수에 해당한다.
    - var 키워드를 사용하면 변수의 값은 변경할 수 있지만 변수의 타입은 고정돼 바뀌지 않는다.
    ```kotlin
    var answer = 42
    answer = "no answer" // "Error: type mismatch" 컴파일 오류 발생
    ```

#### 문자열 템플릿 : 더 쉽게 문자열 형식 지정

- 문자열 템플릿 예제
    ```kotlin
      fun main(args: Array<String>) {
      val name = if (args.size > 0) args[0] else "Kotlin"
      println("Hello $name")
    }
    ```
  - 다른 스크립트 언어와 비슷하게 코틀린에서도 문자열 안에서 변수를 사용할 수 있다.
  - 문자열의 필요한 곳에 변수를 넣되 변수 앞에 "$"를 추가하면 변수 값을 불러올 수 있다.
  - 같은 연산: "Hello, " + name + "!"
  - 복잡한 식인 경우 중괄호({})로 둘러싸서 문자열 템플릿 안에 넣는 것도 가능하다.
    ```kotlin
    fun main(args: Array<String>) {
        if (args.size > 0) {
            println("Hello, ${args[0]}!")
        }
    }
    ```

###### 출처

- Kotlin IN Action / 출판사: 에이콘