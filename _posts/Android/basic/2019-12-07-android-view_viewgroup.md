---
layout: post
title:  "[Android] View와 ViewGroup"
categories: Android
tags: CS
author: TaeHyungK
---

* content
{:toc}

### View와 ViewGroup

- View
  - 화면에 보이는 각각의 구성 요소 (버튼, 텍스트 등등)
  - 흔히 Control이나 Widget이라 불리는 UI 구성 요소

- ViewGroup
  - 뷰들을 여러개 포함하고 있는 것
  - 뷰 그룹도 뷰를 상속하여 뷰가 됨
- Widget
  - 뷰 중에서 일반적인 컨트롤의 역하을 하고 있는 것
  - 버튼, 텍스트 등등
- Layout
  - 뷰 그룹 중에서 내부에 뷰를 포함하고 있으면서 그것들을 배치하는 역할을 하는 것
    - ex) LinearLayout, RelativeLayout, FrameLayout 등