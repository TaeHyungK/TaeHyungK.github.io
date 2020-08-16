---
layout: post
title:  "[Kotlin] 코틀린 연산자 오버로딩과 기타 관례"
categories: [Android, Kotlin]
tags: [Kotlin]
---

> Kotlin IN ACTION(출판사: 에이콘) 책을 통해 Kotlin을 배워보자

## 연산자 오버로딩과 기타 관례

- 연산자 오버로딩
- 관례 : 여러 연산을 지원하기 위해 특별한 이름이 붙은 메소드
- 위임 프로퍼티



### OverViews

어떤 클래스 안에 pluse라는 이름의 특별한 메소드를 정의하면 그 클래스의 인스턴스에 대해 + 연산자를 사용할 수 있다. 이런 식으로 어떤 언어 기능과 미리 정해진 이름의 함수를 연결해주는 기법 -> `관례`



언어 기능을 타입에 의존하는 자바와 달리 코틀린은 함수 이름을 통한 관례에 의존한다.

이 관례를 채택한 이유는 기존 자바 클래스를 코틀린 언어에 적용하기 위함이다. 기존 자바 클래스가 구현하는 인터페이스는 이미 고정되어 있다. 그래서 코틀린 쪽에서 자바 클래스가 새로운 인터페이스를 구현하도록 할 수 없다.

반면, 확장 함수를 사용하면 기존 클래스에 새로운 메소드를 추가할 수 있다. 따라서 기존 자바 클래스에 대해 확장 함수를 구현하면서 관례에 따라 이름을 붙이면 기존 자바 코드를 바꾸지 않아도 새로운 기능을 부여할 수 있다.









이번 장에서는 Point라는 클래스를 예제로 사용할 것이다. 코드는 아래와 같다.

```kotlin
data class Point(val x: Int, val y: Int)
```





### 7.1 산술 연산자 오버로딩

자바는 원시 타입에 대해서만 산술 연산자를 정의할 수 있고, 추가적으로 String에 대해 + 연산자 사용이 가능하다.

하지만 다른 클래스에서도 유용한 경우가 있을 수 있다.

Ex) BigInteger 클래스의 add를 호출하기 보다는 +연산을 사용하는 편이 낫다. 

어떻게 하는지 알아보자.



#### 7.1.1 이항 산술 연산 오버로딩

```kotlin
data class Point(val x: Int, val y: Int) {
    operator fun plus(other: Point): Point {
        return Point(x + other.x, y + other.y)
    }
}

class PointTest {

    @Test
    fun `포인트 테스트`() {
        val p = Point(10, 20)
        val p2 = Point(30, 40)

        println(p + p2) // p.plus(p2) 로 컴파일된다.
    }
}
// Result
Point(x=40, y=60)
```

- 연산자를 오버로딩 하는 함수 앞에 operator 키워드가 있어야 한다. 이를 통해 이 함수가 관례를 따르는 함수임을 명확하게 알 수 있다.
- operator 없이 관례에서 사용하는 함수 이름을 쓰면 "operator modifier is required .. " 오류를 접하게 된다.
- 즉, plus 처럼 미리 정해진 이름의 함수를 operator 키워드를 통해 선언하면 +와 연결되어 + 호출로 연산을 수행할 수 있다.



연산자를 확장 함수로 정의할 수도 있다.

```kotlin
operator fun Point.plus(other: Point): Point{
  return Point(x+other.x, y+other.y)
}
```



- 코틀린에서는 프로그래머가 직접 연산자를 만들어 사용할 수 없고, 언어에서 미리 정해둔 연산자만 오버로딩할 수 있으며, 관례에 따르기 위해 클래스에서 정의해야 하는 이름이 연산자별로 정해져 있다.



| 식   | 함수 이름        |
| :--- | ---------------- |
| a*b  | times            |
| a/b  | div              |
| a%b  | mod(1.1부터 rem) |
| a+b  | plus             |
| a-b  | minus            |
|      |                  |

