---
layout: post
title:  "[Kotlin] 코틀린 기초 - 클래스와 프로퍼티"
categories: [Android, Kotlin IN Action]
tags: [Kotlin]
---

> Kotlin IN ACTION(출판사: 에이콘) 책을 통해 Kotlin을 배워보자

### 클래스와 프로퍼티

#### 클래스

- 클래스를 선언해보자
  - 자바로 선언
    ```java
    public class Person {
        private final String name;
    
        pulic Person(String name) {
            this.name = name;
        }
    
        public String getString() {
            return name;
        }
    }
    ```

  - 코틀린으로 선언
    ```kotlin
    class Person(val name: String)
    ```
  > 자바 선언과 코틀린 선언의 차이
  >  - 접근 지정자 생략: 코틀린의 경우 public이 기본이므로 생략해도 된다.
  >  - getter/setter 생략: 코틀린의 경우 getter/setter 메소드를 기본적으로 제공한다.





#### 프로퍼티
- 클래스 안에서 변경 가능한 프로퍼티를 선언해보자
    ```kotlin
    class Person(
        val name: String, 
        var isMarried: Boolean
    )
    ```
  - val name 구문: 읽기 전용 프로퍼티로, 코틀린은 private 필드(default)와 필드를 읽는 public 게터를 만들어낸다.
  - var isMarried 구문: 읽고 쓸 수 있는 프로퍼티로, 코틀린은 private 필드와 public 게터, public 세터를 만들어낸다.
  - 필드의 게터/세터뿐만 아니라 기본적으로 **생성자가 필드를 초기화하는 구현**이 숨겨져있다.
  
#### 클래스의 사용
- 클래스를 사용해보자
  - 자바에서 사용
    ```java
    Person person = new Person("TaeHyungK", false);
    System.out.println(person.getName());
    // 결과: TaeHyungK
    System.out.println(person.isMarried());
    // 결과: false
    ```
  - 코틀린에서 사용
    ```kotlin
    val person = Person("TaeHyungK", false)
    println(person.name)
    // 결과: TaeHyungK
    println(person.isMarried)
    // 결과: false
    ```
    > 코틀린과 자바의 차이
    >   - 코틀린에서는 게터를 호출하는 대신 프로퍼티를 직접 사용한다. (내부적인 로직은 게터와 동일하다.)
    >   - 변경 가능한 프로퍼티의 세터도 동일하다.
    >   - 예를 들어 자바에서 person.setMarried(false) 와 코틀린에서 person.isMarried = false 는 같다.

#### 커스텀 접근자
- 프로퍼티의 접근자를 직접 작성해보자
    ```kotlin
    class Rectangle(val height: Int, val width: Int) {
        val isSquare: Boolean
          get() {
              return height == width
          }     
    }
    ```
    - isSquare 프로퍼티에는 자체 값을 저장하는 필드가 필요 없다. 이 프로퍼티에는 자체 구현을 제공하는 게터만 존재한다. 클라이언트가 프로퍼티에 접근할 때마다 게터가 프로퍼티 값을 매번 다시 계산한다.
        ```kotlin
        val rectangle = Rectangle(41, 43)
        println(rectangle.isSquare)
        // 결과: false
        ```
> 파라미터가 없는 함수를 정의하는 방식 vs 커스텀 게터를 정의하는 방식
>   - 두 방식 모두 비슷하다.
>   - **구현이나 성능상 차이는 없다.**
>   - 가독성의 차이만 있다. 

###### 출처

- Kotlin IN Action / 출판사: 에이콘