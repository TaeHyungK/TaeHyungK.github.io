---
layout: post
title:  "[Kotlin] Chapter4. 클래스, 객체, 인터페이스 총정리"
categories: [Android, Kotlin]
tags: [Kotlin]
---

> Kotlin IN ACTION(출판사: 에이콘) 책을 통해 Kotlin을 배워보자

## 목차

- 클래스와 인터페이스
- 뻔하지 않은 생성자와 프로퍼티
- 데이터 클래스
- 클래스 위임
- object 키워드 사용








## [클래스와 인터페이스]

### 4.1.1 코틀린 인터페이스

- 인터페이스 안에는 추상 메소드 뿐 아니라 구현이 있는 메소드도 정의 가능하다. 다만 아무런 상태(필드)도 들어갈 수 없다.

```kotlin
interface Clickable{
  fun onClick()
  fun showOff() = println("Show~") // 디폴트 구현. 
}

class Button : Clickable {
  override fun onClick() = println("Hi~")
}

Button().onClick()
```

- 클래스 이름 뒤에 콜론(:)을 붙여 인터페이스와 클래스 이름을 적는 것으로 상속과 인터페이스 구현을 모두 처리한다.
  - 자바 : extends, implements
- 자바와 마찬가지로 다중 구현은 허용되지만, 다중 상속은 불가능하다.
- @Override 어노테이션과 override 변경자는 상위 클래스나 상위 인터페이스에 있는 프로퍼티나 메소드를 오버라이드 한다는 뜻이다. 다만, override 변경자는 코틀린에서 꼭 표시해야 한다.
- 디폴트 구현을 제공할 수 있으며, 특별한 키워드를 붙이지 않고 메소드 본문을 적어주면 된다.
  - 이 경우, Clickable을 구현하는 하위 클래스에서 showOff를 새롭게 정의할 수도 있고, 정의를 생략해서 디폴트 구현을 사용할 수도 있다.
- 아래 코드를 함께 보자.

```kotlin
interface Focusable{
  fun setFocus() = ...
  fun showOff() = println("I'm Focus")
}
```

- 이제 한 클래스에서 Clickable, Focusable  두 인터페이스를 구현하면 어떻게 될까? 두 인터페이스 모두 디폴트 구현이 들어있다. 
- 결론은 어느 쪽의 showOff() 메소드도 호출되지 않는다. 클래스가 구현하는 두 상위 인터페이스에 showOff() 구현을 대체할 오버라이딩 메소드를 직접 제공하지 않으면 아래와 같은 컴파일 오류가 발생한다.

> The class 'Button' must override public open fun showOff() 
>
> because it inherits many implementations of it.



- 코틀린 컴파일러는 두 메소드를 아우르는 구현을 하위 클래스에 강제한다.

```kotlin
class Button: Clickable, Focusable{
  override fun onClick() = ...
  override fun showOff(){
    super<Clickable>.showOff()
    super<Focusable>.showOff()
  }
}
```



- 즉, 이름과 시그니처가 같은 멤버 메소드에 대해 둘 이상의 디폴트 구현이 존재하는 경우, 인터페이스를 구현하는 하위 클래스에서 명시적으로 새로운 구현을 제공해야 한다.
- 자바와 달리 상위 타입의 이름을 꺽쇠 괄호(<>) 사이에 넣어서 super를 지정하면 어떤 상위 타입의 멤버 메소드를 호출할지 지정할 수 있다.





### 4.1.2 open, final, abstract 변경자

- 자바에서는 기본적으로 상위 클래스에 대해 하위 클래스에서 상속하는 걸 막지 않는다. 막기 위해서는 final을 붙여 상속을 할 수 없게 한다.
- 기본적으로 상속이 가능하면 편리하지만, 문제가 생기는 경우도 있다.
- `취약한 기반 클래스` 라는 문제는 하위 클래스가 기반 클래스에 대해 가졌던 가정이 기반 클래스를 변경함으로써 깨져버린 경우에 생긴다. 어떤 클래스가 자신을 상속하는 방법에 대해 정확한 규칙을 제공하지 않는다면 그 클래스의 클라이언트는 기반 클래스를 작성한 사람의 의도와 다른 방식으로 메소드를 오버라이드할 위험이 존재한다. 
- 즉, 기반 클래스를 변경하는 경우 하위 클래스의 동작이 예기치 않게 바뀔 수도 있다는 면에서 기반 클래스는 **취약**하다.
- 이런 점을 보완하기 위해 코틀린에서는 클래스와 메소드는 기본적으로 **final**이다.
- 클래스의 상속을 허용하려면 클래스 앞에 **open** 변경자를 붙여야 한다. 
- 오버라이드를 허용하고 싶은 메소드나 프로퍼티 앞에도 붙일 수 있다.