- 직접 정의한 함수를 통해 구현하더라도 연산자 우선순위는 언제나 표준 숫자 타입에 대한 연산자 우선순위와 같다.
- 연산자를 정의할 때, 두 피연산자는(연산자 함수의 두 파라미터) 같은 타입일 필요는 없다.
- Ex) 어떤 점을 비율에 따라 확대 및 축소하는 연산자를 정의하면 아래와 같다.



```kotlin
operator fun Point.times(scale: Double): Point {
    return Point((x * scale).toInt(), (y * scale).toInt())
}

@Test
fun `포인트 times 테스트`() {
  val p = Point(10, 20)
  println(p.times(1.5))
  println(p * 1.5)
}
// Result
Point(x=15, y=30)
Point(x=15, y=30)
```

- 코틀린 연산자는 자동으로 교환 법칙을 지원하지 않는다.
- 따라서 `p * 1.5` 가 된다고 해서 `1.5 *  p` 가 되지는 않는다. 역을 지원하기 위해서는 마찬가지로 역의 식에 대응하는 연산자 함수를 정의해야 한다.
- 또한, 연산자 함수의 반환 타입이 두 피연산자 중 하나와 일치하지 않아도 된다.
- 일반 함수와 마찬가지로 operator 함수도 오버로딩 가능하다. 따라서 이름은 같지만 파라미터 타입이 서로 다른 연산자 함수를 여럿 만들 수 있다. 
- 대신 operator 함수는 파라미터의 개수는 1개밖에 정의하지 못한다. 이항 연산이기 때문!



> 비트 연산자에 대해 특별한 연산자 함수를 사용하지 않는다.

코틀린은 표준 숫자 타입에 대해 비트 연산자를 정의하지 않는다. 따라서 커스텀 타입에서 비트 연산자를 정의할 수도 없다. 

대신, 중위 연산자 표기법을 지원하는 일반 함수를 사용해 비트 연산을 수행한다.



#### 7.1.2 복합 대입 연산자 오버로딩

```kotlin
var point = Point(1,2)
point +=Point(3,4)
println(point)
// Result
Point(x=4, y=6)
```

- +=, -= 등의 연산자를 복합 대입 연산자라 부른다.
- point +=Point(3,4) 식은 point = point + Point(3,4) 라고 쓴 식과 동일하다. 물론, 변경 가능한 경우에만 복합 대입 연산자를 사용할 수 있다.
- += 연산은 객체에 대한 참조를 다른 참조로 바꿔치기 한다.
  - point = point + Point(3,4)의 실행을 살펴보자. point의 plus는 새로운 객체를 반환한다. 
  - point + Point(3,4)는 두 점의 좌표 각각 더한 값을 좌표로 갖는 새로운 Point 객체를 반환한다. 그 후 대입이 이뤄지면 point 변수는 새로운 Point 객체를 가리키게 된다.
- 코틀린 표준 라이브러리는 MutableCollection에 대해 plusAssign을 정의하며, 아래와 같다. 이는 원래 객체의 내부 상태를 변경한다.

```kotlin
operator fun <T> MutableCollection<T>.plusAssign(element: T){
  this.add(element)
}
```



- +=를 plus와 plusAssign 양쪽으로 컴파일 할 수 있다. 어떤 클래스가 이 두 함수를 모두 정의하고 둘 다 +=에 사용 가능한 경우 컴파일러는 오류를 보여준다. 
- 일반 연산자를 이용해 해결하거나 var를 val로 바꿔서 plusAssign 적용을 불가능하게 할 수도 있다.
- 하지만, 일반적으로 새로운 클래스를 일관성 있게 설계하는 게 가장 좋다. plus와 plusAssign을 동시에 정의하는 것을 피해야 한다.
- 코틀린은 컬렉션에 대해 두 가지 접근 방법을 제공한다. 
- +, -는 항상 새로운 컬렉션을 반환한다.
- +=, -= 연산자는 항상 변경 가능한 컬렉션에 작용해 메모리에 있는 객체 상태를 변화시킨다. 
- 또한, 읽기 전용 컬렉션에서 +=, 0-는 변경을 적용한 복사본을 반환한다. 
- 이런 연산자의 피연산자로 개별 원소를 사용하거나 원소 타입이 일치하는 다른 컬렉션을 사용할 수 있다.

