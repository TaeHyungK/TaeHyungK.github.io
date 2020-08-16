---
layout: post
title:  "[Android] 메모리 관리 - OOM(Out Of Memory)"
categories: [Android]
tags: [CS]
author: [TaeHyungK]
---

* content
{:toc}

### 메모리 관리(OOM(OutOfMemory))
- Bitmap
  - 안드로이드 3.0 이하: Bitmap의 메모리가 Dalvik VM에 할당되지 않고 Native Heap 영역에 할당되기 때문에 Bitmap이 VM의 GC(Garbage Collecting)의 대상이 되지 않음.
  
    > Activity의 생명주기에 맞춰 Bitmap의 recycle() 호출하자!

  - ![스크린샷 2019-12-29 오후 6.06.20](/img/스크린샷 2019-12-29 오후 6.06.20.png)
  - 안드로이드 3.0 이상: Bitmap의 메모리가 VM에 할당되기 때문에 다른 객체들 처럼 참조를 끊는 것이 가능하며 참조를 끊으면 GC의 대상이 됨

- Context

	1. Activity Context에 관한 참조를 오랫동안 유지하지 말자.
	2. Activity Context보단 Application Context를 사용(Context.getApplicationContext() 또는 Activity.getApplication()을 통해 얻을 수 있음)
	3. Activity를 참조하는 내부 클래스 사용 지양

###### 출처
- [안드로이드 메모리 관리(OutOfMemory) 1편. Bitmap](http://ccdev.tistory.com/2?category=554484)