```kotlin
open class RichButton : Clickable{
  fun disable()
  
  open fun animate()
  
  override fun onClick()
  
  final override fun onClick() 
  // 오버라이드한 메소드의 구현을 하위 클래스에서 오버라이드 하지 못하게 막을 수 있다.
}
```



- RichButton 클래스는 다른 클래스가 상속할 수 있다.(open)
- disable() : final이며, 오버라이드 할 수 없다.
- animate() : oepn으로 오버라이드 가능.
- onClick() : 상위 클래스에서 선언된 열려있는 메소드를 오버라이드 한다. 오버라이드한 메소드는 기본적으로 열려있다.



- abstract로 선언한 추상 클래스는 인스턴스화 할 수 없으며, 추상 클래스에는 구현이 없는 추상 멤버가 있기 때문에 하위 클래스에서 그 추상 멤버를 오버라이드 해야 하는게 보통이다. 
- 추상 멤버는 항상 열려 있기 때문에 open 변경자를 명시할 필요가 없다.

Todo 표 추가



### 4.1.3 가시성 변경자

- 코틀린의 기본 가시성은 자바와 다르며, 아무 변경자도 없는 경우 public이다.
- 자바의 기본 가시성인 package private이 없다. 코틀린은 패키지를 네임스페이스를 관리하기 위한 용도로만 사용한다.
- 코틀린은 `internal`이라는 새로운 가시성을 도입했다.
  - 이는 모듈 내부에서만 볼 수 있음을 뜻한다. 
  - 모듈 : 한 번에 한꺼번에 컴파일되는 파일들을 의미한다.
  - 모듈 내부 가시성은 모듈의 구현에 대해 진정한 캡슐화를 제공한다는 장점이 있다. 
- 코틀린에서는 최상위 선언에 대해 private 가시성을 허용한다.

>? 모듈에 대해 생각해볼 필요가 있어 보인다.



| 변경자    | 클래스멤버                       | 최상위 선언                    |
| --------- | -------------------------------- | ------------------------------ |
| public    | 모든 곳에서 볼 수 있다.          | 모든 곳에서 볼 수 있다.        |
| internal  | 같은 모듈 안에서만 볼 수 있다.   | 같은 모듈 안에서만 볼 수 있다. |
| protected | 하위 클래스에서만 볼 수 있다.    | 적용할 수 없음.                |
| private   | 같은 클래스 안에서만 볼 수 있다. | 같은 파일 안에서만 볼 수 있다. |



Ex)

```kotlin
internal open class TalkativeButton: Focusable{
  private fun yell() = println("Hey~")
  protected fun wishper() = println("Let's talk!")
}

fun TalkativeButton.giveSpeech(){
  yell()
  
  whisper()
}
```

- public 멤버가 자신의 internal 수신 타입인 TalkativeButton을 노출함
  - Public 함수인 giveSpeech 안에서 그보다 가시성이 더 낮은 internal 타입인 TalkativeButton을 참조하지 못하게 한다. 
  - 이는 어떤 클래스의 기반 타입 목록에 들어있는 타입이나 제네릭 클래스의 타입 파라미터에 들어있는 타입의 가시성은 그 클래스 자신의 가시성과 같거나 더 높아야 한다.
  - 이는 일반적인 규칙에 해당한다.
- yell은 private이라 접근이 불가능.
- whisper는 상속 관계에서 하위 클래스만 접근이 가능. 따라서 불가능.

- 컴파일 오류를 없애기 위해서는 TalkativeButton을 public으로 바꾸거나 giveSpeech 확장 함수의 가시성을 internal로 바꾸면 된다.
- **자바와 달리 코틀린에서의 protected 멤버는 오직 어떤 클래스나 그 클래스를 상속한 클래스 안에서만 보인다.**
- **클래스를 확장한 함수는 그 클래스의 private, protected 멤버에 접근할 수 없다는 사실을 알아야 한다.**



> 코틀린의 가시성 변경자와 자바

코틀린의 public, protected, private 변경자는 컴파일된 자바 바이트 코드 안에서도 그대로 유지된다. 