```kotlin
val list = arrayListOf(1,2)
list +=3 // 변경 가능한 컬렉션 list에 대해 +=을 통해 객체 상태를 변경.
val newList = list + listOf(4,5) // 두 리스트를 +로 합쳐 새로운 리스트를 반환.
println(list)
println(newList)

// Result
[1,2,3]
[1,2,3,4,5]
```



#### 7.1.3 단항 연산자 오버로딩

```kotlin
operator fun Point.unaryMinus(): Point {
    return Point(-x, -y)
}

@Test
fun `단항 연산자 테스트`() {
  val p = Point(10, 20)
  println(-p)
}

// Result
Point(x=-10, y=-20)
```

- 이항 연산자의 오버로딩과 마찬가지로 미리 정해진 이름의 함수를 멤버나 확장 함수로 선언하면서 operator를 표시하면 된다.
- 단항 연산자를 오버로딩하기 위해 사용하는 함수는 인자를 취하지 않는다. 



**[오버로딩할 수 있는 단항 산술 연산자]**

| 식       | 함수 이름  |
| -------- | ---------- |
| +a       | unaryPlus  |
| -a       | unaryMinus |
| !a       | not        |
| ++a, a++ | inc        |
| --a, a-- | dec        |
|          |            |

Ex)

```kotlin
operator fun BigDecimal.inc() = this + BigDecimal.ONE

@Test
fun `증가 연산자 테스트`(){
  var bd= BigDecimal.ZERO
  println(bd++) // 0
  println(bd) // 1
  println(++bd) // 2
}

// Result
0
1
2
```

- 후위 ++ 연산은 bd 값을 반환한 후, bd의 값을 증가시킨다.
- 전휘 ++ 연산은 그 반대로 동작한다. 
- 전위와 후위 연산을 처리하기 위해 별다른 처리를 해주지 않아도 제대로 동작한다.



### 7.2 비교 연산자 오버로딩

equals, compareTo를 호출해야 하는 자바와 달리 코틀린에서는 == 비교 연산자를 직접 사용함으로써 코드가 간결하며 이해하기 쉬운 장점이 있다. 



#### 7.2.1 동등성 연산자 : equals

- != 연산자도 equals로 컴파일된다. 이는 비교 결과를 뒤집은 값을 결과값으로 사용한다.
- ==와 !=는 내부에서 인자가 널인지 검사하므로 다른 연산과 달리 널이 될 수 있는 값에도 적용할 수 있다. 아래 코드를 보자.

```kotlin
a == b
// 위의 식은 아래처럼 컴파일 된다.
a?.equals(b) ?: (b == null)
```

- a가 널인지 판단해서 널이 아닌 경우에만 a.equals(b)를 호출한다. 
- 만약 a가 널이라면 b도 널인 경우에만 결과가 true가 된다.
- Point는 data class이므로 컴파일러가 자동으로 equals를 생성해준다. 구현한다면 아래와 같을 것이다.

```kotlin
class Point(val x: Int, val y: Int){
  override equals(obj: Any?): Boolean{
    if(this === obj) return true
    if(obj !is Point) return false
    
    return x == obj.x && y == obj.y
  }
}
```

- ===(식별자 비교 연산자)를 사용해 equals의 파라미터가 수신 객체와 같은지 확인한다.
- ===는 자바의 == 연산자와 같다. 따라서 ===는 자신의 두 핀연산자가 서로 같은 객체를 가리키는지(원시 타입인 경우 두 값이 같은지) 비교한다.
- ===를 사용해 자기 자신과의 비교를 최적화하는 경우가 많으며, ===는 오버로딩할 수 없다.
- Any의 equals에는 operator가 붙어있지만 그 메소드를 오버라이드하는 하위 클래스의 메소드 앞에는 operator를 붙이지 않아도 자동으로 상위 클래스의 operator 지정이 적용된다. 또한, Any에서 상속받은 equals가 확장 함수보다 우선순위가 높기 때문에 equals를 확장 함수로 정의할 수 없다.



