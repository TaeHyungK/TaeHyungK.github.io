---
layout: post
title:  "[Kotlin] 클래스, 객체, 인터페이스 2탄"
categories: [Android, Kotlin]
tags: [Kotlin]
---

> Kotlin IN ACTION(출판사: 에이콘) 책을 통해 Kotlin을 배워보자

### 클래스, 객체, 인터페이스

#### 가시성 변경자(접근 변경자, access modifier || visibility modifier)

|수식어|클래스 멤버|최상위 선언
|-----|---------|-------------|
|public (기본 가시성)|모든 곳에서 접근 가능|모든 곳에서 접근 가능|
|internal|같은 모듈에서만 접근 가능|같은 모듈에서만 접근 가능|
|protected|같은 클래스 및 하위 클래스에서만 접근 가능|(최상위 선언에 적용 불가)|
|private|같은 클래스에서만 접근 가능|같은 파일 내에서만 접근 가능|








#### 가시성 규칙

```kotlin
interface open class TalkativeButton : Focusable {
    private fun yell() = println("Hey!")
    protected fun whisper() = println("Let's talk!")
}

// 해당 함수는 public
fun TalkativeButton.giveSpeech() {
    yell() // private 메서드 호출 -> 컴파일 에러
    whisper() // protected 메서드 호출 -> 컴파일 에러
}

```
  - 기반 타입 목록에 있는 타입은 자신보다 더 가시성이 같거나 넓어야 한다.
  - 기반 타입의 제네릭 타입 파라미터에 있는 타입은 자신보다 가시성이 같거나 넓어야 한다.
  - 메서드 시그니처에 사용된 모든 타입의 가시성은 메서드의 가시성과 같거나 넓어야 한다. 
  - 위 코드에서는 `giveSpeech()` 메서드를 internal로 변경하거나 각 `TalkativeButton` 클래스를 public으로 변경하여야 한다.

> 자바 코드와의 결합
>   - 코틀린의 public, protected, private 변경자는 컴파일된 자바 바이트코드 안에서도 그대로 유지된다.
>   - 단, private 클래스의 경우 내부적으로 코틀린은 private 클래스를 패키지-전용 클래스로 컴파일한다.
>
> 그렇다면 `internal` 은?
>   - 자바에는 `internal` 과 딱 맞는 가시성이 없기 때문에 바이트 코드상에서는 public이 된다.
 
#### 중첩 클래스

```kotlin
class Button : View {
    override fun getCurrentState(): State = BUttonState()
    override fun restoreState(state: State) { /* TODO */ }

    class ButtonState: State { /* TODO */ } // 자바의 정적(static) 중첩 클래스와 대응
}
```

|클래스 B 안에 정의된 클래스 A|자바에서는|코틀린에서는
|-----------------------|-------|--------|
|중첩 클래스(바깥쪽 클래스에 대한 참조를 저장하지 않음)|static class A|class A|
|내부 클래스(바깥쪽 클래스에 대한 참조를 저장함)|class A|inner class A|

- 자바에서 클래스 내부에 선언된 클래스는 자동으로 inner class가 되며 내부 클래스는 바깥쪽 클래스에 대해 참조를 묵시적으로 포함한다.
- 코틀린에서는 위와 반대로 동작한다. 즉, 클래스 내부에서 선언된 클래스는 바깥쪽 클래스를 묵시적으로 참조하지 않으며 참조를 위해서는 `inner` 를 명시적으로 적어줘야 한다.
  - 내부 클래스에서 바깥쪽 클래스를 참조하려면 `this@<클래스명>` 이라고 써야 한다.
    ```kotlin
    class Outer {
        inner class Inner {
            fun getOuterReference(): Outer = this@Outer
        }
    }
    ```
    
#### sealed 클래스

클래스 계층 정의 시 계층 확장을 제한하는 변경자이다.

```kotlin
sealed class Expr {
    class Num(val value: Int): Expr() // 하위 클래스를 중첩 클래스로 나열
    class Sum(val left: Expr, val right: Expr): Expr() // 하위 클래스를 중첩 클래스로 나열
}

fun eval (e: Expr): Int = 
    // when 식이 Expr이 가지는 모든 하위 클래스를 검사하므로 else 문이 별도로 필요하지 않음!!
    when (e) {
        is Expr.Num -> e.value
        is Expr.Sum -> eval(e.right) + eval(e.left)
    }
```
- `sealed` 로 표시된 클래스는 자동으로 open 이다.