유일한 예외는 private이며, 자바에서는 클래스를 private으로 만들 수 없으므로 코틀린은 내부적으로 private 클래스는 package private으로 컴파일한다.



internal은 자바에서 딱 맞는 가시성이 없다. package private과는 다르다. 모듈은 보통 여러 패키지로 이뤄지며 서로 다른 모듈에 같은 패키지에 속한 선언이 들어있을 수도 있다. 따라서 internal은 자바 바이트 코드에서 public이 된다.



코틀린과 자바 선언에 차이가 존재하기 때문에 다음과 같은 접근이 가능하다.

- 다른 모듈에 정의된 internal 클래스나 internal 최상위 선언을 모듈 외부의 자바 코드에서 접근 가능.
- protected로 정의한 멤버를 코틀린 클래스와 같은 패키지에 속한 자바 코드에서 접근 가능. 



### 4.1.4 내부 클래스와 중첩된 클래스

```kotlin
interface State: Serializable

interface View{
  fun getCurrentState(): State
  fun restoreState(state: State){}
}

class Button : View{
  override fun getCurrentState(): State = ButtonState()
  
  override fun restoreState(state: State){
    ...
  }
  
  class ButtonState: State{
    ...
  }
  // 내부 클래스
  inner ButtonState: State{
    
  }
}
```

- 코틀린에서 ButtonState는 중첩 클래스에 해당되며 아무런 변경자가 붙지 않으면 자바의 static 중첩 클래스와 같다. 따라서 바깥쪽 클래스에 대한 참조가 없고, 이로 인해서 직렬화가 가능하다. 
- 이를 내부 클래스로 변경해서 바깥쪽 클래스에 대한 참조를 포함하고 싶으면 `inner` 변경자를 붙이면 된다. 이렇게 되면 **NoSerializableException: Button** 이라는 예외가 발생한다. 
  - 왜 Button을 직렬화할 수 없을까? 
  - 내부 클래스는 바깥쪽 클래스에 대한 참조를 포함한다. 그 참조로 인해 직렬화할 수 없다. Button 클래스가 직렬화할 수 있는 상태가 아니기 때문에 결론적으로 ButtonState도 직렬화할 수 없는 것이다.

> 📌 알고 넘어가기~

- 내부 클래스 : 바깥쪽 클래스에 대한 참조를 갖는다.
- 중첩 클래스 : 바깥쪽 클래스에 대한 참조를 갖지 않는다. 



| 클래스 B안에 정의된 클래스 A                           | 자바에서는     | 코틀린에서는  |
| ------------------------------------------------------ | -------------- | ------------- |
| 중첩 클래스(바깥쪽 클래스에 대한 참조를 저장하지 않음) | Static class A | class A       |
| 내부 클래스(바깥쪽 클래스에 대한 참조를 저장함)        | class A        | inner class A |



### 4.1.5 Sealed Class

- 기존에는 클래스 계층에 새로운 하위 클래스를 추가하더라도 컴파일러가 when이 모든 경우를 처리하는지 제대로 검사할 수 없다. 그래서 새로운 클래스에 대한 처리를 잊어버리면 디폴트 분기가 선택되기 때문에 버그가 발생할 가능성이 높다. 
- 해법을 제공한다. 클래스에 sealed 변경자를 붙이면 그 상위 클래스를 상속한 하위 클래스 정의를 제한할 수 있다. 

- sealed 클래스의 하위 클래스를 정의할 때는 반드시 상위 클래스 안에 중첩시켜야 한다.
- Sealed 클래스는 자동으로 open이다.

```kotlin
sealed class Expr{
  class Num(val value: Int): Expr()
  class Sum(val left: Expr, val right: Expr): Expr()
}

fun eval(e: Expr): Int = 
	when(e){
    is Expr.Num -> e.value
    is Expr.Sum -> eval(e.left) + eval(e.right)
  }
```



- When 식이 모든 하위 클래스를 검사하므로 else 분기가 없어도 된다. 
- 클래스 외부에 sealed 클래스 자신을 상속한 클래스를 둘 수 없다. 
- 나중에 sealed 클래스의 상속 계층에 새로운 하위 클래스를 추가하면 when 식이 컴파일되지 않는다. 따라서 식을 고쳐야 한다는 사실을 파악하기 쉽다.
- 내부적으로 Expr 클래스는 private 생성자를 갖는다. 그 생성자는 클래스 내부에서만 호출할 수 있다. 



