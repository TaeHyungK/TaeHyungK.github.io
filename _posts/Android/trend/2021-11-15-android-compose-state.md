---
layout: post
title:  "[Compose] State 란?"
categories: [Android, Compose]
tags: [Jetpack]
---

최근 GDG dev 2021을 통해서 Compose Codelab 공부를 하고 있는데, Compose의 중요한 개념 중 하나인 State에 대해 간단하게 정리하고자 한다.

[Using state in Jetpack Compose](https://www.youtube.com/watch?v=XXKmlKolcPk)





## Compose State

#### State 란?

* 네트워크 연결을 할 수 없을 때 보여주는 스낵바
* 블로그 포스트와 관련된 댓글 UI들
* 클릭할 때 재생되는 ripple 애니메이션
* 사용자가 이미지 위에 그릴 수 있는 스티커
* 등..

위와 같이 **시간이 지남에 따라 변경될 수 있는 모든 것**을 의미한다.

<br><br><br>

#### Compose는 내부적으로 단방향 통신으로 구현되어 있다.

* Activity/Fragment -> ViewModel: event 를 전달만 한다.
* ViewModel -> Activity/Fragment: state 를 흘러 내려주기만 한다.

통상 사용하는 `ViewModel` 을 생각하면 이해가 쉽다.

<br><br><br>

#### Compose 에서 프로그래밍 할 때 알아야 할 사항

* Compose 함수는 순서와 관계없이 실행 가능하다.
* Compose는 동시에 실행이 가능하다.
    * **위 2가지 이유로 인해 각 컴포넌트들이 독립적이여야 한다.**
* Recomposition은 변경된 구성 요소만 실행하게 된다.
* 같은 데이터라면 동일한 결과가 나온다.
* Compose 함수는 UI에서 매우 자주 실행될 수 있다.
    * 비용이 많이 드는 작업은 Compose에서 하지 말고 외부의 다른 스레드에서 해주어야 함.

<br><br><br>

#### Remember

* Stateful: `Remember` 를 사용해서 내부 State 생성
    * 호출하는 쪽에서 State를 관리할 필요가 없다.
    * 그래서 재사용성이 떨어지며 테스트하기 어렵다.
* Stateless: `State` 를 갖지 않는 Composable
    * State를 호출해야 하는 곳(Composable 밖에서)에서 제어
    * 그래서 재사용성이 높고 테스트하기가 쉬워진다.

재사용성이나 테스트를 위해 Stateful -> Stateless로 만드는 것이 좋은데, Stateful을 Stateless로 만드는 것을 **State hoisting** 이라고 한다.

<br><br><br>

#### State hoisting

하나의 state가 여러 컴포넌트에 영향을 줄 때 해당 state를 상위에 두어 같은 level에 위치한 컴포넌트들이 하나의 state를 바라보도록 하는 것을 말한다.

State Hoisting을 할 때 어디로 위치시켜야 할 지 알아내는데 도움이 되는 세가지 규칙

1. State는 State를 사용하는 모든 Composable의 최소 공통 상위 항목으로 보낸다.
2. State는 최소한 수정 할 수 있는 최고 수준으로 이동해야 한다.
3. 동일한 이벤트에 대한 응답으로 두 State가 변경되면 함께 이동해주면 된다.

<br><br><br>

##### 글을 마치며

앞에 2단계는 꽤나 간단(?)했는데 State 라는 개념이 들어가면서 갑자기 어려워진 느낌이다 ㅎ.. 
재미도 있으면서 실제로 state를 사용해본 적이 없어서 사용감이 없는 것이 걱정이다.. 😢<br>

###### 참고 사이트
* [Jetpack Compose 가이드](https://developer.android.com/jetpack/compose/documentation)
* [Using state in Jetpack Compose : Codelab](https://developer.android.com/codelabs/jetpack-compose-state?authuser=4&continue=https%3A%2F%2Fdeveloper.android.com%2Fcourses%2Fpathways%2Fcompose%3Fhl%3Den%26authuser%3D4%23codelab-https%3A%2F%2Fdeveloper.android.com%2Fcodelabs%2Fjetpack-compose-state&hl=en#1)
* [Using state in Jetpack Compose : Youtube](https://www.youtube.com/watch?v=XXKmlKolcPk)