#### 7.2.2 순서 연산자 : compareTo

- 자바에서 정렬이나 최댓값, 최솟값 등 값을 비교하는 알고리즘에 사용할 클래스는 Comparable 인터페이스를 구현한다.
- 코틀린도 똑같은 Comparable 인터페이스를 지원한다. 게다가 코틀린은 Comparable 인터페이스 안에 있는 compareTo 메소드를 호출하는 관례를 제공한다.
- 따라서 비교 연산자 (<, >, <=, >=)는 compareTo 호출로 컴파일 된다.
- 반환값은 Int이다. 다른 비교 연산자도 동일한 방식으로 동작한다. 

```kotlin
a >= b
// 위의 코드는 아래로 컴파일된다.
a.compareTo(b) >= 0

println("abc" < "bac")
// Result
true
```



### 7.3 컬렉션과 범위에 대해 쓸 수 있는 관례



#### 7.3.1 인덱스로 원소에 접근 : get, set

- 배열, 리스트, 맵에 접근할 때 []를 통해서 접근이 가능하다.
- []는 원소를 읽는 연산일 때는 get 연산자 메소드로 변환되고, 원소를 쓰는 연산은 set 연산자 메소드로 변환된다.

```kotlin
operator fun Point.get(index: Int): Int {
    return when (index) {
        0 -> x
        1 -> y
        else -> throw IndexOutOfBoundsException("Invalid coordinate $index")
    }
}
@Test
fun `get 테스트`(){
  val p = Point(10,20)
  println(p[1]) // p[1] -> p.get(1) 호출로 변환된다.
}
// Result
20
```

- get 연산자를 정의한다. 
- get 메소드의 파라미터로 Int가 아닌 타입도 사용할 수 있다. 맵의 경우는 키 타입이 될 수도 있다.
- 여러 파라미터를 사용하는 get을 정의할 수도 있다. 

```kotlin
operator fun get(rowIndex: Int, colIndex:Int){
  ...
}

// matrix[row, col]로 호출한다.
```

- 인덱스에 해당하는 컬렉션 원소를 쓰고 싶을 때는 set 함수를 정의하면 된다. 

```kotlin
data class MutablePoint(
    var x: Int,
    var y: Int
)

operator fun MutablePoint.set(index: Int, value: Int) {
    when (index) {
        0 -> x = value
        1 -> y = value
        else -> throw IndexOutOfBoundsException("Invalid coordinate $index")
    }
}
@Test
fun `set 테스트`(){
  val p = MutablePoint(10,20)
  p[0] = 30 // p[0] = 30 -> p.set(30)
  p[1] = 60 // p[1] = 60 -> p.set(60)
  println(p)
}
// Result
MutablePoint(x=30, y=60)
```



#### 7.3.2 in 관례

- 객체가 컬렉션에 들어있는지 검사한다.
- in 연산자와 대응하는 함수는 **contains**이다.

```kotlin
data class Rectangle(val upperLeft: Point, val lowerRight: Point)

operator fun Rectangle.contains(p: Point): Boolean {
    return p.x in upperLeft.x until lowerRight.x &&
            p.y in upperLeft.y until lowerRight.y
}

@Test
fun `in 테스트`() {
  val rect = Rectangle(Point(10, 20), Point(50, 50))
  println(Point(10, 30) in rect) // a in rect -> rect.contains(a)
  println(Point(10, 50) in rect)
}
// Result
true
false
```

- 범위를 만들고 x, y 좌표가 그 범위 안에 있는지 검사한다.
- until 함수를 사용해 열린 범위를 만든다.
- 열린 범위 : 끝 값을 포함하지 않는 범위를 말한다. 
  - Ex) 10..20 식을 사용해 일반적인 (닫힌) 범위를 만들면 10 이상 20 이하인 범위가 생긴다.(20을 포함.) 
  - Ex) 1o until 20으로 만드는 열린 범위는 10 이상 19이하인 범위며, 20은 범위 안에 포함되지 않는다.





#### 7.3.3 rangeTo 관례