## [뻔하지 않은 생성자와 프로퍼티]

- 주 생성자 : 클래스를 초기화할 때, 주로 사용하는 간략한 생성자로 클래스 본문 밖에서 정의한다.
- 부 생성자 : 클래스 본문 안에서 정의한다.

### 4.2.1 클래스 초기화 : 주 생성자와 초기화 블록

```kotlin
class User(val name: String)
```

- 중괄호도 없고 괄호 사이에 val만 존재한다.
- 이처럼 클래스 이름 뒤에 오는 괄호로 둘러싸인 코드는 **주 생성자**라 부른다. 
- 주 생성자는 생성자 파라미터를 정의하고 그 생성자 파라미터에 의해 초기화되는 프로퍼티를 정의하는 두 가지 목적에 쓰인다.



```kotlin
class User constructor(_nickname: String){
  val nickname: String
  init{
    nickname = _nickname
  }
}
```

- Constructor : 주 생성자나 부 생성자 정의를 할 때 사용되며, 주 생성자의 경우 어노테이션이나 가시성 변경자가 없으면 생략해도 된다. 
- Init : 초기화 블록으로 클래스의 객체가 만들어질 때, 실행될 초기화 코드가 들어간다.
- 초기화 블록은 주 생성자와 함께 쓰인다. 이유는 주 생성자가 제한적이기 때문에 별도의 코드를 포함할 수 없기 때문!



```kotlin
class User(_nickname: String){
  val nickname = _nickname
}
```

- 이처럼 초기화 블록 없이 프로퍼티 선언에 초기화를 포함시킬 수 있다.
- 그렇다면 더 간단하게 할 수 있을까? 아래의 코드를 보자.



```kotlin
class User(val nickname: String){
  ...
}

fun main(args: Array<String>){
  val lee = User("VictoryWoo") // new 없이 바로 생성자 호출!
}
```

- 주 생성자의 파라미터로 프로퍼티를 초기화한다면 그 주 생성자 파라미터 이름 앞에 val을 추가하는 방식으로 프로퍼티 정의와 초기화를 간략히 쓸 수 있다.
- 함수 파라미터와 마찬가지로 생성자 파라미터에도 디폴트 값을 사용할 수 있다.



```kotlin
open class User(val name: String){ ... }

class TwitterUser(name: String) : User(name){
  ...
}
```

- 상위 클래스 초기화를 위해서 클래스 뒤에 괄호 안에 생성자로 인자를 넘긴다.



```kotlin
open class Button
interface Click

class RadioButton: Button()
class CustomView : Click
```

- Button : 별도의 생성자를 정의하지 않아 컴파일러가 자동으로 아무 일도 하지 않는 인자 없는 디폴트 생성자를 만든다.
- RadioButton : Button 클래스를 상속했기 때문에 Button의 생성자를 호출해야 한다.
- CustomView : Click이 인터페이스이므로 이름만 명시.



### 4.2.2 부 생성자

- 코틀린의 디폴트 값 + 이름 있는 인자를 사용해 생성자가 여럿 있는 경우 처리가 가능하다.
- **인자에 대한 디폴트 값을 제공하기 위해 부 생성자를 여럿 만들지 말고 대신 파라미터의 디폴트 값을 생성자 시그니처에 명시하라.**

```kotlin
open class View{
  constructor(context: Context){
    ...
  }
  
  constructor(context: Context, attr: AttributeSet){
    ...
  }
}

class MyButton : View{
  constructor(context: Context) : super(context){
    ...
  }
  
  // 디폴트 값을 넘겨 같은 클래스의 다른 생성자 호출. 
  constructor(context: Context) : this(context, WOO_STYLE){
    ...
  }
  
  constructor(context: Context, attr: AttributeSet)
  	: super(context, attr){
      ...
    }
}
```

- MyButton에서는 super()를 통해 상위 클래스의 생성자를 호출함으로써 객체 생성을 위임한다.
- 클래스에 주 생성자가 없다면 모든 부 생성자는 반드시 상위 클래스를 초기화하거나 다른 생성자에게 생성을 위임해야 한다. 



> 프로퍼티란?

- 코틀린은 프로퍼티를 언어의 기본 기능으로 제공.

- 이는 필드와 접근자를 통칭하는 것이다.
- 즉, 데이터를 저장하고 get, set이 가능함을 의미한다.



```kotlin
class Person(val name: String, var isMarried: Boolean)
```



