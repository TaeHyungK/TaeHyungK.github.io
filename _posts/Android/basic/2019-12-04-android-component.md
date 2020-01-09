---
layout: post
title:  "[Android] 4대 컴포넌트"
categories: Android
tags: CS
author: TaeHyungK
---

* content
{:toc}

### Activity

 - UI 화면을 담당하는 컴포넌트
 - 자바 소스에서 Activity 클래스를 상속 받아야 함
 - 안드로이드 앱은 반드시 하나 이상의 Acitivity를 포함해야 함
 - 두 개의 액티비티를 동시에 display할 수 없음
 - 다른 앱의 액티비티 호출도 가능
 - [Activity 상세보기](https://taehyungk.github.io/2019/12/07/android-activity/)






### Service

 - Background에서 동작하는 컴포넌트
 - Activity가 종료되어 있는 상태에서도 동작시키기 위함 (ex. MP3 Player)
 - 사용법
    1. startService() - stopService()
    
       ```java
       Intent intent = new Intent("test.test.com.startService");
       intent.putExtra("count", "1");
       startService(intent); //service 실행
       
       Intent intent = new Intent("test.est.com.startService");
       stopService(intent); //service 중지
       // 동일한 Intent Filter를 사용해서 Service를 구현해야 한다.
       ```
   
    2. bindService()
      - 다른 프로세스(어플리케이션)와도 Data 공유 및 통신을 하게 되는 Service, 클라이언트 - 서버에서 서버의 역할처럼 수행 
      Service 내에 onBind() 함수 필수 구현
    
    - 주의사항
      -  Service의 ANR발생 : 모든 컴포넌트들은 Main Thread 안에서 동작함. 즉, Service도 Main Thread(UI 작업 처리해주는 Thread)를 통해서 동작. <br>Thread 작업이 필요한 경우, 작업 Thread를 생성해서 관리해줘야 함. 그렇지 안하면 ANR(Application Not Responding) 발생
      - Service 실행 중 startService()호출 : Service 실행 도중 startService() 실행시, Service의 onStartCommand() 메소드 호출함.
      <br>때문에, 중요한 작업일 경우 onCreate가 아닌 onStartCommand에 구현해주어야 함

- Service 주기
  - ![스크린샷 2019-12-29 오후 5.19.48](/img/스크린샷 2019-12-29 오후 5.19.48.png)
  -  onStartCommand() 메소드의 3가지 리턴 타입
    - START_STICKY
      - Service가 강제 종료되었을 경우 시스템이 다시 Service를 재시작 시켜주는 플래그. 이 때, intent값은 null로 초기화 시켜 재시작됨
    - START_NOT_STICKY
      - 강제종료된 Service가 재시작되지 않도록하는 플래그
    - START_REDELIVER_INTENT
      - START_STIKCY와 마찬가지로 강제종료되었을 경우 시스템이 Service를 재시작 시켜줌. 이 때, intent 값을 유지시켜줌.

### Content Provider

 - 데이터를 관리하고 다른 앱 데이터를 제공해주는 컴포넌트
 - DB의 데이터를 전달할 때 많이 사용
 - 생명주기가 없음
 - 파일입출력, SQLiteDB, Web등을 통해 데이터를 관리
 - 다른 앱의 데이터 변경도 가능

### Broadcast Receiver

 - 특정 작업이 벌어질 경우 그것을 받고자 하는 곳에 알려주는 기능을 하는 컴포넌트

 - 리시버의 동작 제한(인텐트 플래그를 통한)
   - FLAG_EXCLUDE_STOPPED_PACKAGES
     - 앱이 한번이라도 실행됐을 때만 리시버가 동작하도록 API 12 이후로는 이 플래그가 기본으로 설정됨
   - FLAG_INCLUDE_STOPPED_PACKAGES
     - 한번도 실행되지 않은 앱도 리시버가 동작하도록
   - FLAG_RECEIVER_REGISTERD_ONLY
     - 오직 동적 receiver만 방송받을 수 있도록
   - FLAG_RECEIVER_REPLACE_PENDING
     - 동일한 액션으로 중복해서 방송하는 경우 중복된 방송제거 하도록
     
 - 리시버 동작시간 제한
   - Background Receiver: 60초
   - Foreground Receiver: 10초
   
 - setPackage()함수로 원하는 패키지의 리시버만 동작하도록 방송 가능
   - 정적 Receiver
     - 한번 등록되면 해제 불가
     - Manifest에 receiver를 등록하는 방식으로 receiver를 등록
     - 해당 앱이 설치될 때 자동으로 등록됨
     - 동시에 실행되지 않음. 한 개씩만 처리
     - 앱이 먼저 설치된 우선순위로 실행됨 (setPriority(1)함수로 우선순위 조작 가능)
     - 사용법
     
         ~~~
         1. Manifest에 <receiver></receiver> 등록 or 코드에 Receiver 등록
         2. sendBroadcast(new Intent(“test.test.com.sendreceiver.gogogo”);
         ~~~
	 
   - 동적 Receiver
     - 등록과 해제가 자유로움
     - Manifest가 아닌 소스상에 등록함
     - 동시에 실행됨(우선순위 중요하지 않음)
     - 리시버가 작성된 컴포넌트가 종료되면 동작하지 않음(onDestory()함수에 unregister(receiver)로 리시버 해제를 하지 않을 경우 동작함. 하지만 메모리 누수 때문에 리시버 해제 필수!)
       - sendOrderBroadcast() 사용시 우선순위 적용해서 처리 가능
       - 사용법
           1. 동적리시버 생성 및 등록
           
               ```java
              IntentFilter intentFilter = new IntentFilter();
              IntentFilter.addAction(“test.test.com.action.TEST”);
              BoradcastReceiver receiver = new BroadcastReceiver(){
                  @Override
                  Public void onReceive(Context context, Intent intent){
                      // TODO 동작
                  }
              }
              // 위에서 설정한 인텐트필터 + 리시버정보로 리시버 등록
              registerReceiver(receiver, intentFilter);
              ```
              
           2. 동적으로 등록한 리시버 호출
           
                ```java
                Intent intent = new Intent();
                intent.setAction(“test.test.com.action.TEST”);
                sendBroadcast(intent);
                ```

###### 출처
- [Service](http://arabiannight.tistory.com/entry/%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9CAndroid-Service-%EC%82%AC%EC%9A%A9%EB%B2%95)
- [BroadcastReceiver](http://thereclub.tistory.com/15)