- 1..10 : 1부터 10까지 모든 수가 들어있는 범위를 가리킨다.
- .. 연산자는 rangeTo 함수를 간략하게 표현하는 방법이다.
- 따라서 .. 는 rangeTo로 컴파일된다.
- 범위를 반환하며, 아무 클래스에나 정의할 수 있다.
- rangeTo 연산자는 다른 산술 연산자보다 우선순위가 낮다. 하지만 혼동을 피하기 위해 괄호로 감싸주는 것이 더 좋다.
- 또한, 범위 연산자는 우선 순위가 낮아서 범위의 메소드를 호출하려면 범위를 괄호로 둘러싸야 한다.

```kotlin
val n = 9
println(0 .. (n + 1))
0..10


// 아래 식은 컴파일할 수 없다.
0..n.forEach{}

// 아래 코드처럼 범위의 메소드를 호출하려면 범위를 괄호로 둘러싸면 된다.
(0..n).forEach{
  ...
}
```

- 추가적으로 코틀린에서는 모든 Comparable 객체에 대해 적용 가능한 rangeTo 함수를 제공한다. rangeTo는 ClosedRange 객체를 반환한다.

```kotlin
operator fun <T: Comparable<T>> T.rangeTo(that: T): ClosedRange<T>
```



#### 7.3.4 for 루프를 위한 iterator 관례

- 2장에서 살펴봤듯이 코틀린의 for 루프는 범위 검사와 똑같이 in 연산자를 사용한다.
- 하지만 의미는 다르다.
- 아래 코드는 list.iterator()를 호출해서 이터레이터를 얻은 다음, 자바와 마찬가지로 그 이터레이터에 대해 hasNext, next 호출을 반복하는 식으로 변환된다.

```kotlin
for (x in list){
  ...
}
```

- 이 또한 관례이므로 iterator 메소드를 확장 함수로 정의할 수 있다. 이런 성질로 인해 자바 문자열에 대한 for 루프가 가능하다.
- 코틀린은 String의 상위 클래스인 CharSequence에 대한 iterator 확장 함수를 제공한다. 따라서 아래와 같은 구문이 가능하다.

```kotlin
operator fun CharSequence.iterator(): CharIterator

for(c in "abc"){
  ...
}
```



- 클래스 안에 직접 iterator를 구현한 예이다.

```kotlin
operator fun ClosedRange<LocalDate>.iterator(): Iterator<LocalDate> =
        object : Iterator<LocalDate> {
            var current = start

            override fun hasNext() =
                current <= endInclusive

            override fun next() = current.apply {
                current = plusDays(1)
            }
        }

fun main(args: Array<String>) {
    val newYear = LocalDate.ofYearDay(2017, 1)
    val daysOff = newYear.minusDays(1)..newYear
    for (dayOff in daysOff) { println(dayOff) }
}
```

- 앞에서 rangeTo 함수가 ClosedRange 인스턴스를 반환한다. 코드에서 ClosedRange< LocaDate > 에 대한 확장 함수 Iterator를 정의했기 때문에 LocalDate의 범위 객체를 for 루프에서 사용할 수 있다.



### 7.4 구조 분해 선언과 component 함수

- 구조 분해를 사용하면 복합적인 값을 분해해서 여러 다른 변수를 한꺼번에 초기화할 수 있다.
- 구조 분해 선언은 일반 변수 선언과 비슷하다. 다만, = 좌변에 여러 변수를 괄호로 묶었다는 점이 다르다.

 ```kotlin
val p = Point(10,20)
val (x,y) = p
println(x)
println(y)
// Result
10
20
 ```

- 내부에서 구조 분해 선언은 관레를 사용한다. 구조 분해 선언의 각 변수를 초기화하기 위해 componentN이라는 함수를 호출한다.

```kotlin
val (a,b) = p
// 위의 구조 분해 선언은 아래의 componentN() 함수 호출로 변환된다.
val a = p.component1()
val b = p.component2()
```

- data class의 주 생성자에 있는 프로퍼티에 대해서는 컴파일러가 자동으로 componentN 함수를 만들어준다.

