---
layout: post
title:  "[Kotlin] 애노테이션과 리플렉션"
categories: [Android, Kotlin]
tags: [Kotlin]
---

> Kotlin IN ACTION(출판사: 에이콘) 책을 통해 Kotlin을 배워보자

## 10장

### Overviews

- 어노테이션 적용과 정의
- 리플렉션을 사용해 실행 시점에 객체 내부 관찰
- 코틀린 실전 프로젝트 예제

> 들어가기 전에

애노테이션을 사용하면 라이브러리가 요구하는 의미를 클래스에게 부여할 수 있고, 리플렉션을 사용하면 실행 시점에 컴파일러 내부 구조를 분석할 수 있다.

### 10.1 어노테이션 선언과 적용

코틀린과 자바에서의 개념은 같다.

메타 데이터를 선언에 추가하면 어노테이션을 처리하는 도구가 컴파일 시점이나 실행 시점에 적절한 처리를 해준다.








##### 10.1.1 어노테이션 적용

- 적용하려는 대상 앞에 붙이면 된다.
- @과 어노테이션 이름으로 이루어진다.
- 함수나 클래스 등 여러 다른 코드에 붙일 수 있다.

```kotlin
class MyTest{
  @Test fun testTrue(){
    Assert.assertTrue(true)
  }
}
```

- @Test 를 사용해 JUnit 프레임워크에게 이 메소드를 테스트로 호출하라고 지시한다.





- 코틀린에서는 @Deprecated와 replaceWith 파라미터를 통해 옛 버전을 대신할 수 있는 패턴을 제시할 수 있다. 
- 따라서 API 사용자는 패턴을 보고 지원이 종료될 API 기능을 더 쉽게 새 버전으로 포팅할 수 있다.

```kotlin
@Deprecated("Use removeAt(index) insted.", ReplaceWith("removeAt(index)"))
fun remove(index: Int){ ... }
```



- 어노테이션의 인자로는 원시 타입의 값, 문자열, enum 클래스 참조, 다른 어노테이션 클래스 등등 들어갈 수 있다. 
- 어노테이션 인자를 지정하는 문법은 자바와 약간 다르다.
  - 클래스를 어노테이션 인자로 지정할 때는 클래스 이름 뒤에 ::class를 넣는다.
  - 다른 어노테이션을 인자로 지정할 때는 인자로 들어가는 어노테이션의 이름 앞에 @를 붙이지 않는다. 위의 Deprecated와 ReplaceWith의 예가 그러하다.
  - 배열을 인자로 지정하려면 @RequestMapping(path=arrayOf("/foo", "/bar")) 처럼 arrayOf 함수를 사용한다. 



어노테이션 인자를 컴파일 시점에 알 수 있어야 한다. 따라서 임의의 프로퍼티를 인자로 지정할 수 없다.

프로퍼티를 어노테이션 인자로 사용하기 위해서는 앞에 **const** 를 붙여야 한다. 컴파일러는 const가 붙은 프로퍼티를 컴파일 시점에 상수로 취급한다.

```kotlin
const val TIME_OUT = 100L
@Test (timeout = TIME_OUT) fun testMethod(){
  ...
}
```



const가 붙은 프로피티는 파일의 맨 위나 object 안에 선언해야 하며, 원시 타입이나 String 값으로 초기화해야 한다. 일반 프로퍼티를 어노테이션 인자로 사용하려 시도하면 "Only const val can be used in constant expressions"라는 오류를 만나게 된다. (const val만 상수 식에 사용할 수 있음)



##### 10.2 어노테이션 대상

- 코틀린에서의 선언을 컴파일한 결과가 여러 자바 선언과 대응하는 경우가 있다.
- 이때, 코틀린 선언과 대응하는 여러 자바 선언에 각각 어노테이션을 붙여야 할 때가 있다.
- Ex) 코틀린 프로퍼티는 기본적으로 자바 필드와 게터 메소드 선언과 대응한다. 프로퍼티가 변경 가능하면 세터에 대응하는 자바 세터 메소드와 세터 파라미터가 추가된다. 이외에도 생성자와 관련해서도 존재하며, 어노테이션을 붙일 때, 이런 요소 중 어떤 요소에 어노테이션을 붙일지 표시할 필요가 있다.
- `사용 지점 대상` 선언으로 어노테이션 붙일 요소를 정할 수 있다. @ 기호와 어노테이션 이름 사이에 붙으며, 어노테이션 이름과는 콜론(:)으로 분리된다.

```kotlin
@get:Rule
```

- get : 사용 지점 대상
- Rule : 어노테이션 이름



Ex)

규칙을 지정하려면 공개 필드나 메소드 앞에 @Rule을 붙여야 한다. 하지만 코틀린 테스트 클래스의 folder라는 프로퍼티 앞에 @Rule을 붙이면 "The @Rule 'folder' must be public"(@Rule을 지정한 folder는 public이어야 함)라는 JUnit 예외가 발생한다. @Rule은 필드에 적용되었지만, 코틀린의 필드는 기본적으로 비공개이기 때문에 이런 예외가 발생한다. (디컴파일 시 필드는 private이 되지만, getter, setter는 public이기 때문에 프로퍼티 입장으로 볼 때는 public 이라고 할 수 있다. -> 4장 참고!)



