---
layout: post
title:  "[Android] Handler, Looper, Thread에 대하여"
categories: [Android, CS]
tags: [CS]
---

### 안드로이드에서는 왜 Main Thread(UI Thread) 에서만 UI를 그릴 수 있을까?

UI 작업을 비동기적으로 처리하게 되면 두 스레드가 동시에 같은 View에 대하여 처리를 요청하게 되었을 때 동기화 문제가 발생하게 된다.

그러한 동기화 문제를 해결하기 위해 **안드로이드에서 UI 변경은 Main Thread에서만 가능**하다.


### Handler, Looper, Thread ?

멀티 스레드 환경에서 Main Thread와 Worker Thread 간 즉, 서로 다른 스레드 간에 통신을 도와주는 도구가 바로 `Handler`와 `Looper` 이다.







#### 동작 방식

1. 핸들러의 sendMessage()를 통해 `Message Queue`에 메세지를 차례대로 넣는다.
2. Looper는 `Message Queue`로부터 하나씩 메세지를 꺼내 적당한 Handler에게 전달하게 된다.
3. Looper로부터 메세지를 전달받은 Handler는 handleMessage()를 통해 메세지를 처리한다.

#### Handler 란?

Handler를 생성하게 되면 호출한 Thread의 Message Queue와 Looper에 자동으로 연결이 된다.

Handler는 혼자서는 아무 동작을 할 수 없기 때문에 Thread, Looper, Message Queue가 없다면 아무것도 할 수가 없다.

만약 Worker Thread에서 호출된 Handler 내에서 UI 관련 작업을 하게 되면 Exception이 발생하게 된다.
- Exception: `android.view.ViewRootImpl$CalledFromWrongThreadException: Only the original thread that created a view hierarchy can touch its views.`

#### Looper 란?

Looper는 Thread 당 하나씩만 가질 수 있고 Message Queue가 비어있을 때에는 아무 동작하지 않다가 Message가 들어오게 되면 Message를 꺼내 적절한 Handler로 전달하는 역할을 한다.

기본적으로 새로 생성한 Thread는 Looper를 가지지 않고 Looper.prepare() 메소드를 호출해야 Looper가 생성되게 된다.

#### 주의 사항

- Looper.loop()을 통해 Looper가 Message를 기다리는 동작을 시작한 후에는 해당 Activity가 onDestory() 될 때에 **꼭** Looper.quit() 를 통해 루퍼를 정지시켜줘야 한다.

  