- 일반 클래스에서는 아래와 같이 구현한다.

```kotlin
class Point(val x: Int, val y: Int){
  operator fun component1() = x
  operator fun component2() = y
}
```

- 또한, 구조 분해 선언은 함수에서 여러 값을 반환할 때 유용하다.
- 여러 값을 반환해야 하는 함수가 있다면 반환해야 하는 모든 값이 들어갈 holder 역할의 데이터 클래스를 정의하고 함수의 반환 타입을 그 데이터 클래스로 바꾼다. 구조 분해 선언 구문을 사용해 이 함수가 반환하는 값을 쉽게 풀어 여러 변수에 넣을 수 있다.

```kotlin
data class NameComponents(val name: String,
                          val extension: String)

fun splitFilename(fullName: String): NameComponents {
    val result = fullName.split('.', limit = 2)
    return NameComponents(result[0], result[1])
}

fun main(args: Array<String>) {
    val (name, ext) = splitFilename("example.kt")
  	// 구조 분해 선언 구문을 사용해 데이터 클래스를 푼다.
    println(name)
    println(ext)
}
// Result
example
kt
```

- 코틀린은 맨 앞의 다섯 원소에 대한 componentN 함수를 제공한다. 따라서 컬렉션의 크기가 5보다 작아도 1~5까지접근이 가능하다. 하지만, IndexOutOfBoundsException이 발생한다. 
- 여섯 개 이상의 변수를 사용하는 구조 분해를 컬렉션에 대해 적용하면 컴파일 오류가 발생한다.



#### 7.4.1 구조 분해 선언과 루프

- 변수 선언이 들어갈 수 있는 장소라면 어디든 구조 분해 선언을 사용할 수 있다.
- 맵의 원소에 대해 이터레이션할 때, 구조 분해 선언이 유용하다.

```kotlin
fun print(map: Map<String, String)){
  for((key, value) in map){
    println("$key -> $value")
  }
}

val map = mapOf("Oracle" to "Java", "JetBrains" to "Kotlin")
print(map)
//Result
Oracle -> Java
JetBrains -> Kotlin
```

- 객체를 이터이션하는 관례, 구조 분해 선언 2가지 관례를 사용한다.
- 코틀린의 맵은 확장 함수로 iterator가 들어있다. 그 iterator는 맵 원소에 대한 이터레이터를 반환한다. 따라서 자바와 달리 코틀린에서는 맵을 직접 이터레이션할 수 있다.



### 7.5 프로퍼티 접근자 로직 재활용 : 위임 프로퍼티

- 위임이란 객체가 직접 작업을 수행하지 않고 다른 도우미 객체가 그 작업을 처리하게 맡기는 디자인 패턴.
- 작업을 처리하는 객체를 위임 객체(delegate)라고 한다.



#### 7.5.1 위임 프로퍼티

```kotlin
class Foo{
  val p : Type by Delegate()
  // by 키워드는 프로퍼티와 위임 객체를 연결한다.
}
```

- p 프로퍼티는 접근자 로직을 다른 객체에게 위임한다. 여기서는 Delegate 클래스의 인스턴스를 위임 객체로 사용한다.
- by 뒤에 있는 식을 계산해서 위임에 쓰일 객체를 얻는다.

```kotlin
class Foo{
  private val delegate = Delegate()
  val p: Type
  set(value: Type) = delegate.setValue(..., value)
  get() = delegate.getValue(...)
}
```

- 위의 코드처럼 컴파일러는 숨겨진 도우미 프로퍼티를 만들고 그 프로퍼티를 위임 객체의 인스턴스로 초기화한다. 
- p 프로퍼티는 바로 그 위임 객체에게 자신의 작업을 위임한다.
- Delegate 클래스를 단순화하면 다음과 같다.

```kotlin
class Delegate{
  operator fun getValue(...){
    ...
    // getter를 구현하는 로직을 담는다.
  }
  
  operator fun setValue(...){
    ...
    // setter를 구현하는 로직을 담는다.
  }
}
```

```kotlin
val foo = Foo()
val oldValue = foo.p // 1
foo.p = newValue // 2
```