> class Num (val value: Int) : Expr() 의 의미는?
>   - **public** 한 class Num을 정의
>   - class Num은 변경이 불가능한 멤버 변수 value를 가지며 해당 변수는 **private** 하고 **public** 한 getter를 가진다.
>   - class Num을 생성하기 위해서는 `Num(value)` 와 같이 생성해주면 된다.

#### 뻔하지 않은 생성자와 프로퍼티

**주 생성자**: 클래스를 초기화할 때 주로 사용하는 간략한 생정자로, 클래스 본문 밖에서 정의한다.

**부 생성자**: 클래스 본문 안에서 정의한다.

#### 클래스 초기화: 주 생성자와 초기화 블록

```kotlin
class User constructor(_nickname: String) { // 파라미터가 1개인 주 생성자
    val nickname: String
    init {
        // 초기화 블록
        nickname = _nickname
    }
}

// 프로퍼티를 생성자 파라미터로 초기화
// 별다른 어노테이션이나 가시성 변경자가 없다면 constructor 키워드 생략 가능
class User(_nickname: String) {
    val nickname = _nickname    
}

// 파라미터로 프로퍼티를 바로 초기화
class User(val nickname: String)

// 디폴트값 설정
class User(val nickname: String, 
            val isSubscribed: Boolean = true)

// 인스턴스 생성 시 new 키워드 없이 직접 호출
// >>> val txxbro = User("txxbro")
// >>> println(txxbro.isSubscribed)
// 결과: true
```

주 생성자﴾primary constructor﴿
  - 클래스 이름 뒤에 오는 괄호로 둘러싸인 코드
    - 생성자 파라미터 지정
    - 생성자 파라미터로 초기화할 프로퍼티를 정의
  - init 블록이나 프로퍼티 초기화 식에서만 주 생성자의 파라미터 참조 가능
  - 생성자 파라미터에 디폴트 값 가능
    - 모든 파라미터에 디폴트 값 지정하면, 파라미터 없는 생성자를 만들어줌
 
#### 기반 클래스 생성자 호출
```kotlin
open class User(val nickname: String) { /* TODO */ }

class TwitterUser(nickname: String): User(nickname) { /* TODO */ }
```

- 주 생성자에서 기반 클래스의 생성자를 호출해야 한다.
  - 기반 클래스 이름 뒤에 괄호를 통해 파라미터를 넘긴다.

#### 디폴트 생성자
```kotlin
open class Button

// 기반 클래스의 인자가 없는 디폴트 생성자 호출
class RadioButton: Button()
```

- 클래스 정의 시 별도 생성자를 정의하지 않으면 컴파일러가 자동으로 아무 일도 하지 않는 인자가 없는 디폴트 생성자를 만들어 준다.

#### 수식어와 생성자
```kotlin
class Secretive private constructor() { /* TODO */ }
```

- 생성자에 수식어를 붙이려면 `constructor` 키워드가 필요하다.
- 위와 같이 정의 시 주 생성자는 비공개(private)이 된다.

#### 부 생성자: 상위 클래스를 다른 방식으로 초기화

여러가지 방법으로 인스턴스를 초기화할 수 있게 다양한 생성자를 지원해야 하는 경우 **부 생성자**를 사용하게 된다.

```kotlin
open class View {
    // 부 생성자1
    constructor(ctx: Context) { /* TODO */ }
    // 부 생성자2
    constructor(ctx: Context, attr: AttributeSet) { /* TODO */ }
}

class MyButton: View {
    // 상위 클래스의 생성자 호출
    constructor(ctx: Context): super(ctx) { /* TODO */ }
    // 상위 클래스의 생성자 호출
    constructor(ctx: Context, attr: AttributeSet): super(ctx, attr) { /* TODO */ }
}
``` 

- `super()` 키워드를 통해 상위 클래스 생성자에게 객체 생성을 **위임** 한다.
- 자바와 동일하게 `this()` 를 통해 해당 클래스의 다른 생성자에게 객체 생성을 위임할 수도 있다.

#### 부 생성자와 주 생성자