따라서 @Rule 어노테이션을 정확한 대상에 적용하려면 다음과 같이 사용해야 한다.

```kotlin
class HasTempFolder{
  @get:Rule
  val folder = TemporaryFolder()
  
  @Test
  fun testUsingTempFolder(){
    val createdFile = folder.newFile("myFile.txt")
    val createdFolder = folder.newFolder("subfolder")
    ...
  }
}
```

- 프로퍼티가 아니라 게터에 어노테이션을 붙인다.
- 코틀린으로 어노테이션을 선언하면 프로퍼티에 직접 적용할 수 있는 어노테이션을 만들 수 잇다.
- 사용 지점 대상을 지정할 때, 지원하는 대상 목록은 다음과 같다.
  - **property** : 프로퍼티 전체. 자바에서 선언된 어노테이션에는 이 사용 지점 대상을 사용할 수 없다.
  - **field** : 프로퍼티에 의해 생성되는 (뒷받침하는) 필드
  - **get** : 프로퍼티 게터
  - **set** : 프로퍼티 세터
  - **receiver** : 확장 함수나 프로퍼티의 수신 객체 파라미터
  - **param** : 생성자 파라미터
  - **setparam** : 세터 파라미터
  - **delegate** : 위임 프로퍼티의 위임 인스턴스를 담아둔 필드
  - **file** : 파일 안에 선언된 최상위 함수와 프로퍼티를 담아두는 클래스



> 자바 API를 어노테이션으로 제어하기

코틀린은 코틀린으로 선언한 내용을 자바 바이트 코드로 컴파일하는 방법과 코틀린 선언을 자바에 노출하는 방법을 제어하기 위한 어노테이션을 많이 제공한다. 다음은 코틀린 선언을 자바에 노출시키는 방법을 변경할 수 있다.

- @JvmName : 코틀린 선언이 만들어내는 자바 필드나 메소드 이름을 변경한다.
- @JvmStatic : 메소드, 객체 선언, 동반 객체에 적용하면 그 요소가 자바 정적 메소드로 노출된다.
- @JvmOverloads : 디폴트 파라미터 값이 있는 함수에 대해 컴파일러가 자동으로 오버로딩한 함수를 생성해준다.
- @JvmField : 프로퍼티에 사용하면 게터나 세터가 없는 공개된 자바 필드로 프로퍼티를 노출시킨다.



##### 10.1.3 어노테이션을 활용한 JSON 직렬화 제어

- **직렬화** : 객체를 저장장치에 저장하거나 네트워크를 통해 전송하기 위해 텍스트나 이진 형식으로 변환하는 것.
- **역직렬화** : 텍스트나 이진 형식으로 저장된 데이터로부터 원래의 객체를 만들어낸다.
- 직렬화에 자주 쓰이는 형식은 JSON이며, JSON을 변환할 때 자주 쓰이는 라이브러리는 잭슨과 지슨이 있다.



