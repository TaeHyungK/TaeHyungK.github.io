---
layout: post
title:  "[Kotlin] 코틀린 기초 - while과 for 루프"
categories: [Android, Kotlin IN Action]
tags: [Kotlin]
---

* content
{:toc}

> Kotlin IN ACTION(출판사: 에이콘) 책을 통해 Kotlin을 배워보자

### while과 for 루프

#### while 루프

- while 루프
  - 자바의 while문과 동일하게 사용된다.
  ```kotlin
  while (조건) {
      // TODO 조건이 참인 경우 반복 실행
  }
  
  do {
      // TODO 최초 1회 실행된 후 조건이 참인 경우 반복 실행
  } while(조건)
  ```

#### for 루프

- for 루프
  - 코틀린의 경우 자바의 `for (int i = 0; i < length; i++)`에 해당하는 루프가 없다. 대신, `범위`를 사용한다.
     ```kotlin
     // 범위 1~100 (100을 포함)
     val oneToTen = 1..10
     ```
     - `..`의 경우 우항을 포함한다. 만약 포함하고 싶지 않다면 `until` 을 사용하면 된다.
       - `for (x in 0 until size)` == `for (x in 0..size - 1)` 








  - 피즈버즈 게임 예제
     ```kotlin
     fun fizzBuzz(i: Int) = when {
         i % 15 == 0 -> "FizzBuzz "
         i % 3 == 0 -> "Fizz "
         i % 5 == 0 -> "Buzz "
         else -> "$i " // 위에 해당되지 않는 경우 값 그대로 반환
     }
     
     // 단순한 fizzBuzz 게임
     // >>> for (i in 1..100) {
     //         print(fizzBuzz(i))
     //     }
     // 결과: 1 2 Fizz 4 Buzz Fizz 7 ...
     
     // 100부터 거꾸로 돌며 짝수만으로 fizzBuzz 게임 진행
     // >>> for (i in 100 downTo 1 step 2) {
     //         print(fizzBuzz(i))
     //     }
     // 결과: Buzz 98 Fizz 94 92 FizzBuzz 88 ...
     ```
    
    - `100 downTo 1`: 100에서 1까지 역방향 수열을 만든다.

    - `step 2`: 증가 값의 절대값을 2로 설정한다.
    
    > 개인적으로 아직 익숙하지 않으나 기존 java의 for문보다는 더욱 더 직관적으로 변화된 것 같다.


#### 맵에 대한 이터레이션

- 맵 초기화 후 이터레이션하기
  ```kotlin
  val binaryReps = TreeMap<Char, String>()
  
  for (c in 'A'..'F') {
      val binary = Integer.toBinaryString(c.toInt())
      binaryReps[c] = binary;
  }
  
  for ((letter, binary) in binaryReps) {
      println("$letter = $binary")
  }
  
  // 결과: A = 1000001
  //      B = 1000010
  //      C = 1000011
  //      D = 1000100
  //      E = 1000101
  //      F = 1000110
  ```
  - 맵을 이터레이션 하는 경우 키와 값을 두 변수에 저장하며 이터레이션하게 된다.
    - letter의 경우 키가 들어가며 binary의 경우 값(2진 표현)
  
  - 자바는 put(), get()을 통해 맵에 대한 값을 제어하지만 코틀린의 경우 배열처럼 대괄호를 이용하여 제어하게 된다.
    - `binaryReps[c] = binary` == `binaryReps.put(c, binary)` 
    
  > 객체를 이터레이션 하는 경우 어떻게 되는지 궁금해진다. 객체를 풀어서 각 부분을 분리하는 구조 분해 문법은 7장에서 다루어 보자!

- 컬렉션 이터레이션하기
  ```kotlin
  val list = arrayListOf("10", "11", "1001")
  for ((index, element) in list.withIndex()) {
      print ("$index = $element")
  }
  
  // 결과: 0 = 10
  //      1 = 11
  //      2 = 1001
  ```
  
- `in`을 통해 값이 범위에 속하는지 검사하기
  ```kotlin
  fun isLetter(c: Char) = c in 'a'..'z' || c in 'A'..'Z'
  fun isNotDigit(c: Char) = c !in '0'..'9'
  
  // >>> println(isLetter('q'))
  // 결과: true
  // >>> println(isNotDigit('x'))
  // 결과: true
  ```
  - `!in`: 해당 범위에 포함되지 **않는**지를 검사할 수 있다. 즉, 다른 `!`과 동일하게 부정이다.
  
  - `in`과 `!in` 연산자는 `when`식에서도 사용이 가능하다.
  
  - 내부적으로 `c in 'a'..'z'`는 **c >= 'a' && c <= 'z'** 로 변환된다. 
  
  > ~~헷갈리지만 막상 사용하면 강력한~~ 정규표현식을 사용한 것처럼 꽤나 편리했다. 잘 사용하면 강력한 무기가 되지 않을까 생각이 든다.


###### 출처

- Kotlin IN Action / 출판사: 에이콘