```kotlin
open class View(ctx: Context, attr: AttributeSet, type: Int) {
    constructor(ctx: Context): this(ctx, AttributeSet(), 0)
    constructor(ctx: Context, attr: AttributeSet): this(ctx, attr, 0)
}
```

- 클래스에 **주 생성자가 없다면** 모든 부 생성자는 반드시 상위 클래스를 초기화하거나 다른 생성자에게 생성을 위임해야 한다.
- **주 생성자가 있다면** 직접 또는 간접적으로(다른 부 생성자를 통해) 주 생성자를 호출해야 한다.

#### 인터페이스에 선언된 프로퍼티 구현

추상 프로퍼티 선언이 들어있는 인터페이스 선언

```kotlin
interface User {
    val nickname: String
}
```

- 인터페이스에 있는 프로퍼티 선언에는 뒷받침하는 필드(backing field) 나 게터 등의 정보가 들어있지 않다.
  - 인터페이스를 구현하는 하위 클래스에서 상태 저장을 위한 프로퍼티 등을 만들어야 한다.

```kotlin
class PrivateUser(override val nickname: String) : User // 생서자의 프로퍼티로 구현
class SubscribingUser(val email: String) : User {
    override val nickname: String
        get() = email.substringBefore('@') // 커스텀 게터
}
class FacebookUser(val accountId: Int) : User {
    override val nickname: String = getFacebookName(accountId) // 프로퍼티 초기화 식
}

// >>> println(PrivateUser("test@kotlinlang.org").nickname)
// 결과: test@kotlinlang.org
// >>> println(FacebookUser("test@kotlinlang.org").nickname)
// 결과: test
```

- SubscribingUser는 **커스텀 게터**를 사용하며 해당 프로퍼티에 접근할 때마다 매번 새로 계산한다.
- 반면, FacebookUser의 경우 해당 프로퍼티에 값을 초기화한 후 해당 값을 가져다가 쓴다.

#### 인터페이스에서 게터와 세터가 있는 프로퍼티 선언
```kotlin
inerface User {
    val email: String
    val nickname: String
        get() = email.substringBefore('@') // 프로퍼티를 뒷받침하는 필드가 없다. 대신 매번 결과르라 계산해 반환한다.
}
```

- 위 인터페이스를 구현한 하위 클래스는 추상 프로퍼티인 "email"을 반드시 오버라이드해야 한다.
- 반면, 커스텀 게터가 있는 "nickname"의 경우 오버라이드하지 않고 상속할 수 있다.

#### 게터와 세터에서 뒷받침하는 필드에 접근
```kotlin
class User(val name: String) {
    val address: String = "unspecified"
        set(value: String) {
            println("""
                Address was changed for $name:
                "$field" -> "$value".""".trimIndent()) // 뒷받침하는 필드 값 읽기
            field = value // 뒷받침하는 필드 값 변경하기
        }
}

// >>> val user = User("txxbro")
// >>> user.address = "Seoul, 1234-123"
// 결과: Address was changed for txxbro:
//      "unspecified" -> "Seoul, 1234-123".
```

접근자의 본문에서 `field` 라는 **특별한 식별자**를 통해 뒷받침하는 필드에 접근할 수 있다.

- 게터에서는 `field` 값을 읽을수만 있고, 세터에서는 field 값을 읽거나 쓸 수 있다.
- 컴파일러는 게터나 세터에서 `field` 에 접근하면 뒷받침하는 필드를 생성해준다.
- `field` 에 접근하지 않는 커스텀 게터/세터 구현 시에는 뒷받침하는 필드를 생성하지 않는다.

#### 접근자의 가시성 변경

기본적으로는 접근자의 가시성은 프로퍼티의 가시성과 같다. 다만, 원한다면 `get`이나 `set` 앞에 가시성 변경자를 추가해서 접근자의 가시성을 변경할 수 있다.

```kotlin
class LengthCounter {
    var counter: Int = 0
        private set // 이 클래스 밖에서 이 프로퍼티의 값을 바꿀 수 없다.
    fun addWord(word: String) {
        counter += word.length
    }
}

// >>> val lengthCounter = LengthCounter()
// >>> lengthCounter.addWord("Hi!")
// >>> println(lengthCounter.counter)
// 결과: 3
```

###### 출처

- Kotlin IN Action / 출판사: 에이콘
  