이 교재에서는 **[jkid](https://github.com/yole/jkid)** 라는 라이브러리를 사용한다. 소스 코드의 양이 그리 많지 않아서 쉽게 파악할 수 있다고 한다. 성능도 나쁘지 않다고 하니, 한번씩 보는 것을 추천한다.

- serialize : 이 함수를 사용해 객체를 직렬화 하여 JSON 표현이 담긴 문자열을 얻는다.
- deserialize : 이 함수를 사용해  JSON 표현을 객체로 만들 수 있으며, 원하는 객체의 타입을 지정해야 한다.
- jkid 라이브러리에서는 어노테이션을 활용해 객체를 직렬화하거나 역직렬화하는 방법을 제어할 수 있다. 
- 객체 -> JSON으로 직렬화할 때, jkid는 기본적으로 모든 프로퍼티를 직렬화하며, 프로퍼티 이름을 키로 사용한다. 
- 어노테이션을 활용해 이 동작을 바꿔보자.
  - @JsonExclude : 어노테이션을 사용하면 직렬화나 역직렬화시 그 프로퍼티를 무시할 수 있다.
  - @JsonName : 어노테이션을 사용하면 프로퍼티를 표현하는 키/값 쌍의 키로 프로퍼티 이름 대신 어노테이션이 지정한 이름을 쓰게 할 수 있다.

```kotlin
data class Person{
  @JsonName("alias") val firstName: String,
  @JsonExclude val age: Int?=null
}
```

- firstName 프로퍼티를 JSON으로 저장할 때 사용하는 키를 alias로 한다.
- age 프로퍼티는 직렬화나 역직렬화시 사용되지 않는다. **이처럼 직렬화 대상에서 제외할 프로퍼티에는 반드시 디폴트 값을 지정해야만 한다. 그렇지 않으면 역직렬화시 Person의 인스턴스를 새로 만들 수 없다.**





##### 10.1.4 어노테이션 선언

- jkid 라이브러리에 선언된 코드를 보며 하나씩 살펴보자. 먼저, 어노테이션 선언부터 확인해보자.
- 어노테이션의 선언은 다음과 같다.

```kotlin
annotation class JsonExclude
```

- 어노테이션 클래스는 오직 선언이나 식과 관련있는 메타 데이터의 구조를 정의하기 때문에 내부에 아무 코드도 들어있을 수 없다.
- 이러한 이유로 컴파일러는 어노테이션 클래스에서 본문을 정의하지 못하게 막는다.
- 파라미터가 있는 어노테이션을 정의하려면 어노테이션 클래스의 주 생성자에 파라미터를 선언해야 한다.
- 일반 클래스의 주 생성자 선언과 같지만, **모든 파라미터 앞에 val를 붙여줘야 한다.**

```kotlin
annotation class JsonName(val name: String)
```

- 자바 어노테이션과 비교하면 다음과 같다.

```java
public @interface JsonName{
  String value();
}
```

- 자바에서는 value라는 메소드를 사용하면 이는 특별하다.
- 어떤 어노테이션을 적용할 때, value를 제외한 모든 어트리뷰트에는 이름을 명시해야 한다.  반면, 코틀린의 어노테이션 적용 문법은 일반적인 생성자 호출과 같다. 따라서 인자의 이름을 명시하기 위해 이름 붙은 인자 구문을 사용할 수도 있고, 이름을 생략할 수도 있다. 여기서는 name이 JsonName 생성자의 첫 번째 인자이므로 @JsonName(name = "first_name")은 @JsonName("first_name")과 같다.
- 자바에서 선언한 어노테이션을 코틀린의 구성 요소에 적용할 때는 value를 제외한 모든 인자에 대해 이름 붙은 인자 구문을 사용해야만 한다. 코틀린도 자바 어노테이션에 정의된 value를 특별하게 취급한다.
- **음,, 이게 무슨 말일까? 함께 고민이 필요하다.**
- 아마 리플렉션에서도 사용하고 코틀린에서의 디폴트 파라미터 혹은 인자 이름을 생략해서 호출하는게 자바에서 안되니까 그런 것 뜻하는 의미인 것 같기도 하다. 생각해보기...



##### 10.1.5 메타어노테이션 : 어노테이션을 처리하는 방법 제어

- 코틀린 어노테이션 클래스에도 어노테이션을 붙일 수 있다.
- 어노테이션 클래스에 적용할 수 있는 어노테이션을 **메타 어노테이션**이라 한다.
- 표준 라이브러리에는 몇 가지 메타 어노테이션이 있으며, 그런 메타 어노테이션들은 컴파일러가 어노테이션을 처리하는 방법을 제어한다.
- 가장 흔히 쓰이는 메타 어노테이션은 **@Target**이다. 앞서 살펴본 jkid의 JsonExclude, JsonName 어노테이션에도 적용 가능 대상을 지정하기 위해 **@Target**을 사용한다.

```kotlin
@Target(AnnotationTarget.PROPERTY)
annotation class JsonExclude
```

- @Target 메타 어노테이션은 어노테이션을 적용할 수 있는 요소의 유형을 지정한다. 어노테이션 클래스에 대해 구체적인 @Target을 지정하지 않으면 모든 선언에 적용할 수 있는 어노테이션이 된다. jkid 라이브러리는 프로퍼티 어노테이션만을 사용하므로 어노테이션 클래스에 @Target을 꼭 지정해야 한다.
- 어노테이션이 붙을 수 있는 대상이 정의된 enum은 **AnnotationTarget**이다. 이 안에는 클래스, 파일, 프로퍼티, 프로퍼티 접근자, 타입, 식 등에 대한 enum 정의가 들어있다.
- 둘 이상의 대상을 한꺼번에 선언할 수도 잇다.
- 메타 어노테이션을 직접 만들어야 한다면 ANNOTATION_CLASS를 대상으로 지정하면 된다.

```kotlin
@Target(AnnotationTarget.ANNOTATION_CLASS)
annotation class BindingAnnotation

@BindingAnnotation
annotation class MyBinding
```

- 대상을 **PROPERTY**로 지정한 어노테이션은 자바에서 사용할 수 없다. 자바에서 사용하려면 AnnotationTarget.FIELD를 두번째 대상으로 추가해줘야 한다. 그렇게 하면 어노테이션을 코틀린 프로퍼티와 자바 필드에 적용할 수 있다.



> @Retention 이란?

- @Retention은 정의 중인 어노테이션 클래스를 소스 수준에서만 유지할지, .class 파일에 저장할지, 실행 시점에 리플렉션을 사용해 접근할 수 있게 할지를 지정하는 메타 어노테이션이다.
- 자바 컴파일러는 기본적으로 어노테이션을 .class 파일에 저장하지만, 런타임에는 사용할 수 없다. 하지만 대부분의 어노테이션은 런타임에도 사용할 수 있어야 하므로 코틀린에서는 기본적으로 어노테이션의 @Retention을 RUNTIME으로 지정한다.



##### 10.1.6 어노테이션 파라미터로 클래스 사용

- 클래스 참조를 파라미터로 하는 어노테이션 클래스를 선언하면 어떤 클래스를 선언 메타 데이터로 참조할 수 있다.
- Ex) jkid의 @DeserializeInterface -> 인터페이스 타입인 프로퍼티에 대한 역직렬화를 제어할 때 쓰는 어노테이션이다.
- 인터페이스의 인스턴스를 직접 만들 수 없다. 따라서 역직렬화 시 어떤 클래스를 사용해 인터페이스를 구현할 지를 지정할 수 있어야 한다.

