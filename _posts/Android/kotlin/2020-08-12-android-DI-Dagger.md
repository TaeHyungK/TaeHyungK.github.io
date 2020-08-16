---
layout: post
title:  "[DI] 의존성 주입? Dagger?"
categories: [Android, DI]
tags: [Kotlin]
---

* content
{:toc}

### DI의 기본 개념 이해하기

#### DI와 IoC

DI는 Dependency Injection의 준말로 의존성 주입을 뜻합니다.
의존성 주입이란 외부에서 의존 객체를 생성하여 넘겨주는 것을 의미합니다.

예를들어 A 클래스가 B 클래스를 의존할 때 B Object를 A가 직접 생성하지 않고 **외부에서 생성하여 넘겨주면 의존성 주입했다**고 합니다.






![DI_inject](/img/DI_inject.jpeg)

그림에서 왼쪽은 A 클래스에서 B,C를 생성하는 일반적인 의존 형태이고 오른쪽은 **외부에서 의존 객체를 생성**하고 **주입**하는 형태입니다.

DI를 위해서는 객체를 생성하고 넘겨주는 외부의 무언가가 필요합니다. 이것이 DI Framework가 하는 일입니다. 외부에서 넘겨주는 무언가를 Spring에서는 컨테이너, Dagger에서는 `Component`와 `Module` 이라고 부릅니다.

DI는 이렇게 의존성이 있는 객체의 제어를 외부 Framework로 올리면서 **IoC(Inversion of Control, 제어의 역전) 개념**을 구현합니다.

<br>

![DI_IoC_container](/img/DI_IoC_container.jpeg)

그림에서 외부 컨테이너가 객체를 생성, 주입합니다. 제어가 거꾸로 가는 개념을 IoC라고 하고, DI는 IoC를 구현하는 방법 중 하나입니다.

<br><br>

#### DI는 왜 필요한가?

```kotlin
class A(private val b: B) {
    fun something() {
        b.something()
    }
}
```

앞서 이미지로 봤던 것처럼 보통은 위와 같은 코드를 통해 생성자로 주입하는 방식을 많이 사용합니다.

1. 의존성 파라미터를 생성자에 작성하지 않아도 되므로 **보일러 플레이트 코드**를 줄일 수 있습니다.
2. Interface에 구현체를 쉽게 교체하면서 상황에 따라 적절한 행동을 정의할 수 있습니다. 특히 테스팅 시에 Mock 객체와 실제 객체를 바꿔가며 테스트할 때 용이합니다.
3. 의존 받는 객체(A)는 의존 객체(B)의 구현을 알 필요가 없습니다. 이는 프로그램의 설계가 훨씬 유연해진 다는 것을 의미합니다.

DI가 무엇이고 DI가 왜 필요한지에 대해 알아보았습니다.

안드로이드에서 DI 구현을 돕는 라이브러리인 Dagger에 대해 알아보겠습니다.

<br><br>

### Dagger 기본 개념 이해하기

Dagger에는 아래와 같이 5가지의 필수 개념이 있습니다.
1. Inject
2. Component
3. Subcomponent
4. Module
5. Scope

이 개념들에 대해 하나씩 알아보겠습니다.

#### Inject

의존성 주입을 요청합니다. Inject 어노테이션으로 주입을 요청하면 연결된 Component가 Module로 부터 객체를 생성하여 넘겨줍니다.

<br><br>

#### Component

연결된 Module을 이용하여 의존성 객체를 생성하고, Inject로 요청받은 인스턴스에 생성한 객체를 주입합니다. 의존성을 요청받고 주입하는 Dagger의 주된 역할을 수행합니다.

<br><br>

#### Subcomponent

Component는 계층 관계를 만들 수 있습니다. Subcomponent는 Inner Class 방식의 하위 계층 Component 입니다. Sub의 Sub도 가능합니다.
Subcomponent는 Dagger의 중요한 컨셉인 그래프를 형성합니다. Inject로 주입을 요청받으면 Subcomponent에서 먼저 의존성을 검색하고, 없으면 부모로 올라가면서 검색합니다.

<br><br>

#### Module

Component에 연결되어 의존성 객체를 생성합니다. 생성 후 Scope에 따라 관리도 합니다.

<br><br>

#### Scope

생성된 객체의 Lifecycle 범위입니다. 안드로이드에서는 주로 Activity, Fragment 등 화면의 생명주기에 맞추어 사용합니다. Module에서 Scope를 보고 객체를 관리합니다.

위 5가지 개념을 따라 Dagger가 의존성을 주입하는 플로우는 다음과 같습니다.

![DI_scope_flow_1](/img/DI_scope_flow_1.png)

![DI_Inject_In_Component](/img/DI_Inject_In_Component.png)

Inject로 의존성을 요청하면 Subcomponent에서부터 Module을 검색하고, Scope에 따라 객체를 가져와 주입합니다.

Dagger에 대해 알아보았으니, 다음에는 Hilt에 대해서 알아보겠습니다.


<br><br><br>

###### 참고 사이트
* [VictoryWoo: [Android] DI에 대해서](https://woovictory.github.io/2019/07/08/DI/)
* [Seungmin — maryang: DI 기본개념부터 사용법까지, Dagger2 시작하기](https://medium.com/@maryangmin/di-%EA%B8%B0%EB%B3%B8%EA%B0%9C%EB%85%90%EB%B6%80%ED%84%B0-%EC%82%AC%EC%9A%A9%EB%B2%95%EA%B9%8C%EC%A7%80-dagger2-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0-3332bb93b4b9)
