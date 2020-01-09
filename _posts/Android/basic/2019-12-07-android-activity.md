---
layout: post
title:  "[Android] Activity"
categories: Android
tags: CS
author: TaeHyungK
---

* content
{:toc}

### Activity

- Activity?
  - 사용자에게 UI가 있는 화면을 제공하는 앱 컴포넌트
  - 어플리케이션은 무조건 하나이상의 액티비티가 존재해야함

- Activity LifeCycle
  - ![ActivityLifeCycle](/img/ActivityLifeCycle.png)
  - onCreate(): 액티비티가 생성될 때 호출되며 사용자 인터페이스 초기화에 사용됨
  - onStart(): 액티비티가 사용자에게 보여지기 바로 직전에 호출됨
  - onResume(): 액티비티가 사용자와 상호작용하기 바로 전에 호출됨
  - onPause(): 다른 액티비티가 보여질 때 호출됨. 데이터 저장, 스레드 중지 등의 처리하기에 적당한 메소드
  - onStop(): 액티비티가 더이상 사용자에게 보여지지 않을 때 호출됨. 메모리가 부족할 경우에는 onStop() 메소드가 호출되지 않을 수도 있음
  - onDestroy() 액티비티가 소멸될 때 호출됨. finish() 메소드가 호출되거나 시스템이 메모리 확보를 위해 액티비티를 제거할 때 호출됨
  - onRestart(): 액티비티가 멈췄다가 다시 시작되기 바로 전에 호출됨. onStop() 에서 호출되며 onStart()를 호출함

  > 추가 tip
  >
  > - 사용자가 홈 키를 눌렀는지를 파악하는 방법
  >   - Activity 클래스에서 제공하는 onUserLeavHint()는 사용자가 홈 키를 눌렀을 때 호출되는 메소드이다.
  >   - 해당 함수를 오버라이딩하면 사용자가 홈 키를 눌렀는지 파악이 가능하다.

- Intent?

  - 액티비티간에 데이터를 전송할 때 사용
  - putExtra(), getExtra()

- Activity가 종료되는 경우?

  - 사용자가 Back 키를 눌러 Activity를 종료한 경우
  - Activity가 백그라운드에 있을때 시스템 메모리가 부족해진 경우(OS가 강제 종료시키는 경우)
  - 언어 설정을 변경한 경우
  - 화면을 가로/세로 회전할 때
  - 폰트 크기나 폰트를 변경했을 때

- onSaveInstanceState()?

  - 해당 메소드를 이용하면 Activity가 종료될 때 데이터를 저장할 수 있다.
  - Activity가 종료되는 경우?
    - 사용자가 Back 키를 눌러 Activity를 종료한 경우
    - Activity가 백그라운드에 있을때 시스템 메모리가 부족해진 경우(OS가 강제 종료시키는 경우)
    - 언어 설정을 변경한 경우
    - 화면을 가로/세로 회전할 때
    - 폰트 크기나 폰트를 변경했을 때
  - 화면 회전시 LifeCycle
    - onPause() -> onSaveInstanceState() -> onStop() -> onDestroy() -> onCreate() -> onStart() -> onResume()