```kotlin
interface Company{
  val name: String
}

data class CompanyImpl(override val name: String) : Company

data class Person{
  val name: String,
  @DeserializeInterface(CompanyImpl::class) val company: Company
}
```

- 직렬화된 Person 인터페이스를 역직렬화하는 과정에서 company 프로퍼티를 표현하는 JSON을 읽으면 jkid는 그 프로퍼티 값에 해당하는 JSON을 역직렬화하면서 CompanyImpl의 인스턴스를 만들어서 Person 인스턴스의 company 프로퍼티에 설정한다.
- 이렇게 역직렬화를 사용할 클래스를 지정하기 위해 @DeserializeInterface 어노테이션의 인자로 CompanyImpl::class를 넘긴다.



- @DeserializeInterface(CompanyImpl::class)처럼 클래스 참조를 인자로 받는 어노테이션 선언은 다음과 같다.

```kotlin
annotation class DeserializeInterface(val targetClass: KClass<out Any>)
```

- KClass는 자바의 java.lang.Class 타입과 같은 역할을 하는 코틀린 타입.
- 코틀린 클래스에 대한 참조를 저장할 때, KClass 타입을 사용한다.
- KClass의 타입 파라미터는 이 KClass의 인스턴스가 가리키는 코틀린 타입을 지정한다. CompanyImpl::class의 타입은 KClass< CompanyImpl >이며, 이 타입은 위에서 살펴본 DeserializeInterface의 파라미터 타입인 KClass< out Any >의 하위 타입이다.
- KClass의 타입 파라미터를 쓸 때, out 없이 KClass< Any > 라고 쓰면 DeserializeInterface에게 CompanyImpl::class를 인자로 넘길 수 없고, 오직 Any::class만 넘길 수 있다.
- 반면, out이 존재하면 모든 코틀린 타입 T에 대해 KClass< T >가 KClass< out Any >의 하위 타입이 된다. 이는 9장 제네릭에서 살펴본 **공변성** 개념이다. 따라서 DeserializeInterface의 인자로 Any 뿐 아니라 Any를 확장하는 모든 클래스에 대한 참조를 전달할 수 있다.



##### 10.1.7 어노테이션 파라미터로 제네릭 클래스 받기

- jkid는 기본적으로 원시 타입이 아닌 프로퍼티를 중첩된 객체로 직렬화한다. 이런 기본 동작을 변경하고 싶으면 값을 직렬화하는 로직을 직접 제공하면 된다.
- @CustomSerializer 어노테이션은 커스텀 직렬화 클래스에 대한 참조를 인자로 받는다. 이 직렬화 클래스는 ValueSerializer 인터페이스를 구현해야만 한다.

```kotlin
interface ValueSerializer<T>{
  fun toJsonValue(value: T) : Any?
  fun fromJsonValue(jsonValue: Any?) : T
}
```

- ValueSerializer< Date >를 구현하는 DateSerializer를 사용해 Person 클래스에 적용해보자.

```kotlin
data class Person{
  val name: String,
  @CustomSerializer(DateSerializer::class) val birthDate: Date
}
```

- ValueSerializer 클래스는 제네릭이므로 타입 파라미터가 있다. 따라서 ValueSerializer 타입을 참조하려면 항상 타입 인자를 제공해야 한다. 하지만, 이 어노테이션이 어떤 타입에 대해 쓰일지 알 수 없으므로 **스타 프로젝션** (*)을 사용할 수 있다.

```kotlin
annotation class CustomSerializer(val serializerClass: KClass<out ValueSerializer<*>>)
```

- **이를 한 단계씩 이해해보자.**
- CustomSerializer 는 ValueSerializer를 구현하는 클래스만 인자로 받아야 한다.
-  out(공변성 개념) : ValueSerializer::class 뿐 아니라 ValueSerializer를 구현하는 모든 클래스를 받아들인다.
- ValueSerializer : DateSerializer::class는 올바른 인자로 받아들이지만, Date::class는 거부한다. DateSerializer가 ValueSerializer를 구현하기 때문이다.
- "*" : ValueSerializer를 사용해 어떤 타입의 값이든 직렬화 할 수 있게 허용한다.



간단한 정리

- 클래스를 인자로 받아야 하는 경우.

```kotlin
KClass<out 허용할 클래스 이름>
```

- 제네릭 클래스를 인자로 받아야 하는 경우. (스타 프로젝션 사용)

```kotlin
KClass<out 허용할 클래스 이름<*>>
```





### 10.2 리플렉션