- 코틀린의 기본 가시성은 public이기 때문에 getter, setter도 동일하게 가져간다. 그런데, 필드에 저장된 데이터의 가시성은 public이 아니다. **생성자에 선언된 데이터는 private이 된다.** 

`Q. 외부에서 Person 객체를 생성한 뒤, name, isMarried에 접근이 가능할까??`

— 고민의 시간 — 



- **[반면, 일반 클래스 필드에 넣은 데이터는 private으로 지정해주지 않을 경우 public이 된다.]** 라고 하는데 확인해 본 결과 필드에 넣은 데이터 또한 private으로 선언되며, getter가 public으로 지정된다.
- **선언된 데이터가 private이 된다는 것은 자바 코드로 변환했을 때, 필드의 선언이 private이 되며 getter, setter는 public인 상태가 된다는 것을 의미한다.** 만약, 생성자의 프로퍼티를 private으로 지정한다면 getter, setter도 private이 되어 접근이 불가능하다. (위의 질문에 대한 답이 된다.)



**[생성자 파라미터의 val, var의 차이]**

```kotlin
class Person(val name: String) // 1

class Person(name: String) // 2
```



1번의 경우 자바 코드로 변환되면 아래와 같다.

![스크린샷 2020-02-27 오후 3.09.35](/Users/woo/Desktop/스크린샷 2020-02-27 오후 3.09.35.png)



- 생성자에 있는 name은 프로퍼티가 되며, 외부에서도 접근이 가능하다.



2번의 경우 자바 코드로 변환시 아래와 같다.

![스크린샷 2020-02-27 오후 3.10.03](/Users/woo/Desktop/스크린샷 2020-02-27 오후 3.10.03.png)



- 이 경우 name은 생성자에서 프로퍼티를 초기화하는 역할만을 하고, 사용할 수 없다. 따라서 클래스의 생성자 외 다른 메소드에서 사용할 수 없다. (프로퍼티가 아니기 때문)



**[주의할 점]**

위에서 설명했지만, 한번 더 짚고 넘어간다. 

디컴파일한 자바 코드를 보면 필드가 private으로 되어 있는 것을 볼 수 있다. 

이렇다고 하더라도 코틀린의 프로퍼티가 private은 아니다. 여기서 우리는 **필드와 프로퍼티를 다르게 인식할 줄 알아야 한다.** 자바는 기본적으로 필드로 다루고, 코틀린은 프로퍼티(필드+접근자)를 기본으로 다루는 언어이기 때문에 약간의 차이가 존재한다.



```kotlin
class Person(val name: String)
```



```java
public class Person{
  private String name;
  
  public Person(String name) {
        this.name = name;
  }

  public Void setName(String value) {
        this.name = value;
  }

  public String getName() {
        return this.name;
  }
}
```



필드인 name 자체만 보면 private 키워드가 붙어있으므로 private이 맞지만, 프로퍼티 전체를 보면 다르다. getter/setter로 접근이 모두 가능하기 때문에 프로퍼티는 private하다고 볼 수 없다.

위의 코드에서 name 프로퍼티가 private이기 위해서는 아래와 같이 수정해야 한다.

```kotlin
class Person(private val name: String)
```



```java
public final class Person {
   private String name;

   public Property(@NotNull String name) {
      Intrinsics.checkParameterIsNotNull(name, "name");
      super();
      this.name = name;
   }
}
```



getter/setter가 없어 프로퍼티는 private이라고 볼 수 있다.

