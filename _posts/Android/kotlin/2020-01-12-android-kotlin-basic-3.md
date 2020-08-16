---
layout: post
title:  "[Kotlin] 코틀린 기초 - enum과 when"
categories: [Android, Kotlin IN Action]
tags: [Kotlin]
---

> Kotlin IN ACTION(출판사: 에이콘) 책을 통해 Kotlin을 배워보자

### enum과 when

#### enum
- enum 클래스 정의
    ```kotlin
    enum class Color {
        RED, ORANGE, YELLOW, GREEN, BLUE, INDIGO, VIOLET
    }
    ```
  - 코틀린에서는 enum class를 사용하지만 자바에서는 enum을 사용한다.
  - 코틀린에서 enum은 소프트 키워드(soft keyword)라 부른다.
  - class 앞에 붙여질 경우 특별한 의미를 지니지만 다른 곳에서는 이름에 사용할 수 있다.






- 프로퍼티와 메소드가 있는 enum 클래스 선언해보자
    ```kotlin
    enum class Color(val r: Int, val g: Int, val b: Int) {
        RED(255, 0, 0), ORANGE(255,165, 0), 
        YELLOW(255, 255, 0), GREEN(0, 255, 0), BLUE(0, 0, 255),
        INDIGO(75, 0, 130), VIOLET(238, 130, 238);
  
        fun rgb() = (r * 256 + g) * 256 + b
    }
    
    // >>> pirntln(Color.BLUE.rgb())
    // 결과: 255
    ```
    > 각 프로퍼티들을 생성한 후(위 코드의 마지막 아이템인 VIOLET 뒤에) 세미콜론은 필수로 명시해주어야 한다. (정확하게는 enum 상수 목록과 메소드 정의 사이에 세미콜론이 필수적으로 필요하다.)
    >  
    > 코틀린에서 유일하게 세미콜론이 필수인 부분이다!
    
    - enum에서도 일반적인 클래스와 마찬가지로 생성자와 프로퍼티를 선언한다.

#### when
- when은 자바의 **switch**를 대치한다.
- if와 마찬가지로 when도 값을 만들어내는 식이다.
  - 따라서, 식이 본문인 함수에 when을 바로 사용할 수 있다. 
- when으로 enum 클래스를 다뤄보자
    ```kotlin
    fun getMnemonic(color: Color) =
        when (color) {
            Color.RED -> "It is RED!!"
            Color.ORANGE -> "It is ORANGE!!"
            Color.YELLOW -> "It is YELLOW!!"
            Color.GREEN, Color.BLUE -> "It is GREEN OR BLUE!!"
            // 이하 생략..
        }
    
    // >>> println(getMnemonic(Color.ORANGE))
    // 결과: It is ORANGE   
    ```
    - 색이 특정 enum 상수와 같을 때 그 상수에 대응하는 문자열을 반환해준다.
    - 자바와 달리 `break`를 넣지 않아도 된다.
    - 한 분기 안에서 여러 값을 매치 패턴으로 사용하려면 콤마(,)로 구분한다.