- 리플렉션은 실행 시점에 동적으로 객체의 프로퍼티와 메소드에 접근할 수 있게 해주는 방법이다.
- 보통은 객체의 메소드나 프로퍼티에 접근할 때는 프로그램 소스 코드 안에 구체적인 선언이 있는 메소드나 프로퍼티 이름을 사용하며, 컴파일러는 그런 이름이 실제로 가리키는 선언을 컴파일 시점에 (**정적으로**) 찾아내서 해당하는 선언이 실제 존재함을 보장한다.
- 하지만 타입과 관계 없이 객체를 다뤄야 하거나 객체가 제공하는 메소드나 프로퍼티 이름을 오직 실행 시점에만 알 수 있는 경우가 있다. 그 예가 JSON 라이브러리다. 직렬화 라이브러리는 어떤 객체든 JSON으로 변환할 수 있어야 하고, 실행 시점이 되기 전까지는 라이브러리가 직렬화할 프로퍼티나 클래스에 대한 정보를 알 수 없다. 이런 경우, 리플렉션을 사용해야 한다.
- 코틀린에서 리플렉션을 사용할 수 있는 API
  1. java.lang.reflect : 자바가 제공하는 api이다. 코틀린 클래스가 자바 바이트 코드로 컴파일 되므로 코틀린도 완벽히 지원한다.
  2. kotlin.reflect : 자바에는 없는 프로퍼티나 널이 될 수 있는 타입과 같은 코틀린 고유 개념에 대한 리플렉션을 제공한다. 하지만 이는 자바 리플렉션 API를 완전히 대체할 수 있는 복잡한 기능을 제공하지는 않는다. 또한, 코틀린 리플렉션 API를 사용해도 다른 JVM 언어에서 생성한 바이트 코드를 충분히 다룰 수 있다.



> Note!

안드로이드와 같이 런타임 라이브러리 크기가 문제가 되는 플랫폼을 위해 코틀린 리플렉션 API는 별도의 .jar 파일에 담겨 제공되며, 새 프로젝트를 생성할 때, 리플렉션 패키지 .jar 파일에 대한 의존 관계가 자동으로 추가되는 일은 없다. 따라서 사용이 필요하면 직접 추가해줘야 한다.





##### 10.2.1 코틀린 리플렉션 API

- KClass : java.lang.Class에 해당하며, 이를 사용하면 클래스 안에 있는 모든 선언을 열거하고 각 선언에 접근하거나 클래스의 상위 클래스를 얻는 등의 작업이 가능하다.

```kotlin
class Person(val name: String, val age: Int)

val person = Person("Lee", 27)
val kClass = person.javaClass.kotlin // KClass<Person>의 인스턴스를 반환한다.
println(kClass.simpleName)
>> Person

kClass.memberProperties.forEach{ println(it.name) }
>> age
>> name
```

- 실행 시점에 객체의 클래스를 얻기 위해서 먼저 객체의 javaClass 프로퍼티를 사용해 객체의 자바 클래스를 얻어야 한다. 이후에는 .kotlin 확장 프로퍼티를 통해 자바에서 코틀린 리플렉션 API로 옮겨올 수 있다.
- memberProperties를 통해 클래스와 모든 조상 클래스 내부에 정의된 비확장 프로퍼티를 모두 가져온다.
- KClass의 선언을 보면 클래스의 내부를 살펴볼 때, 사용할 수 있는 다양한 메소드들이 존재한다. 이는 교재 혹은 문서를 통해 확인하기 바란다.
- KClass에 대해 사용할 수 있는 다양한 기능은 실제로는 kotlin-reflect 라이브러리를 통해 제공하는 확장 함수이다. 확장 함수를 사용하려면 `import kotlin.reflect.full.*` 로 확장 함수 선언을 임포트 해야 한다.
- 클래스의 모든 멤버의 목록이 KCallable 인스턴스의 컬렉션이다.
- **KCallable** : 함수와 프로퍼티를 아우르는 공통 상위 인터페이스다. call 메소드가 있으며 이를 사용해 함수나 프로퍼티의 getter를 호출할 수 있다.



```kotlin
interface KCallable<out R>{
  fun call(vararg args: Any?) : R
  ...
}
```



```kotlin
fun main(args: Array<String>) {
    val kFunction = ::foo
    kFunction.call(42)
}

fun foo(x:Int) = println(x)
```

- ::는 멤버 참조로 이미 선언된 함수를 값으로 사용할 때, 쓸 수 있다. ::foo 식의 값 타입이 리플렉션 API에 있는 KFunction 클래스의 인스턴스가 된다. 이 함수가 참조가 가리키는 함수를 호출하려면 KCallable.call 메소드를 호출해야 한다.
- call에 넘긴 인자 개수와 정의된 파라미터의 개수가 동일해야 한다.
- 여기서 함수를 호출하기 위해 더 구체적인 메소드를 사용할 수도 있다. ::foo의 타입 KFunction1< Int, Unit > 에는 파라미터와 반환 값 타입 정보가 들어있다. 1은 이 함수의 파라미터가 1개라는 의미다. 
- KFunction1 인터페이스를 통해 함수를 호출하려면 invoke() 메소드를 호출하면 된다. invoke는 정해진 개수의 인자만을 받아 들이며, 인자 타입은 KFunction1 제네릭 인터페이스의 첫 번째 타입 파라미터와 같다.
- 게다가 KFunction을 직접 호출할 수도 있다. -> 이는 11장에서 다룬다.

```kotlin
fun main(args: Array<String>) {
    val kFunction2 = ::sum
    println(kFunction2.invoke(42, 8) + kFunction2(8, 42))
}

fun sum(x: Int, y: Int) = x + y
// 100
```