- [참고](https://wooooooak.github.io/kotlin/2019/05/24/property/)



### 4.2.3 인터페이스에 선언된 프로퍼티 구현

```kotlin
interface User{
  val name: String
}
```

- 인터페이스에 추상 프로퍼티 선언이 있고, 상태를 저장하기 위해서는 해당 인터페이스를 구현한 하위 클래스에서 상태 저장을 위한 프로퍼티 등을 만들어야 한다.
- 아래는 각기 다른 방식으로 이를 구현한다.



```kotlin
class PrivateUser(override val nickname: String): User
// 주 생성자에 있는 프로퍼티

class SubscribingUser(val email: String): User{
  override val name: String
  	get() = email.substringBefore('@')
}
// 커스텀 게터

class FacebookUser(val accountId: Int): User{
  override val nickname = getFacebookNam(accountId)
}
// 프로퍼티 초기화 식
```



- **SubscribingUser** : nickname은 매번 호출될 때마다 substringBefore()를 호출해 새로운 값을 계산하는 커스텀 getter를 사용한다.
- **FacebookUser** : nickname은 객체 초기화 시 계산한 데이터를 저장했다가 불러오는 방식을 이용한다.



### 4.2.4 게터와 세터에서 뒷받침하는 필드 접근

```kotlin
class User(val name: String) {
    var address: String = "unspecified"
        set(value: String) {
            println("""
                Address was changed for $name:
                "$field" -> "$value".""".trimIndent())
            field = value
        }
}

fun main(args: Array<String>) {
    val user = User("Alice")
    user.address = "Elsenheimerstrasse 47, 80687 Muenchen"
}

```



- setter 접근자의 본문에서 field를 통해 뒷받침하는 필드에 접근할 수 있다.(**address**)
- getter는 field 값을 읽을 수만 있고, setter는 field 값을 읽거나 쓸 수 있다.



### 4.2.5 접근자의 가시성 변경

```kotlin
class LengthCounter {
    var counter: Int = 0
        private set

    fun addWord(word: String) {
        counter += word.length
    }
}

fun main(args: Array<String>) {
    val lengthCounter = LengthCounter()
    lengthCounter.addWord("Hi!")
    println(lengthCounter.counter)
}

```



- 기본 가시성을 가지는 getter를 컴파일러가 생성하게 냅두고 setter의 가시성을 private으로 지정하여 외부 코드에서 단어 길이의 합을 마음대로 바꾸지 못하게 하였다.



## [컴파일러가 생성한 메소드 : 데이터 클래스와 클래스 위임]

### 4.3.1 모든 클래스가 정의해야 하는 메소드

자바와 마찬가지로 코틀린 클래스도 toString, equals, hashCode 등을 오버라이드할 수 있다.

> 알고 넘어가기!

자바는 == 를 원시 타입과 참조 타입을 비교할 때 사용한다. 

원시 타입의 경우 ==는 두 피연산자의 값이 같은지 비교하고, 참조 타입의 경우 == 는 두 피연산자의 주소가 같은지 비교한다. 따라서 자바에서 두 객체의 동등성을 알려면 equals()를 호출해야 한다. 

**코틀린에서는 == 연산자가 두 객체를 비교하는 기본적인 방법이다.** ==는 내부적으로 equals()를 호출해서 객체를 비교한다. 따라서 클래스가 equals()를 오버라이드하면 == 을 통해 안전하게 클래스의 인스턴스를 비교할 수 있다.

참조 비교를위해서는. === 연산자를 사용할 수 있다. 

=== 연산자는 자바에서 객체의 참조를 비교할 때 사용하는 ==와 같다.



### 4.3.2 data class

```kotlin
data class Client(val name: String, val postalCode: Int)
```

- class 앞에 data를 붙이면 자동으로 toString(), copy(), equals(), hashCode()를 포함한다.
- 이를 데이터 클래스라고 부르며 주로 데이터를 저장하는 역할을 한다.
- 주의할 점은 주 생성자 밖에 정의된 프로퍼티는 equals나 hashCode를 계산할 때 고려의 대상이 아니다.



**[copy() 메소드]**

- 데이터 클래스의 프로퍼티가 모두 val일 필요는 없다. var여도 된다. 하지만 데이터 클래스의 모든 프로퍼티를 읽기 전용으로 만들어 불변 클래스로 만들라고 권장한다.
- 불변의 장점 : 다중 스레드에서 동기화를 고려하지 않아도 됨.
- Copy() : 객체를 복사하면서 일부 프로퍼티를 바꿀 수 있게 해준다. 
- 객체를 메모리 상에서 직접 바꾸는 대신 복사본을 만드는 편이 더 낫다. 복사본은 원본과 다른 생명주기를 가지며, 복사를 하면서 일부 프로퍼티 값을 바꾸거나 복사본을 제거해도 프로그램에서 원본을 참조하는 다른 부분에 전혀 영향을 끼치지 않는다.



### 4.3.3 클래스 위임 : by

- 인터페이스를 구현할 때, by 키워드를 통해 그 인터페이스에 대한 구현을 다른 객체에 위임 중이라는 사실을 명시할 수 있다.

```kotlin
class DelegatingCollection<T>(innerList: Collection<T> = ArrayList<T>()) : Collection<T> by innerList {
  
}
```

- 컴파일러가 전달 메소드를 자동으로 생성하며, 자동 생성한 코드의 구현은 책에 실린 코드와 비슷하다.
- Collection의 구현을 innerList에게 위임한다.
- 메소드 중 일부의 동작을 변경하고 싶을 때는 메소드를 오버라이드 하면 컴파일러가 오버라이드한 메소드를 쓴다.



Ex)

