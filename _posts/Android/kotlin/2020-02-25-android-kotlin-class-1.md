---
layout: post
title:  "[Kotlin] 클래스, 객체, 인터페이스 1탄"
categories: [Android]
tags: [Kotlin]
---

* content
{:toc}

> Kotlin IN ACTION(출판사: 에이콘) 책을 통해 Kotlin을 배워보자

### 클래스, 객체, 인터페이스

#### 인터페이스

인터페이스
```kotlin
interface Clickable { 
    fun click()
}
```

인터페이스 구현
```kotlin
class Button: Clickable {
    override fun click() = println("I was clicked!")
}
```

- 클래스 이름 뒤에 세미콜론(:)을 붙이고 인터페이스와 클래스 이름을 적는 것으로 클래스 확장과 인터페이스 구현을 모두 처리한다.
  - 단, 자바와 동일하게 **인터페이스**는 갯수 제한 없으며 **클래스**는 한 개만 상속이 가능

- `override 변경자`: 상위 클래스나 인터페이스에 있는 프로퍼티 또는 메소드를 오버라이드 한다는 표시로 **재정의시 꼭** 사용하여야 한다.









디폴트 구현이 있는 인터페이스
```kotlin
interface Clickable {
    fun click()
    fun showOff() = println("I'm clickable. showOff()")
}
```

- 메소드 본문을 메소드 시그니처 뒤에 추가하면 된다.

동일한 시그니처를 가진 인터페이스를 한 클래스에서 상속 시
```kotlin
interface Clickable {
    fun click()
    fun showOff() = println("I'm clickable")
}

interface Focusable {
    fun showOff() = println("I'm focusable")
}
```

- 동일한 시그니처 `showOff()`를 가진 인터페이스 2개를 한 클래스에서 상속받을 경우 **컴파일 에러**가 발생하게 된다.
  - 즉, 어떠한 메소드를 사용할지 직접 제공해야 한다.
  
    ```kotlin
    class Button : Clickable, Focusable {
        override fun click() = println("I was clicked")
    
        // 어떤 메소드를 사용할지 선택하기 위해 재정의
        override fun showOff() {
            // super<타입>으로 사용할 상위 타입 지정
            super<Clickable>.showOff()
            super<Focusable>.showOff()
        }
    }
    ```
    > 위 코드에서 자바와의 차이점?
    >   - super (상위 클래스) 호출 시 꺽쇠 괄호(<>) 안에 타입을 지정한다.
    >      - 자바: `Clickable.super.showOff()`
    >      - 코틀린: `super<Clickable>.showOff()`

자바에서 코틀린의 메소드가 있는 인터페이스 구현하기
  - 코틀린은 자바 6와 호환되게 설계가 되어있다.
  - 코틀린은 디폴트 메소드가 있는 인터페이스를 **일반 인터페이스 + 디폴트 메소드 구현이 정적 메소드로 들어있는 클래스** 를 조합해 구현한다.
  - 즉, 자바에서 디폴트 인터페이스가 포함된 코틀린 인터페이스를 상속해 구현하고 싶다면 **메소드 본문을 포함하고 있는 메소드를 포함하는 모든 메소드에 대한 본문을 작성**해야 한다.


#### open, final, abstract 변경자

```kotlin
open class RichButton : Clickable {
    fun disable() {} // final 메소드. 하위 클래스에서 오버라이드 불가
    open fun animate() {} // 하위 클래스에서 오버라이드 가능한 메소드
    override fun click() {} // override 한 메소드는 기본적으로 open (이미 상속을 받았기 때문에 open!)
}
```

- 클래스와 클래스의 멤버는 기본적으로 `final`
  - 상속을 허용하려면 `open 변경자` 를 붙여 허용 가능하도록 명시해주어야 한다.
  - 오버라이딩을 허용하고 싶은 메소드나 프로퍼티 앞에도 `open 변경자` 를 붙여야 한다.
  
- override 한 메소드를 재정의 시 하위 클래스에서 override를 막고 싶다면 `final`을 명시해야 한다. 
  
> 코틀린에서 기본적으로 final 인 이유?
>   - 하위 클래스에서 부모 클래스를 상속받은 후 특정 메소드를 override 하였을 때 추후 부모 클래스가 수정되면 부모 클래스에서는 모든 하위 클래스를 분석할 수 없다. **즉, 부모 클래스가 변경되는 경우 하위 클래스의 동작이 예기치 못하게 변경될 수 있다는 뜻**이다.
>   - 코틀린에서는 위와 같은 상황을 막기위해 클래스와 메소드들은 기본적으로 `final`로 채택되었다.

- 클래스를 `abstract` 로 선언하면 추상 클래스
  - `abstract` 한 멤버는 항상 `open` 이다. 따라서 별도로 `open` 을 명시할 필요가 없다.
```kotlin
abstract class Animated {
    abstract fun animate() // 추상 메소드로 하위 클래스에서 반드시 오버라이드 해야 한다.
    
    open fun stopAnimating() {} // 추상 클래스에 포함된 비추상 클래스로 기본적으로 final 이지만 open 키워드를 통해 오버라이드를 허용할 수 있다.

    fun animateTwice() {} // 위와 동일하다. 추상 클래스 내의 멤버는 기본적으로 open 이다!
}
```

클래스 내에 Access Modifier(상속 접근 제어 변경자)

|변경자|override|설명
|-----|---------|-------------|
|final|오버라이드 불가|클래스 멤버의 기본 변경자|
|open|오버라이딩 가능|`open` 을 명시해야 오버라이딩 가능|
|abstract|오버라이딩 필수|추상 클래스의 멤버에만 붙일 수 있음, 구현이 필수|
|override|상위 클래스나 상위 인스턴스의 멤버를 오버라이딩|오버라이딩하는 멤버는 기본적으로 `open`. 금지가 필요하다면 `final` 명시 필요|


###### 출처

- Kotlin IN Action / 출판사: 에이콘
  