- invoke 메소드 역시 호출할 때, 인자 개수나 타입이 맞아야 한다. 그렇지 않으면 컴파일 오류가 발생한다. 
- 따라서 KFunction의 인자 타입과 반환 타입을 모두 안다면 invoke 메소드를 호출하는 게 낫다. call 메소드는 모든 타입의 함수에 적용할 수 있는 일반적인 메소드이지만, 타입 안정성을 보장해주지 않기 때문이다.

> 언제 그리고 어떻게 KFunctionN 인터페이스가 정의되는가?

KFunction1과 같은 타입은 파라미터 개수가 다른 여러 함수를 포함한다. 각 KFunctionN 타입은 KFunction을 확장하며, N과 파라미터 개수가 같은 invoke를 추가로 포함한다. 

ex) KFunction2<P1, P2, R>에는 operator fun invoke(p1: P1, p2: P2) : R 선언이 들어 있다.



이런 함수 타입들은 컴파일러가 생성한 합성 타입이다. 따라서 kotlin.reflect 패키지에서는 찾을 수 없다. 코틀린에서는 컴파일러가 생성한 합성 타입을 사용하기 때문에 원하는 수만큼 많은 파라미터를 갖는 함수에 대한 인터페이스를 사용할 수 있다. 합성 타입을 사용하기 때문에 코틀린은 kotlin-runtime.jar의 크기를 줄일 수 있고, 함수 파라미터 개수에 대한 인위적인 제약을 피할 수 있다.



- get 메소드에 접근하기 위해서는 프로퍼티가 선언된 방법에 따라 올바른 인터페이스를 사용해야 한다.
- 최상위 프로퍼티는 KProperty0 인터페이스의 인스턴스로 표현되며, KProperty0 안에는 인자가 없는 get 메소드가 있다.

```kotlin
var counter = 0

fun main(args: Array<String>) {
    val kProperty = ::counter
    kProperty.setter.call(21)
    println(kProperty.get())
}
```

- 위의 코드는 최상위 프로퍼티에 대해서 리플렉션 기능을 통해 세터를 호출하면서 21을 인자로 넘긴다.
- get을 호출해 프로퍼티 값을 가져온다.



- 반면, 멤버 프로퍼티는 KProperty1 인스턴스로 표현된다. 그 안에는 인자가 1개인 get 메소드가 들어있다. 멤버 프로퍼티는 어떤 객체에 속해 있는 프로퍼티이므로 멤버 프로퍼티의 값을 가져오려면 get 메소드에게 프로퍼티를 얻고자 하는 객체 인스턴스를 넘겨야 한다.

```kotlin
fun main(args: Array<String>) {
    val person = Person("Lee", 27)
    val memberProperty = Person::age
    println(memberProperty.get(person))
  	// 27
}

class Person(val name: String, val age: Int)
```

- memberProperty에 프로퍼티 참조를 저장한 뒤, memberProperty.get(person) 호출을 통해서 person 인스턴스의 프로퍼티 값을 가져온다. 
- **memberProperty가 Person 클래스의 age 프로퍼티를 참조한다면 memberProperty.get(person)은 동적으로 person.age를 가져온다.**
- 좀 더 자세히 살펴보자.
- memberProeprty는 KProperty<Person, Int> 타입으로 첫 번째 타입 파라미터는 수신 객체 타입, 두 번째 타입 파라미터는 프로퍼티 타입을 표현한다. 따라서 수신객체를 넘길 때는 KProperty1의 타입 파라미터와 일치하는 타입의 객체만을 넘길 수 있다. memberProperty.get("Lee")와 같은 호출은 컴파일 되지 않는다.
- 최상위 수준이나 클래스 안에 정의된 프로퍼티만 리플렉션으로 접근할 수 있고, 함수 안의 지역 변수에는 접근할 수 없다. 함수 안에서 지역 변수 x에 대해 ::x로 그 변수에 대한 참조를 얻으려 시도하면 (변수에 대한 참조는 아직 지원하지 않음)이라는 오류 메시지를 볼 수 있다.





간단하게 정리!

- KClass : 클래스와 객체를 표현할 때, 쓰임
- KProperty : 모든 프로퍼티를 표현할 수 있고, 그 하위 클래스인 KMutableProperty는 var로 정의한 변경 가능한 프로퍼티를 표현한다.
- 다른 인터페이스는 교재를 참고하기 바란다.



##### 10.2.2 리플렉션을 사용한 객체 직렬화 구현

[Code]

```kotlin
private fun StringBuilder.serializeObject(obj: Any) {
    val kClass = obj.javaClass.kotlin // 객체의 KClas를 얻는다.
    val properties = kClass.memberProperties // 클래스의 모든 프로퍼티를 얻는다.

    properties.joinToString(this, prefix = "{", postfix = "}") { prop ->
        serializeString(prop.name) // 프로퍼티 이름을 얻는다.
        append(": ")
        serializePropertyValue(prop.get(obj)) // 프로퍼티 값을 얻는다.
    }
}
```