```kotlin
import java.util.HashSet

class CountingSet<T>(
        val innerSet: MutableCollection<T> = HashSet<T>()
) : MutableCollection<T> by innerSet {

    var objectsAdded = 0

    override fun add(element: T): Boolean {
        objectsAdded++
        return innerSet.add(element)
    }

    override fun addAll(c: Collection<T>): Boolean {
        objectsAdded += c.size
        return innerSet.addAll(c)
    }
}

fun main(args: Array<String>) {
    val cset = CountingSet<Int>()
    cset.addAll(listOf(1, 1, 2))
    println("${cset.objectsAdded} objects were added, ${cset.size} remain")
}

```

- add, addAll을 오버라이드해서 count를 증가시키고 MutableCollection 인터페이스의 나머지 메소드는 내부 컨테이너인 innerSet에게 위임한다.

- CountingSet은 MutableCollection의 구현 방식에 대한 의존 관계가 생기지 않는다. CountingSet 코드는 위임 대상 내부 클래스인 MutableCollection의 API를 활용하므로 API를 변경하지 않는 한 CountingSet 코드가 계속 잘 작동할 것임을 확신할 수 있다. 

  -> **CountingSet의 생성자 파라미터에 존재하는 innerSet 프로퍼티에게 MutableCollection의 구현을 위임했기 때문에 CountingSet은 MutableCollection의 구현 방식에 대한 의존 관계가 없다. innerSet이 MutableCollection에 의존 관계를 가지고 있다.**



## object 키워드 : 클래스 선언과 인스턴스 생성

### 4.4.1 객체 선언 : 싱글톤

```kotlin
object Payroll{
  val allEmployees = arrayListOf<Person>()
  fun calculateSalary(){
    for(person in allEmployees){
      ...
    }
  }
}
```

- object를 통해 기본적으로 싱글톤 기능을 언어 레벨에서 제공한다.
- 객체 선언 = 클래스 선언 + 그 클래스에 속한 단일 인스턴스의 선언
- 생성자는 쓸 수 없다. 싱글톤 객체는 객체 선언문이 있는 위치에서 생성자 호출 없이 즉시 만들어지기 때문에 생성자 정의가 필요없다.
- object 선언도 클래스나 인터페이스 상속이 가능하다.
- 클래스 안에 object 선언도 가능하다. 이 객체도 인스턴스는 단 하나뿐이다. (바깥쪽 클래스의 인스턴스마다 중첩 객체 선언에 해당하는 인스턴스가 따로 하나씩 생기는 것이 아니다.)



### 4.4.2 동반 객체

- kotlin에서는 static 개념이 존재하지 않는다.
- 패키지 수준의 최상위 함수가 정적 메소드 역할을 대신할 수 있다.  객체 선언은 정적 필드를 대신할 수도 있다.
- 최상위 함수를 권장하지만, 클래스에 비공개 멤버를 포함하면 이 멤버에 접근할 수 없다. 
- 그래서 클래스의 인스턴스와 관계 없이 호출해야 하지만, 클래스 내부 정보에 접근해야 할 때 companion object를 사용하면 된다.
- companion object는 외부 클래스의 private한 멤버 접근이 가능하기 때문에 팩토리 메소드를 만들 때 유용하다.



```kotlin
class A {
    companion object {
        fun bar() {
            println("Companion object called")
        }
    }
}

fun main(args: Array<String>) {
    A.bar()
}

```

- 호출할 때, 클래스 이름으로 바로 호출이 가능하다.
- 이름을 따로 지정하지 않아도 되며, 사용 예를 보면 자바의 정적 메소드 호출이나 정적 필드 사용 구문과 같아진다.
- 즉, 자바의 static 함수 호출과 동일하다.



Ex)

**부 생성자가 여럿 있는 클래스**

```kotlin
class User{
  val nickname: String
  
  constructor(email: String){
    nickname = email.substringBefore('@')
  }
  
  constructor(accountId: Int){
    nickname = getFacebookName(accountId)
  }
}
```



**팩토리 메소드로 부 생성자 대신하기**

