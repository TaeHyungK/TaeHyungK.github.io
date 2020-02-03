---
layout: post
title:  "[Android] Player - 스트리밍 종류, 동작방식 및 상태"
categories: Android
tags: CS
author: TaeHyungK
---

* content
{:toc}

### 동영상 스트리밍 종류

 - RTSP (RealTime Streaming Protocol)
   - 실제적인 미디어 스트림 데이터를 전송하는 용도가 아닌 **미디어 스트림의 실시간 제어를 위해 사용**함
   - 명령어는 "PLAY", "PAUSE" 와 같이 VCR 동작과 유사함
   - **시간 정보**를 바탕으로 서버에 접근함
   - 미디어 스트림 데이터는 대부분 RTP를 통해 전송함
  
 - HLS (HTTP Live Streaming)
   - 전체 스트림을 Http 기반의 작은 단위로 쪼개어 전송하고 이를 다시 재조합하여 플레이어를 통해 재생하게 됨
   - HTTP 를 사용하기 때문에 양방향 방식을 지원하지 않음
   - 요청과 응답이 1:1로 대응됨
   - 방화벽 설정이 단순함
   > HLS 확장자
   >   - m3u8: m3u 파일인데 utf-8로 인코딩 되어있다는 의미
   >   - m3u: 멀티미디어 파일의 재생 목록을 관리하는 파일








### Player

 - Player의 동작 방식
   ```java
   /* ExoPlayer*/
   // 플레이어 초기화
   SimpleExoPlayer player = ExoPlayerFactory.newSimpleInstance(this, new DefaultTrackSelector());
   
   // 뷰에 Player 셋팅
   playerView.setPlayer(player);
   
   // 재생할 미디어 소스 생성
   DataSourceFactory dataSourceFactory = new DefaultDataSourceFactory(this, Util.getUserAgent(this, getString(R.string.app_name)));
   ExtractorMediaSource mediaSource = new ExtractorMediaSource.Factory(datasourceFactory).createMediaSource(dataUri);
   
   // 플레이어에 미디어 소스 셋팅
   player.prepare(mediaSource);
   // 재생할 준비가 되면 바로 재생하도록
   player.setPlayWhenReady(true);
   ...
   playerView.setPlayer(null); // 플레이어 설정 해제
   player.release(); // 플레이어 리소스들 해제
   player = null;
   
   /* MediaPlayer */
   Uri myUri = ...;
   // 플레이어 생성
   MediaPlayer mediaPlayer = new MediaPlayer();
   // 오디오 스트림 설정값 셋팅
   mediaPlayer.setAudioStreamType(AudioManager.STREAM_MUSIC);
   // 재생할 미디어 소스 생성 및 설정
   mediaPlayer.setDataSource(getApplicationContext(), myUri);
   // 플레이어에 준비하도록 알림
   //mediaPlayer.prepare();
   mediaPlayer.prepareAsync();
   // 재생 시작
   mediaPlayer.start();
   ...
   mediPlayer.release(); // MediaPlayer 해제
   mediaPlayer = null;
   ```
    > Android MediaPlayer
    >   - setDataSource()를 통해 재생할 Stream Data를 설정하게 되는데 해당 파일이 존재하지 않을 경우, IllegalArgumentException 또는 IOException 이 발생하므로 주의가 필요하다.
    >   - mediaPlayer 메모리 해제를 하지 않을 경우 Activity가 멈출 때 마다(e.g. 화면 전환할 경우 Activity의 재실행) 새로운 객체를 생성하게 되어 시스템 자원에 leak 이 발생할 수 있다.

 - Player의 상태
   ![스크린샷 2020-02-03 오후 8.58.21](/img/스크린샷 2020-02-03 오후 8.58.21.png)
   > 최초 플레이어 생성 시 `IDLE` 상태가 되고
   >
   > setDataSource() 를 호출하게 되면 `Initialized` 상태가 된다.
   >
   > 이후 prepare() 혹은 prepareAsync() 함수를 호출할 수 있고, MediaPlayer가 준비가 되면 `Prepared` 상태가 된다.
   >
   > 이후에 start() 함수를 호출하여 미디어가 재생되도록 할 수 있다. 이때에는 `Started`, `Paused`, `PlaybackCompleted` 사이를 왔다갔다 할 수 있고 start(), pause(), seekTo() 함수를 사용할 수 있다.
   >
   > stop() 함수를 호출할 경우 `Prepared` 상태로 되지 않으면 start() 함수를 호출할 수 없으니 주의해야 한다.
   >   - 틀린 상태에서 틀린 함수를 호출하게 되면 버그가 발생할 수 있으므로 항상 상태를 염두해 두고 MediaPlayer 클래스를 사용하는 것이 중요하다.

 - MediaPlayer 재생 순서
   1. MediaPlayer 생성
   2. setDataSource() - Stream Data 셋팅
   3. prepare() 또는 prepareAsync() - 재생 전 준비
   4. start() - 재생
   
   > 주의 사항
   >   - prepare()를 할 때에는 메인 스레드에서 하지 않도록 하는 것이 중요하다. prepare()를 할 때에 미디어 데이터를 가져오고 디코딩하는 과정이 포함될 수 있으므로 메인 스레드를 통해 재생하였을 때 그런 과정들이 포함되었을 경우 UI는 일시 정지에 걸리게 되고 ANR과 같은 현상이 발생할 수 있기 때문이다. 그래서 보통 별도의 작업 스레드에서 prepare() 한 후 해당 스레드에서 메인 스레드에 알려주는 방식을 사용한다.
   > 
   >     안드로이드 기본 MediaPlayer에서는 위와 같은 동작을 편리하게 할 수록 prepareAsync() 함수를 제공하고 있다. 이 함수는 미디어를 백그라운드에서 준비시키기 시작하고 이후 미디어가 준비가 되면 `onPreparedListener`의 `onPrepared()` 함수가 호출되며 해당 메소드가 콜되었을 때 미디어를 start() 하면 된다.
   >   - 플레이어 사용을 완료한 경우 MediaPlayer를 해제시켜주는 것 또한 중요하다. 사용자가 화면을 가로/세로로 바꾼다던가 하는 행동을 통해 Activity가 종료되었다가 다시 켜지게 되면 안드로이드는 새로 플레이어를 생성하기 때문에 OOM이 발생할 여지가 있기 때문이다. 플레이어의 메모리 해제는 `release()` 함수를 통해 수행하게 된다.


 - ExoPlayer
   - Android 프레임워크의 일부가 아닌 Android SDK와 별도로 배포되는 오픈 소스 프로젝트
   - MediaPlayer를 더 안정적으로 사용할 수 있도록 만든 플레이어 라이브러리
   - 안드로이드 4.1(젤리빈) 이상에서 사용 가능함
   - 구글에서 YouTube와 Play Movies 앱에서 해당 라이브러리를 사용하고 있음