- when과 임의의 객체를 함께 사용해보자
  - when은 분기 조건에 임의의 객체도 넣을 수 있다.
    ```kotlin
    fun mix(c1: Color, c2: Color) =
        when (setOf(c1, c2)) {
            setOf(RED, YELLOW) -> ORANGE
            setOf(YELLOW, BLUE) -> GREEN
            setOf(BLUE, VIOLET) -> INDIGO
            else -> throw Exception("Dirty color")
        }
    
    // >>> println(mix(BLUE, YELLOW))
    // 결과: GREEN
    ```
    - setOf(): 자바로 치면 set을 만들어 주는 메소드로 HashSet과 비슷하다고 생각하면 된다.
    - c1, c2가 들어오는 순서에 상관이 없다.
    - else: 조건들에 해당하는 것이 없을 경우 실행된다. switch-case문의 default와 같다. 

  - 인자가 없는 when을 사용하여 위 예시 코드를 작성해보자.
    ```kotlin
    fun mixOptimized(c1: Color, c2: Color) = 
        when {
            (c1 == RED && c2 == YELLOW) ||
            (c1 == YELLOW && c2 == RED) -> 
                ORANGE
            (c1 == YELLOW && c2 == BLUE) ||
            (c1 == BLUE && c2 == YELLOW) -> 
                GREEN
            (c1 == BLUE && c2 == VIOLET) ||
            (c1 == VIOLET && c2 == BLUE) ->
                INDIGO
            else -> throw Exception("Dirty color")
        }
    
    // >>> println(mixOptimized(BLUE, YELLOW))
    // 결과: GREEN
    ```
      > **Think**
      >
      > 인자가 없는 when을 만들면 불필요한 인스턴스를 생성하지 않아 불필요한 가비지 객체가 늘어나지 않는 장점이 있지만
      가독성은 떨어지는 코드가 될 수 있다.
      >
      > 성능 개선이 꼭 필요한 상황에만 사용하는 것이 가독성 측면에서 좋을 것 같다. 

  - **스마트 캐스트**
    - **object의 타입 확인과 변환을 한번에 해주는 기능으로 매우 유용한 기능이다.**
    - 트리 구조를 가진 산술식을 계산하는 함수를 만들어보자.
        ```kotlin
        interface Expr
        class Num(val value: Int): Expr
        class Sum(val left: Expr, val right: Expr): Expr
        
        fun eval(e: Expr): Int {
            if (e is Num) { // 스마트 캐스트가 발생한다.
                val n = e as Num // 이미 스마트 캐스트가 발생하였으므로 타입 변환을 해줄 필요가 없다. 즉, 불필요한 타입 변환이므로 생략 가능한 라인이다.
                return n.value          
            }
            if (e is Sum) { // 스마트 캐스트가 발생한다.
                return eval(e.right) + eval(e.left) // 이미 스마트 캐스트가 발생하였으므로 e는 Sum type이다. 따라서 Sum의 프로퍼티인 left, right에 바로 접근이 가능하다.
            }
        }
        ```
        > interface Expr: 식을 위한 인터페이스로 아무 메소드도 선언하지 않으며 단지 여러 타입의 식 객체를 아우르는 공통 타입 역할만 수행한다.
        >
        > class Num, Sum: Expr 인터페이스를 구현한다
        >
        >  - 인터페이스를 구현하기 위해서 콜론 뒤에 인터페이스 이름을 사용한다!
        - is 를 통해 변수 타입을 검사한다. 자바의 `instanceof`와 비슷하다.
        - **스마트 캐스트는 is로 변수에 든 값의 타입을 검사한 다음 그 값이 바뀔 수 없는 경우에만 작동한다.**
          - 즉, 위의 예시처럼 클래스의 프로퍼티에 대해 스마트 캐스트를 사용한다면 **반드시 val**이어야 하며 **커스텀 접근자**도 사용하면 안된다.
          - 해당 프로퍼티에 대한 접근이 항상 같은 값을 내놓는다고 확신할 수 없기 때문이다.
        - 명시적인 타입 캐스팅을 하려면 `as` 키워드 사용하면 된다.
        > Tip.
        > - IDE에서 스마트 캐스트인 부분을 배경색으로 표시해준다.
    - 위의 예제를 좀 더 코틀린다운 언어로 만들어보자.
        - 먼저, 코틀린다운 if 식을 사용해보자.
            ```kotlin
            fun eval(e: Expr): Int =
                if (e is Num) {
                    println("num: ${e.value}")
                    e.value
                } else if (e is Sum) {
                    eval(e.right) + eval(e.left)
                } else {
                    throw IllegalArgumentException("Unknown expression")
                }
             
            // >>> println(eval(Sum(Num(1), Num(2))))
            // 결과: 3
            ```
        - when을 사용하여 더 코틀린다운 코드로 만들어보자.
            ```kotlin
            fun eval(e: Expr): Int = 
                when (e) {
                    is Num -> {
                        println("num: ${e.value}")
                        e.value // 블록으로 되어있는 경우 마지막 식이 반환값이 된다.
                    }
                    is Sum -> eval(e.right) + eval(e.left)
                    else -> throw IllegalArgumentException("Unknown expression")
                }
            ```

###### 출처

- Kotlin IN Action / 출판사: 에이콘