- 위 함수는 클래스의 각 프로퍼티를 차례로 직렬화한다. 결과 JSON은 "{prop1: value1, prop2: value2}" 같은 형태다.
- joinToStringBuilder : 프로퍼티를 콤마로 구분해준다.
- **아래 두 함수는 jkid 라이브러리의 함수이다.**
- serializeString : JSON 명세에 따라 특수 문자를 이스케이프 해준다.
- serializePropertyValue : 어떤 값이 원시 타입, 문자열, 컬렉션, 중첩된 객체 중 어떤 것인지 판단하고 그에 따라 값을 적절히 직렬화한다.
- 위의 코드에서는 어떤 객체의 클래스에 정의된 모든 프로퍼티를 열거하기 때문에 정확히 각 프로퍼티가 어떤 타입인지 알 수 없다. 따라서 prop 변수의 타입은 KProperty1<Any, *>이며, prop.get(obj) 메소드 호출은 Any 타입의 값을 반환한다. 이는 Any 클래스의 어떤 프로퍼티가 들어오든 처리함을 뜻한다.
- 이 경우 수신 객체 타입을 컴파일 시점에 검사할 방법이 없다. 하지만 이 코드에서는 어떤 프로퍼티의 get에 넘기는 객체가 바로 그 프로퍼티를 얻어온 객체이기 때문에 항상 프로퍼티 값이 제대로 반환된다.



##### 10.2.3 어노테이션을 활용한 직렬화 제어



1) JsonExclude

- 지금까지 다루던 부분은 jkid 라이브러리가 어떻게 직렬화를 구현하는지에 대한 내용이 주를 이루고 있다. 
- 따라서 내용 대부분을 적기보다는 책을 참고하는 걸 추천하며, 핵심만 정리하겠다.
- @JsonExclude : 어떤 프로퍼티를 직렬화에서 제외하고 싶을 때 사용한다.
- serializeObject 함수를 수정해 이 어노테이션이 붙은 프로퍼티를 제외하는 것이 목표이다.
- KAnnotatedElement 인터페이스에 annotations 프로퍼티가 있다. 이는 소스 코드 상에서 해당 요소에 적용된(@Retention을 RUNTIME으로 지정한) 모든 어노테이션 인스턴스의 컬렉션이다. KProperty는 KAnnotatedElement를 확장하므로 property.annotations를 통해 프로퍼티의 모든 어노테이션을 얻을 수 있다.
- 하지만 모든 어노테이션이 필요한 것이 아닌, 특정 어노테이션을 찾기만 하면 되기 때문에 findAnnotation라는 함수를 사용하면 적절하다. 이는 인자로 전달받은 타입에 해당하는 어노테이션이 있으면 그 어노테이션을 반환한다. 

```kotlin
import kotlin.reflect.full.findAnnotation
import kotlin.reflect.full.memberProperties

fun main(args: Array<String>){
    val obj = Any()
    val kClass =  obj.javaClass.kotlin
    val properties = kClass.memberProperties.filter { it.findAnnotation<JsonExclude>()  == null }
}
```

- 위의 코드는 JsonExclude 어노테이션을 사용하지 않는 프로퍼티에 대해서만 작업을 수행한다. (어노테이션이 없기 때문에 null 반환.)



2) JsonName

- 이 경우에는 어노테이션의 존재 여부뿐 아니라 어노테이션에 전달한 인자도 알아야 한다.
- findAnnotation()은 여기서도 도움이 된다.

```kotlin
import kotlin.reflect.full.findAnnotation
import kotlin.reflect.full.memberProperties
annotation class JsonName(val name: String)

fun main(args: Array<String>) {
    val people = People("Lee", 27)
    val kClass = people.javaClass.kotlin
    val prop = kClass.memberProperties
    prop.forEach { prop ->
        val jsonName = prop.findAnnotation<JsonName>()
        val propName = jsonName?.name ?: prop.name
        println(propName)
    }
}

data class People(
        @JsonName("alias") val firstName: String,
        val age: Int
)
// age
// firstName
```

- JsonNam 어노테이션이 있으면 그 인스턴스를 얻고, 어노테이션에서 name 인자를 찾고 그런 인자가 없으면 prop.name을 사용한다.
- 그러나 어노테이션을 지정했는데, 왜 firstName, age 프로퍼티 모두 널이 떨어질까? age는 null이 되어 프로퍼티의 이름이 출력되는 것이 맞는데, firstName도 마찬가지로 null이 되어 그대로 출력된다... <같이 고민해보기!>



위에서 본 코드를 조금 수정해보자.

```kotlin
private fun StringBuilder.serializeObject(obj: Any) {
    obj.javaClass.kotlin.memberProperties
            .filter { it.findAnnotation<JsonExclude>() == null }
            .joinToString(this, prefix = "{", postfix = "}") {
                serializeProperty(it, obj)
            }
}
```

- 위 코드를 통해서 JsonExclude로 어노테이션한 프로퍼티를 제외시킨다. 또한, 프로퍼티 직렬화와 관련된 로직을 serializeProperty 확장 함수로 분리해 호출한다.

```kotlin
/ 프로퍼티 직렬화.
private fun StringBuilder.serializeProperty(prop: KProperty1<Any, *>, obj: Any) {
    val jsonNameAnn = prop.findAnnotation<JsonName>()
    val propName = jsonNameAnn?.name ?: prop.name
    serializeString(propName)
    append(": ")
    serializePropertyValue(prop.get(obj))
}
```

