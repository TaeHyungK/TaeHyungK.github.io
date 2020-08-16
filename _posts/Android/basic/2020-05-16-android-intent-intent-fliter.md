---
layout: post
title:  "[Android] 인텐트와 인텐트 필터"
categories: [Android, CS]
tags: [CS]
---

### 인텐트와 인텐트 필터에 대해 알아보자

#### Intent

##### Component를 실행하기 위해 시스템에 넘기는 정보

즉, 실행하고자 하는 컴포넌트 정보를 담은 Intent 구성 -> 시스템 -> Intent 정보를 통해 그에 맞는 Component를 실행하게 된다.

#### IntentFilter

##### 특정 인텐트를 받을지 말지 정의하는 역할을 수행함







### Intent 객체의 구성 요소
- Action(액션): 수행할 액션 이름
- Data(데이터): 수행할 데이터의 URI
  - ex) tel:
- Category(카테고리): 수행할 액션에 대한 추가적인 정보
- Type(타입): 수행할 인텐트 데이터의 명시적인 타입
  - ex) video/mpeg
- Component name(컴포넌트 이름): 대상 컴포넌트의 완전한 클래스 이름
- Extras(추가 정보): 인텐트를 다루는 컴포넌트에 추가적으로 전달할 한 쌍의 키/값

### Intent의 주요 액티비티 액션

|Action|설명|
|-----|---------|
|ACTION_MAIN|시작 액티비티를 지정하기 위한 액션|
|ACTION_VIEW|데이터의 URL로 가장 적절한 액티비티를 호출하는 액션<br>content://contacts/people/1|
|ACTION_DEFAULT|ACTION_VIEW와 동일|
|ACTION_EDIT|수정하기 위해 호출하는 액션|
|ACTION_DELETE|삭제하기 위해 호출하는 액션|
|ACTION_DIAL|전화 다이얼 액티비티를 호출하는 액션<br>content://contact/people/1<br>tel:01012345678|
|ACTION_CALL|전화 액티비티를 호출하는 액션<br>tel:01012345678|
|ACTION_WEB_SEARCH|웹 검색 액티비티를 호출하는 액션|
|ACTION_SEARCH|검색 액티비티를 호출하는 액션|
|ACTION_SENDTO|이메일 등의 메시지 전송을 지정하는 액션|
|ACTION_ANSWER|전화 착신을 위한 액션|

> `content://contacts/people/1`: 주소록에서 식별자가 1인 사람에 대한 정보를 가리킴
>
> `tel:01012345678`: 전화번호 01012345678을 가리킴

###### 참고 사이트
- [📃 안드로이드 인텐트 레퍼런스](http://developer.android.com/reference/android/content/Intent.html)
- [https://kairo96.gitbooks.io/android/ch2.8.html](https://kairo96.gitbooks.io/android/ch2.8.html)