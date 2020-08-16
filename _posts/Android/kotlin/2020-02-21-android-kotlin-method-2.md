---
layout: post
title:  "[Kotlin] 함수 정의와 호출 2탄"
categories: [Android, Kotlin]
tags: [Kotlin]
---

> Kotlin IN ACTION(출판사: 에이콘) 책을 통해 Kotlin을 배워보자

### 함수 정의와 호출

[함수 정의와 호출 1탄](https://taehyungk.github.io/2020/02/18/android-kotlin-method-1/) 에 이어서 2탄을 배워보자.


#### 컬렉션 처리: 가변 길이 인자

인자의 개수가 달라질 수 있는 함수로 메소드를 호출할 때 원하는 개수만큼 값을 인자로 넘기면 컴파일러가 배열에 그 값들을 넣어주는 기능

```kotlin
val list = listOf(2, 3, 5, 7, 11)

// 위의 listOf 함수는?
fun listOf<T> (vararg values: T): List<T> {
    // ... 
}
```
  - 자바의 경우 varargs는 타입 뒤에 `...` 를 붙이는데 코틀린에서는 파라미터 앞에 `vararg` 변경자를 붙인다.

```kotlin
fun main (args: Array<String>) {
    val list = listOf("args: ", *args);
    println(list)
}
```
  - 이미 배열에 있는 원소를 가변 길이 인자로 넘길 때에 `*(스프레드 연산자)` 를 사용하여 배열의 각 원소가 인자로 전달되게 한다.








#### 컬렉션 처리: 값의 쌍 다루기 - 중위 호출 (infix call)

```kotlin
val map = mapOf(1 to "one", 7 to "seven", 53 to "fifty-three")

// 동일한 표현
1.to("one") // "to" 메소드를 일반적인 방식으로 호출
1 to "one" // "to" 메소드를 중위 호출 방식으로 호출

infix fun Any.to(other: Any) = Pair(this, other)

val (number, name) = 1 to "one"
```
  - `infix` 변경자를 메소드 선언 앞에 추가하여 해당 메소드를 중위 호출 사용을 허용할 수 있다.
  - Pair는 코틀린 표준 라이브러리 클래스로, 두 원소로 이뤄진 순서쌍을 표현한다.

#### 컬렉션 처리: 값의 쌍 다루기 - 구조 분해 선언 (destructuring declaration)

```kotlin
val (number, name) = 1 to "one"

for ((index, element) in collection.withIndex()) {
    println("$index: $element")
}
```
  - `val (number, name) = 1 to "one"` 구문에서 1은 key로 number 변수에 대입이 되고, "one"은 value로 name 변수에 대입되게 되는데, 이러한 기능을 **구조 분해 선언**이라고 한다.

#### 문자열 나누기

```kotlin
// 문자열 구분자
"12.345-6.A".split(".", "-")
"12.345-6.A".split('.', '-')

// 정규식을 통한 문자열 나누기
"12.345-6.A".split("\\.|-".toRegex()) // 정규식임을 명시적으로 표시
```
  - 코틀린 정규식 문법과 자바는 똑같다. (~~다만, 정규식 처리하는 API는 표준 자바 라이브러리 API와 비슷하지만 좀 더 코틀린답게 변경됐다고 한다.~~)

#### 3중 따옴표로 묶은 문자열

String 확장 함수를 사용해 경로 파싱해보기
```kotlin
fun parsePath(path: String) {
    val directory = path.substringBeforeLast("/") // directory - 전체 경로의 첫 글자부터 마지막 슬래시 바로 전까지
    val fullName = path.substringAfterLast("/")
    val fileName = fullName.substringBeforeLast(".") // 파일명
    val extension = fullName.substringAfterLast(".") // 확장자
}

// >>> parsePath("Users/yole/kotlin-book/chapter.adoc"
// 결과: Dir: /Users/yole/kotlin-book
//      fileName: chapter
//      extension: adoc
```

경로 파싱에 정규식 사용하기
```kotlin
fun parsePath(path: String) {
    val regex = """(.+)/(.+)\.(.+)""".toRegex()
    val matchResult = regex.matchEntire(path)
    if (matchResult != null) {
        val (directory, filename, extension) = matchResult.destructured
        println("Dir: $directory, name: $filename, extension: $extension")
    }
}
```
  - `"""(.+)/(.+)\.(.+)"""`: 3중 따옴표 문자열
  - 3중 따옴표 문자열에서는 역슬래시(\)를 포함한 어떤 문자도 이스케이프할 필요가 없다.
  - default 로 정규식 엔진은 각 패턴이 가능한 한 가장 긴 부분 문자열과 매치하려고 시도한다. 


여러 줄의 3중 따옴표 문자열

```kotlin
val kotlinLogo = """| /
                    .| //
                    .|/\"""
```
  - 들여쓰기나 줄 바꿈을 포함한 **모든 문자**가 들어가게 된다.


#### 로컬 함수

```kotlin
class User(val id: i=Int, val name: String, val address: String) {
    fun User.validateBeforeSave() {
        if (validate(value: String, filedName: String) {
            if (value.isEmpty() {
                throw IllegaArgumentException("Can't save uiser $id: empty $fileName"); // User와 프로퍼티를 직접 사용할 수 있다.
            })
        })
    }
    validate(name, "Name")
    validatte(address, "Address")
}
```


###### 출처

- Kotlin IN Action / 출판사: 에이콘
  