- 1번과 같은 프로퍼티 호출은 내부에서 delegate.getValue()을 호출한다.
- 2번처럼 프로퍼티 값을 변경하는 문장은 내부에서 delegate.setValue(..., newValue)를 호출한다.



#### 7.5.2 by lazy()를 사용한 프로퍼티 초기화 지연

- 지연 초기화는 객체의 일부분을 초기화하지 않고 남겨뒀다가 실제로 그 부분의 값이 필요할 경우, 초기화할 때 흔히 쓰이는 패턴이다.
- 초기화 과정에 자원을 많이 사용하거나 객체를 사용할 때마다 꼭 초기화하지 않아도 되는 프로퍼티에 대해 지연 초기화 패턴을 사용할 수 있다.

```kotlin
class Email { /*...*/ }
fun loadEmails(person: Person): List<Email> {
    println("Load emails for ${person.name}")
    return listOf(/*...*/)
}

class Person(val name: String) {
    private var _emails: List<Email>? = null
  	// 데이터를 저장하고 emails의 위임 객체 역할을 하는 _emails 프로퍼티.

    val emails: List<Email>
       get() {
           if (_emails == null) {
               _emails = loadEmails(this) // 최초 접근 시 이메일을 가져온다.
           }
           return _emails!! // 저장해둔 데이터가 있으면 그 데이터를 반환한다.
       }
}

fun main(args: Array<String>) {
    val p = Person("Alice")
    p.emails // 최초로 emails를 읽을 때 단 한번만 이메일을 가져온다.
    p.emails
}
```

- 뒷받침하는 프로퍼티라는 기법을 사용한다. 
- _emails 프로퍼티는 값을 저장하고, emails 프로퍼티는 _emails 프로퍼티에 대한 읽기 연산을 제공한다. _emails는 Nullable 하고, emails는 널이 될 수 없는 타입이므로 프로퍼티 2개를 사용해야 한다. 이런 기법은 자주 사용된다.
- 이와 같은 방법은 성가시며, 스레드 안전하지 않아서 언제나 제대로 동작한다고 말할 수 없다.
- 대신 위임 프로퍼티를 사용해보자. 

```kotlin
class Person(val name: String){
  val emails by lazy { loadEmails(this) }
}

fun main(args: Array<String>) {
    val p = Person("Alice")
    p.emails
    p.emails
}
```

- lazy 함수는 코틀린 관례에 맞는 시그니처의 getValue() 메소드가 들어있는 객체를 반환한다. 따라서 lazy와 by 키워드와 함께 사용해 위임 프로퍼티를 만들 수 있다.
- lazy 함수의 인자는 값을 초기화할 때 호출할 람다다. 그리고 lazy 함수는 기본적으로 스레드 안전하다. 추가적으로 필요에 따라 동기화에 사용할 락을 lazy 함수에 전달할 수도 있고, 다중 스레드 환경에서 사용하지 않을 프로퍼티를 위해 lazy 함수가 동기화를 하지 못하게 막을 수도 있다.



#### 7.5.3  위임 프로퍼티 사용

- 위임 프로퍼티를 사용해서 변경을 통지해주는 부분의 코드를 작성해 처음부터 리팩토링 해나가는 과정을 보여주고 있습니다. 
- 설명하기 보다는 직접 읽어보는 것이 좋을 것 같아서 정리하지 않았으니 양해 바랍니다 😁



#### 7.5.4 위임 프로퍼티 컴파일 규칙

```kotlin
class C{
  var prop : Type by MyDelegate()
}

val c = C()
```

- 컴파일러는 MyDelegate 클래스의 인스턴스를 감춰진 프로퍼티에 저장하며 그 감춰진 프로퍼티는 <delegate> 라는 이름으로 부른다. 또한, 컴파일러는 프로퍼티를 표현하기 위해 KProperty 타입의 객체를 사용한다. 이 객체를 <property>라고 부른다.
- 컴파일러는 다음의 코드를 생성한다.