- JsonName에 따라 프로퍼티 이름을 처리한다. 



```kotlin
// 커스텀 직렬화기의 toJsonValue 함수를 호출해서 프로퍼티 값을 JSON 형식으로 변환한다. 다만, 어떤 프로퍼티에 커스텀 직렬화기가
// 지정돼 있지 않다면 프로퍼티 값을 그냥 사용한다.
private fun StringBuilder.serializeProperty(prop: KProperty1<Any, *>, obj: Any) {
    val name = prop.findAnnotation<JsonName>()?.name ?: prop.name
    serializeString(name)
    append(": ")
    val value = prop.get(obj)
    val jsonValue = prop.getSerializer()?.toJsonValue(value) ?: value
    // 프로퍼티에 대해 정의된 커스텀 직렬화기가 있으면 그 커스텀 직렬화기를 사용한다.
    // 커스텀 직렬화기가 없으면 일반적인 방법을 따라 프로퍼티를 직렬화한다.

    serializePropertyValue(jsonValue)
}
```





##### 10.2.4 JSON 파싱과 객체 역직렬화

- 이 부분도 거의 코드로 설명이 되기 때문에 핵심만 짚고 나머지는 책을 통해서 확인하는 걸 추천한다.
- JSON 문자열 입력을 파싱하고, 리플렉션을 사용해 객체의 내부에 접근해서 새로운 객체와 프로퍼티를 생성하기 때문에 JSON을 역직렬화하는 것은 직렬화보다 더 어렵다.
- jkid의 JSON 역직렬화기는 흔히 쓰는 방법을 따라 3 단계로 구현되어 있다.
  1. 어휘 분석기로 렉서(lexer)
  2. 문법 분석기로 파서(parser)
  3. 파싱한 결과로 객체를 생성하는 역직렬화 컴포넌트

- 어휘 분석기 : 여러 문자로 이뤄진 입력 문자열을 토큰의 리스트로 변환한다.
- 토큰
  1. 문자 토큰 : 문자를 표현하며 JSON 문법에서 중요한 의미가 있다.(콤마, 콜론, 중괄호, 각괄호가 문자 토큰에 해당한다.)
  2. 값 토큰 : 문자열, 수, 불리언 값, null 상수를 말한다. 
  3. 왼쪽 중괄호({), 문자열 값("Hi"), 정수 값(42)는 모두 서로 다른 토큰이다.
- 파서 : 토큰의 리스트를 구조화된 표현으로 변환한다. jkid에서 파서는 JSON의 상위 구조를 이해하고 토큰을 JSON에서 지원하는 의미 단위로 변환하는 일을 한다. 그런 의미 단위로는 키, 값 쌍과 배열이 있다.



이후에 나오는 글들은 코드에 대한 설명이 너무 강해서 직접 읽어보는 걸 추천한다.



### 10.3 요약

- 문법은 자바와 거의 유사하다. 그리고 코틀린에서는 자바보다 더 넓은 대상에 어노테이션을 적용할 수 있다. 그런 대상으로는 파일과 식을 들 수 있다.
- 어노테이션 인자로 원시 타입 값, 문자열, 이넘, 클래스 참조, 다른 어노테이션 클래스의 인스턴스, 그리고 지금까지 말한 여러 유형의 값으로 이뤄진 배열을 사용할 수 있다.
- @get:Rule을 사용해 어노테이션의 사용 대상을 명시하면 한 코틀린 선언이 여러 가지 바이트 코드 요소를 만들어내는 경우, 정확히 어떤 부분에 어노테이션을 적용할 지 지정할 수 있다.
- 어노테이션 클래스를 정의할 때는 본문이 없고, 주 생성자의 모든 파라미터를 val 프로퍼티로 표시한 코틀린 클래스를 사용한다.
- 메타어노테이션을 사용해 대상, 어노테이션 유지 방식 등 여러 어노테이션 특성을 지정할 수 있다.
- 리플렉션을 통해 실행 시점에 객체의 메소드와 프로퍼티를 열거하고 접근할 수 있다. 리플렉션 API에는 클래스, 함수 등 여러 종류의 선언을 표현하는 인터페이스가 들어있다.(KClass, KFunction)
- 클래스를 컴파일 시점에 알고 있다면 KClass 인스턴스를 얻기 위해 ClassName::class를 사용한다. 하지만, 실행 시점에 obj 변수에 담긴 객체로부터 KClass 인스턴스를 얻기 위해서는 obj.javaClass.kotlin을 사용한다.
- KFunction, KProperty 인터페이스는 모두 KCallable을 확장한다. KCallable은 제네릭 call 메소드를 제공한다.
- KCallable.callBy 메소드를 사용하면 메소드를 호출하면서 디폴트 파라미터 값을 사용할 수 있다.
- KFunction0, KFunction1 등의 인터페이스는 모두 파라미터 수가 다른 함수를 표현하며, invoke 메소드를 사용해 함수를 호출할 수 있다.


###### 출처

- Kotlin IN Action / 출판사: 에이콘
  