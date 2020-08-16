---
layout: post
title:  "[Android] AAC (Android Architecture Component)"
categories: [Android]
tags: [Architecture]
---

### AAC (Android Architecture Component)

#### 사용 목적
* 복잡한 LifeCycle을 가진 액티비티나 프래그먼트를 좀 더 쉽게 컨트롤 하기 위하여 사용합니다.
* 액티비티, 프래그먼트 잘못 사용 시 Memory Leak이 발생하게 되고 UI에 데이터를 뿌려주는 역할을 더 쉽게 하기 위하여 사용합니다.
* **LiveData**을 이용해 데이터 변화에 noti를 받아 UI를 쉽게 변화시킬 수 있습니다.
* **ViewModel** 데이터에 관계가 있는 데이터를 저장해두는 역할을 합니다.






  > **AAC LiveData의 setValue() vs postValue()**
  >
  > 두 메소드 모두 MutableLiveData의 값을 변경시킬 수 있는 메소드이며 꼭 이 두 메소드로 데이터를 변경해야 데이터 변화를 감지할 수 있다.
  >
  > **setValue()**
  >
  > * 말 그대로 값을 set하는 함수이다. LiveData를 구독하고 있는 옵저버가 있는 상태에서 `setValue()`를 통해 값을 변경 시키면 **메인 스레드**에서 그 값을 즉시 반영한다. 중요한 점은 **메인 스레드**에서 값을 dispatch시킨다는 점이다.
  >
  > **postValue()**
  >
  > * `setValue()` 와 마찬가지로 값을 set하는 함수이다. 다만 그 과정이 **백그라운드**에서 진행된다고 보면 된다.
  >
  >   (내부적으로 `new Handler(Looper.mainLooper()).post(() -> setValue())` 이런 코드가 실행된다. 결국 마지막으로 값을 변경하는 스레드는 메인 스레드이긴 하다.)
  >
  > 따라서, `postValue()`를 한 다음 바로 다음 라인에서 `getValue()`를 호출할 경우 백그라운드에서 돌기 때문에 변경된 값을 받아오지 못할 확률이 크다.
  > 반면에, `setValue()`로 값을 변경할 경우 다음 라인에서 `getValue()`를 호출하여도 변경된 값을 바로 읽어올 수 있다.

###### 참고 사이트
- [📃 안드로이드 인텐트 레퍼런스](https://developer.android.com/topic/libraries/architecture?hl=ko)
- [[Android] AAC ViewModel 을 생성하는 6가지 방법 - ViewModelProvider](https://readystory.tistory.com/176)
- [[생성 패턴] 팩토리 패턴(Factory Pattern) 이해 및 예제](https://readystory.tistory.com/117)
- [AAC LiveData setValue() vs postValue()](https://wooooooak.github.io/android/2019/06/11/Android_liveData_value/)