```kotlin
class C{
  private val <delegate> = MyDelegate()
  var prop : Type
  get() = <delegate>.getValue(this, <property>)
  set(value: Type) = <delegate>.setValue(this, <property>, value)
}
// this는 C 클래스를 가리킨다.
```

- 컴파일러는 모든 프로퍼티 접근자 안에 getValue, setValue 호출 코드를 생성해준다.
- 이 매커니즘은 상당히 단순하지만, 상당히 흥미로운 활용법이 많다고 한다.
- 프로퍼티 값이 저장될 장소를 바꿀 수도 있고(맵, 데이터베이스 테이블, 사용자 세션의 쿠키 등) 프로퍼티를 읽거나 쓸 때 벌어질 일을 변경할 수도 있다.(값 검증, 변경 통지 등) 이 모두를 간결한 코드로 달성할 수 있다.
- 아직까지 위임 프로퍼티를 사용해 본 경험은 없다. 그래서 이 내용이 와닿지 않지만, 저런 식으로 사용하면 확실히 간결하게 코드를 작성할 수 있고 여러 일을 수행하는 객체가 있다면 Delegate 패턴을 사용해 역할을 어느 정도 위임해 분리할 수 있지 않을까란 생각을 해봤다.
- 아래의 링크가 Delegate 패턴에 대해 설명하고 있으니 참고하면 좋을 것 같습니다.
  - [Delegate 패턴](https://the-earth.tistory.com/entry/Delegate-pattern-%EB%94%94%EC%9E%90%EC%9D%B8-%ED%8C%A8%ED%84%B4)
  - [[Java][정리] 위임(delegation)과 구현/포함(Composite) 개념](https://skkcha.tistory.com/32)



#### 7.5.5 프로퍼티 값을 맵에 저장

- 자신의 프로퍼티를 동적으로 정의할 수 있는 객체를 만들 때, 위임 프로퍼티를 활용하는 경우가 자주 있다. 그런 객체를 **확장 가능한 객체**(expando object)라고한다.

```kotlin
class Person {
  	// 추가 정보
    private val _attributes = hashMapOf<String, String>()

    fun setAttribute(attrName: String, value: String) {
        _attributes[attrName] = value
    }

  	// 필수 정보
    val name: String
        get() = _attributes["name"]!!
  	// 수동으로 맵에서 정보를 꺼낸다.
}

fun main(args: Array<String>) {
    val p = Person()
    val data = mapOf("name" to "Dmitry", "company" to "JetBrains")
    for ((attrName, value) in data)
       p.setAttribute(attrName, value)
  
    println(p.name)
}
// Result
Dmitry
```

- 위의 코드를 위임 프로퍼티를 활용하여 변경할 수 있다. by 키워드 뒤에 맵을 직접 넣으면 된다.



```kotlin
class Person {
    private val _attributes = hashMapOf<String, String>()

    fun setAttribute(attrName: String, value: String) {
        _attributes[attrName] = value
    }

    val name: String by _attributes
}

fun main(args: Array<String>) {
    val p = Person()
    val data = mapOf("name" to "Dmitry", "company" to "JetBrains")
    for ((attrName, value) in data)
       p.setAttribute(attrName, value)
    println(p.name)
}

```

- 이와 같은 코드가 동작하는 이유는 표준 라이브러리가 Map과 MutableMap 인터페이스에 대해 getValue, setValue 확장 함수를 제공하기 때문이다.
- getValue에서 맵에 프로퍼티 값을 저장할 때는 자동으로 프로퍼티 이름을 키로 활용한다.
- p.name -> _attributes.getValue(p, prop)라는 호출을 대신한다. 
- 이는 다시 _attributes.getValue(p, prop) -> _attributes[prop.name]을 통해 구현된다.
    

###### 출처

- Kotlin IN Action / 출판사: 에이콘
- [Study Repository](https://github.com/TEAM-ASC/Kotlin/blob/master/Chapter7.%EC%97%B0%EC%82%B0%EC%9E%90%20%EC%98%A4%EB%B2%84%EB%A1%9C%EB%94%A9/7%EC%9E%A5.md)
  - made by [WooVictory](https://github.com/WooVictory)
  