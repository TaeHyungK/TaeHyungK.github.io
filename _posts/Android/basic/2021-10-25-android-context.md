---
layout: post
title:  "[Android] Context"
categories: [Android, CS]
tags: [CS]
---

안드로이드에서는 Context 라는 개념이 굉장히 중요하다.

그래서 이번에 Context가 무엇인지 간략하게 적어보고자 한다.<br>
(사실 개인 Trello에 정리는 해놨는데 잠시 잊었던 안드로이드 관련 지식들을 다시 한번 상기하고자 한다 !)

## Context

어플리케이션과 관련된 정보에 접근하거나 안드로이드 시스템에서 제공하는 API를 호출할 때 사용되는 추상 클래스이다.

* context.getPackageName()
* context.startActivity() 등..

<br><br>

### Context의 종류

Context는 `ApplicationContext`와 `ActivityContext`가 있다.

<br><br>

#### ApplicationContext와 ActivityContext의 차이점

`ApplicationContext`는 항상 앱의 생명주기와 함께하지만, `ActivityContext`는 이름에서 유추할 수 있듯이 액티비티의 생명주기와 함께 작동하고 `onDestroy()`와 함께 사라지게 된다.

`ApplicationContext`는 `getApplication()`, `getApplicationContext()` 으로 가져올 수 있으며<br>
`ActivityContext`는 `getBaseContext()`, `class명.this`로 가져올 수 있다.

> 이전 회사에서는 어떠한 이유인지 모르겠는데.. 싱글 액티비티만을 사용해서 ApplicationContext와 ActivityContext와 관계 없이 사용했었던 것으로 기억한다. <br>
> 당시에는 크게 의문을 가지지 않았는데 지금 생각해보니 왜 싱글 액티비티로만 구성했는지 이해가 안간다 🤔

<br><br>

#### 언제 어떤 것을 사용해야 하는가?

안드로이드에서 Context를 잘못 사용하면 Memory Leak이 발생할 수 있기 때문에 Context를 적재적소에 사용하는 것이 중요하다.

Context가 2종류가 있고 어떻게 불러오는지도 알겠다. 그럼 적재적소는 어떻게 판단할 수 있을까?<br>
정답은 아래와 같다.

**다이얼로그 호출, 액티비티 실행, Layout inflate 작업을 할 때를 제외하고는 `ApplicationContext`를 사용하는 것이 좋다.**

이유는 앞서 말한 것 처럼 Memory Leak 때문인데, `ApplicationContext`의 경우 싱글톤으로 구현되어 있어서 상관 없지만 
`ActivityContext`의 경우 static한 객체가 해당 Context를 참조할 경우 액티비티가 destroy 되었어도 static한 곳에서 Activity를 참조하고 있기 때문에
GC 대상에서 제외되고 메모리를 환원받지 못하게 되는 상황이 연출되게 된다.

이러한 것이 쌓이고 쌓이면 결국 OOM 이 발생하게 되기 때문에 위에 말한 케이스 빼고는 `ApplicationContext`를 사용하도록 하자.

<br><br>

### 회고

지금도 주니어지만.. 오랜만에 Context에 대한 정리를 해보았는데 처음 공부하고 "오..오.. 😮" 했던 기억이 새록새록 떠오른다.

기초가 탄탄한 개발자가 되자 ! (갑자기 분위기 화이팅 🤣)

###### 참고
* 내 Trello..!
