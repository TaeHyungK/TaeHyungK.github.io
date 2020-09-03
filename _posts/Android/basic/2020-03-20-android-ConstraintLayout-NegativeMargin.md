---
layout: post
title:  "[Android] ConstraintLayout의 음수 마진(Negative Margin)"
categories: [Android, Tips]
tags: [tips]
---

### 음수 마진(Negative Margin)

최근 UI 수정 사항이 생기며 기존 xml을 수정해야 하는 상황이 생겼다. (~~다른 분이 짰던 코드를 수정하는 상황이였다.~~) 하여 내가 해결한 방법을 기록하고자 포스트를 하게 되었다.

UI 수정하며 기존 공통 커스텀 뷰는 RelativeLayout을 상속받아 구현했지만 이번 공통 커스텀 뷰는 ConstraintLayout을 사용하였다. 그러자, 음수 마진을 사용한 뷰에서 부모 뷰를 넘치게 구성했던 UI가 부모뷰를 넘어가지 못하는 현상이 발생하였다.

RelativeLayout을 상속받은 경우 음수 마진 값을 사용하여 부모뷰를 넘치는 뷰를 그리는 것이 가능했다. (물론 `clipChildren = false` 옵션을 주어야 정상적으로 노출이 된다.)
- ConstraintLayout을 제외한 LinearLayout, RelativeLayout, FrameLayout 등에서는 문제 없이 사용이 가능하다.








##### RelativeLayout의 음수 마진

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:layout_width="500dp"
        android:layout_height="500dp"
        android:layout_marginStart="40dp"
        android:layout_marginTop="40dp"
        android:background="#ffffff"
        android:clipChildren="false">

    <android.support.constraint.ConstraintLayout
            android:id="@+id/constraint_layout"
            android:layout_width="100dp"
            android:layout_height="100dp"
            android:layout_marginTop="-50dp"
            android:background="#f00"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent" />
</RelativeLayout>
```

- 하기 이미지를 보면 점선으로 부모뷰를 넘어간 것을 볼 수 있다.
![스크린샷 2020-03-22 오후 3.46.35](/img/스크린샷 2020-03-22 오후 3.46.35.png)


##### ConstraiytLayout의 음수 마진

하지만, ConstraintLayout의 경우 **음수 마진이 정상적으로 위치를 잡을 수 없도록**되었으며 ConstraintLayout을 상속받은 커스텀 뷰에서 위치를 정상적으로 잡을 수 없게 되었다.
```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="500dp"
    android:layout_height="500dp"
    android:layout_marginStart="40dp"
    android:layout_marginTop="40dp"
    android:background="#ffffff"
    android:clipChildren="false">

    <android.support.constraint.ConstraintLayout
            android:id="@+id/constraint_layout"
            android:layout_width="100dp"
            android:layout_height="100dp"
            android:layout_marginTop="-50dp"
            android:background="#f00"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent" />
</android.support.constraint.ConstraintLayout>
```

- 하기 이미지를 보면 빨간 뷰가 부모 뷰를 넘어가지 못한 것을 볼 수 있다.
![스크린샷 2020-03-22 오후 3.50.24](/img/스크린샷 2020-03-22 오후 3.50.24.png)


##### ConstraintLayout에서 부모뷰 넘치게 xml 짜기

**ConstraintLayout을 사용하면서 부모뷰를 넘어가고 싶은 경우** 약간의 꼼수를 부리면 RelativeLayout의 음수 마진을 사용한 것처럼 사용할 수 있다.
```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:layout_width="500dp"
        android:layout_height="500dp"
        android:layout_marginStart="40dp"
        android:layout_marginTop="40dp"
        android:background="#ffffff"
        android:clipChildren="false">

    <android.support.constraint.ConstraintLayout
            android:id="@+id/constraint_layout"
            android:layout_width="100dp"
            android:layout_height="100dp"
            android:layout_marginBottom="50dp"
            android:background="#f00"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintBottom_toBottomOf="@+id/guide_line" />

    <android.support.constraint.Guideline
            android:id="@+id/guide_line"
            android:layout_width="0dp"
            android:layout_height="0dp"
            android:orientation="horizontal"
            app:layout_constraintGuide_begin="100dp"/>

</android.support.constraint.ConstraintLayout>
```

- Guideline을 통해 Bottom 기준을 잡고 해당 Guideline으로 부터 margin을 주면 부모 뷰를 넘어가게 구성할 수 있다.
![스크린샷 2020-03-22 오후 3.56.29](/img/스크린샷 2020-03-22 오후 3.56.29.png)

 
##### 정리

내가 부린 꼼수가 좋은 방법인지, ConstraintLayout을 Constraint스럽게 짠 코드인지는 잘 모르겠지만 간단히 정리해보자면..
- ConstraintLayout은 음수 마진(Negativ Margin)을 통해 부모 뷰를 넘어가는 UI를 짜는 것이 불가능하다.
- 위 문제를 해결하기 위해서는 넘어가고 싶은 영역(예시에서 top)의 반대편 영역(예시에서 bottom)을 기준으로 margin을 주면 된다.
