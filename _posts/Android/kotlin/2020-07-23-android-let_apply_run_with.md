---
layout: post
title:  "[Android] let, apply, run, with 함수"
categories: Android
tags: Kotlin
author: TaeHyungK
---

* content
{:toc}

### let, apply, run, with 함수

코틀린에서 제공하는 여러 함수 중 비슷하면서도 조금 다른.. 헷갈리는 함수 4가지에 대해 알아보자.






##### let
* `let()`은 함수를 호출하는 객체를 이어지는 블록의 인자로 넘기고, 블록의 결과값을 반환합니다.
* 사용 예)
  * 함수를 호출한 객체를 인자로 받으므로 이를 사용하여 다른 메소드를 실행하거나 연산을 수행해야 하는 경우 사용할 수 있습니다.

##### apply
* `apply()`는 함수를 호출하는 객체를 이어지는 블록의 리시버로 전달하고 객체 자체를 반환합니다.

  > **리시버?**
  > 바로 이어지는 블록 내에서 메소드 및 속성에 바로 접근할 수 있도록 할 객체를 의미합니다.
* 사용 예)
  * 특정 객체를 생성하면서 함께 호출해야 하는 초기화 코드가 있는 경우 사용할 수 있습니다.

##### run
* 객체 없이 `run()` 함수를 사용하면 인자 없는 익명 함수처럼 사용할 수 있습니다. 이어지는 블럭 내에서 처리할 작업들을 넣어줄 수 있고, 일반 함수처럼 값을 반환하지 않거나(Unit) 특정 값을 반환할 수도 있습니다.
* 객체에서 `run()` 함수를 호출할 경우, 호출하는 객체를 이어지는 블록의 리시버로 전달하고, 블록의 결과값을 반환합니다.
* `run()` 함수는 `with()` 함수를 좀 더 편리하게 사용하기 위해 `let()` 함수와 `with()` 함수를 합쳐놓은 형태라고 생각하면 됩니다.
* 또한, 안전한 호출(Safe calls)를 사용할 수 있어 특정 컴포넌트(?)가 null이 아닐 경우에만 블록 내 명령들을 실행하도록 할 수 있습니다.
* 사용 예)
  * 객체에서 이 함수를 호출하는 경우 객체를 리시버로 전달받으므로, 특정 객체의 메소드나 필드를 **연속적으로** 호출하거나 값을 할당할 때 사용합니다.
* `apply()`와 유사하지만, `apply()`는 **새로운 객체를 생성**함과 동시에 연속된 작업이 필요할 때 사용하고 `run()`은 **이미 생성된 객체**에 연속된 작업이 필요할 때 사용한다는 차이가 있습니다.

##### with
* `with()` 함수는 인자로 받는 객체를 이어지는 블록의 리시버로 전달하며, 블록의 결과를 반환합니다.
* `with()` 함수는 사실상 `run()` 함수와 기능이 거의 동일하며 리시버로 전달할 객체가 어디에 위치하는지만 다릅니다.
* `run()`이 안전한 호출(Safe calls)를 지원하는데 반해 `with()` 함수는 이를 자체적으로 지원하지 않습니다.

###### 참고 사이트
* [코틀린의 유용한 함수들 - let, apply, run, with](https://www.androidhuman.com/lecture/kotlin/2016/07/06/kotlin_let_apply_run_with/)