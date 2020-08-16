---
layout: post
title:  "[Kotlin] 코틀린 기초 - 예외(Exception) 처리"
categories: [Android, Kotlin IN Action]
tags: [Kotlin]
---

> Kotlin IN ACTION(출판사: 에이콘) 책을 통해 Kotlin을 배워보자

### 코틀린의 예외처리

코틀린의 예외처리는 자바나 다른 언어의 예외 처리와 비슷하다. 
즉, 함수 실행 중 오류가 발생하면 예외를 던질(throw) 수 있고 함수를 호출하는 쪽에서는 그 예외를 잡아(catch) 처리할 수 있다. 
예외에 대해 처리를 하지 않는 경우 함수 호출 스택을 거슬러 올라가면서 예외를 처리하는 부분이 나올 때까지 예외를 다시 던진다(rethrow).

```kotlin
if (percentage !in 0..100) {
    throw IllegalArgumentException("A percentage value must be between 0 and 100: $percentage")
}
```
  - throw를 위해 예외 인스턴스를 만들 때에도 당연히 `new`키워드가 필요 없다.
  - 자바와 달리 throw는 `식`이므로 다른 식에 포함될 수 있다.
    ```kotlin
    val percentage = 
        if (number in 0..100) {
            number
        } else {
            throw IllegalArgumentException("A percentage value must be between 0 and 100: $percentage")
        }
    ```
    - **문**과 **식**의 차이점은 [[Android] 코틀린 기초 - 함수와 변수](https://taehyungk.github.io/2020/01/11/android-kotlin-basic-1/#%ED%95%A8%EC%88%98) 편을 참고하자.








#### try, catch, finally
- 파일에서 각 줄을 읽어 수로 변환하되 그 줄이 올바른 수 형태가 아닌 경우 null 반환하기
  ```kotlin
  fun readNumber(reader: BufferedReader): Int? {
      try {
          val line = reader.readLine()
          return Integer.parseInt(line)
      } catch (e: NumberFormatException) {
          return null
      } finally {
          reader.close()
      }
  }
  
  // >>> val reader = BufferedReader(StringReader("239"))
  // >>> println(readNumber(reader))
  // 결과: 239
  ```
  - 자바와의 가장 큰 차이점은 `throws` 절이 코드에 없다는 것이다.
    - 자바에서는 함수 뒤에 throws IOException을 붙여야 한다.
    - IOEception이 **체크 예외** 이기 때문으로 자바는 명시적으로 체크 예외를 표현해야 한다.
    - ![스크린샷 2020-02-12 오후 9.33.28](/img/스크린샷 2020-02-12 오후 9.33.28.png)
    
    > 체크 예외와 언체크 예외
    >   - 체크 예외 : Exception 클래스의 서브클래스이면서 RuntimeException 클래스를 상속하지 않은 것들
    >   - 언체크 예외 : RuntimeException을 상속한 클래스
    > 
    > 체크 예외가 발생할 수 있는 메소드를 사용할 경우 반드시 예외를 처리하는 코드를 함께 작성해야 한다.
    > 사용할 메소드가 체크 예외를 던진다면 이를 catch 문으로 잡든지, 아니면 throws를 정의해서 메소드 밖으로 던져야 한다. 그렇지 않으면 컴파일 에러가 발생한다.
    >
    > 반면, 언체크 예외의 경우 명시적인 예외 처리를 강제하지 않기 때문에 언체크 예외라고 불리며 보통 RuntimeException이라고 부른다.

  - 코틀린은 체크 예외와 언체크 예외를 구별하지 않는다.
  
- try를 식으로 사용
  ```kotlin
  fun readNumber(reader: BufferedReader) {
      val number = try {
          Integer.parseInt(reader.readLine())
      } catch (e: NumberFormatException) {
          return // null 로 수정할 겨우 Exception 발생 시 null을 반환하게 된다.
      }
      println(number)
  }
  
  // >>> val reader = BufferedReader(StringReader("not a number"))
  // >>> readNumber(readNumber(reader))
  // 결과: 아무것도 출력되지 않는다.
  ```
  - `try` 키워드는 `if`나 `when`과 마찬가지로 **식**이다. 따라서 try의 값을 변수에 대입할 수 있다.
  - 다만, 반드시 중괄호 {} 로 감싸주어야 한다.
  - 위의 코드에서 Exception이 발생하며 catch로 이동하게 되고 해당 블록에서 return을 하고 있으므로 그 아래 코드는 실행되지 않는다.
         
        

###### 출처

- Kotlin IN Action / 출판사: 에이콘
- [[JAVA] 예외(Exception) 란? 체크예외와 RuntimeException](https://hyeonstorage.tistory.com/199)
  