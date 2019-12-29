---
layout: post
title:  "[Android] Reflection"
categories: Android
tags: CS
author: TaeHyungK
---

* content
{:toc}

###Reflection

 - 개념 : 구체적인 클래스의 타입을 알지 못해도, 컴파일 된 바이트 코드를 통해 역으로 클래스의 정보를 알아내어 클래스를 사용할 수 있는 기법.
        거울에 비친 모습과 비슷해서 붙여진 이름
 - 리플렉션을 통해 얻을 수 있는 정보
   - ClassName
   -  Class Modifiers (public, private, synchronized 등..)
   -  Package Info
   -  Superclass
   -  Implemented Interface
   -  Constructors
   -  Methods (배열 형태로 모든 메소드를 불러올 수 있음, 메소드 명을 알면 특정 메소드를 불러올 수 있음)
   -  Fileds (배열 형태로 모든 필드를 불러올 수 있음, 필드 명을 알면 특정 필드를 불러올 수 있음)
   -  Annotations 등등 ..
       
- 장점 : 접근이 불가한 private 영역에 접근하게 해주는 등 유연하게 사용할 수 있도록 함

- 단점 : 성능 저하의 요인이 될 수 있음(정보를 찾을 때 드는 시간비용) <br>(특히, for문에서 사용X )


참고 사이 : http://hiddenviewer.tistory.com/114