```kotlin
class User private constructor(val nickname: String){
  companion object{
    fun newSubscribingUser(email: String) = User(email.substringBefore('@'))
    
    fun newFacebookUser(accountId:Int) = User(getFacebookName(accountId))
  }
}

fun main(args: Array<String>){
  val user = User.newSubscribinUser("Victory@gmail.com")
  println(user.nickname)
  // Victory
}
```

- User는 private constructor를 가지기 때문에 외부에서 생성하지 못한다. 따라서 외부에서는 companion object로 제공되는 팩토리 메소드를 이용해서만 객체를 생성할 수 있도록 제한할 수 있다.



### 4.3.3 동반 객체를 일반 객체처럼 사용

동반 객체 = 클래스 안에 정의된 일반 객체

따라서 아래와 같은 작업이 가능하다.

- companion object 이름 명명 가능.
- companion object 내부에 확장 함수와 프로퍼티 정의
- 인터페이스 상속



```kotlin
class Person(val name: String){
  companion object Loader{
    fun fromJson(json: String) : Person {
      ...
    }
  }
}

fun main(args: Array<String>){
  Person.Loader.fromJson("{name: 'Lee'}")
  Person.fromJson("{name: 'Lee'}")
}
// 두 방법 모두 제대로 fromJson을 호출할 수 있다.
```



```kotlin
interface JSONFactory<T> {
    fun fromJSON(json: String): T
}

class Person(val name: String) {
    companion object : JSONFactory<Person> {
        override fun fromJSON(json: String): Person {
            return Person("Lee")
        }
    }
}

fun <T> loadFromJSON(factory: JSONFactory<T>): T? {
    return null
}

fun main() {
    loadFromJSON(Person)
}
```



위의 예제처럼 companion object가 특정 인터페이스를 구현할 수도 있고, **동반 객체가 구현한 JSONFactory 인터페이스를 넘길 때 Person 클래스(외부 클래스)의 이름을 사용한다.**



> 알고 넘어가기!

때로 자바에서 사용하기 위해 코틀린 클래스의 멤버를 정적인 멤버로 만들어야 할 필요가 있다.

그런 경우 `@JvmStatic ` 어노테이션을 코틀린 멤버에 붙이면 된다. 

정적 필드가 필요하다면 `@JvmField` 어노테이션을 최상위 프로퍼티나 객체에서 선언된 프로퍼티 앞에 붙인다. 

이는 10장에서 더 자세히 알아보자!



동반 객체를 이용해 외부에서 확장 함수를 정의할 수 있다고 했다. 아래 코드를 통해서 확인해보자.

```kotlin
class Person(val name: String){
  companion object{
    
  }
}

fun Person.Companion.fromJSON(json: String) : Person{
  // 확장 함수 정의.
}

fun main(){
  Person.fromJSON("json")
}
```

- 마치 동반 객체 안에 fromJSON을 정의한 것처럼 함수를 호출할 수 있다.
- 동반 객체에 대한 확장 함수를 정의하기 위해서는 원래 클래스에 동반 객체를 꼭 선언해야 한다.(비어있어도 괜찮다.)





### 4.4.4 무명 클래스

- 무명 객체를 정의할 때도 object 키워드를 쓴다. 
- 무명 객체는 자바의 무명 내부 클래스를 대신한다.



```kotlin
interface ClickListener{
  fun onClick()
}

val listener = object : ClickListener{
  override fun onClick(){
    println("Clicked Listener!!")
  }
}

fun main(){
  setClickAction(object: ClickListener{
    override fun onClick(){
      println("Clicked!!")
    }
  })
  
  setClickAction(listener)
}

fun setClickAction(clickListener: ClickListener){
  clickListener.onClick()
}
```



- object 선언과 달리 익명 클래스는 싱글톤이 아니다. 따라서 객체 식이 쓰일 때마다 새로운 인스턴스가 생성된다.
- 또한, 무명 객체 즉, 익명 클래스 안에서 함수에 정의된 로컬 변수를 사용할 수도 있다.
  - 자바와 조금 다른 점이다. 자바는 익명 클래스에서 접근 시 무조건 final이어야 한다.

###### 참조

- Kotlin IN Action / 출판사: 에이콘

###### 출처

- [Study Repository](https://github.com/TEAM-ASC/Kotlin/blob/master/Chapter4.%ED%81%B4%EB%9E%98%EC%8A%A4%2C%EA%B0%9D%EC%B2%B4%2C%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4/4%EC%9E%A5.md)
  - made by [WooVictory](https://github.com/WooVictory)


  